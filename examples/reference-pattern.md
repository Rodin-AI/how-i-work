# Example: Reference Pattern — Adding a New API Endpoint

This is what a finished reference pattern doc looks like. The prompt at `prompts/build-reference-pattern.md` generates this output. Your version should reflect YOUR project's real architecture — this is a starting template.

---

# Pattern: Adding a New API Endpoint

## What this is

An API endpoint is the entry point for client-initiated mutations. Every state change in this system goes through: HTTP handler → service → repository. Use this pattern whenever you need to expose a new operation to clients (REST, internal API, webhook receiver).

**When to use:** Adding a new user-facing action, CRUD operation, or webhook.
**When NOT to use:** Background jobs (use the worker pattern), event reactions (use the subscriber pattern), or read-only queries with complex filtering (use the query pattern).

## Files to create

1. `internal/orders/handler.go` — HTTP handler (or add a method to existing handler)
2. `internal/orders/service.go` — business logic
3. `internal/orders/service_test.go` — unit tests for the service
4. `api/openapi.yaml` — update the API spec

## 1. The Handler

```go
func (h *Handler) CreateOrder(w http.ResponseWriter, r *http.Request) {
    var req CreateOrderRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "invalid request body")
        return
    }

    if err := req.Validate(); err != nil {
        respondError(w, http.StatusUnprocessableEntity, err.Error())
        return
    }

    order, err := h.service.CreateOrder(r.Context(), req.ToCommand())
    if err != nil {
        handleServiceError(w, err)
        return
    }

    respondJSON(w, http.StatusCreated, order)
}
```

## 2. The Service

```go
func (s *Service) CreateOrder(ctx context.Context, cmd CreateOrderCommand) (*Order, error) {
    account, err := s.accounts.Get(ctx, cmd.AccountID)
    if err != nil {
        return nil, fmt.Errorf("fetching account %s: %w", cmd.AccountID, err)
    }

    if err := s.validatePermissions(account, PermissionCreateOrder); err != nil {
        return nil, err
    }

    order, err := s.repo.Create(ctx, cmd)
    if err != nil {
        return nil, fmt.Errorf("creating order: %w", err)
    }

    return order, nil
}
```

## 3. Tests

```go
func TestCreateOrder(t *testing.T) {
    tests := []struct {
        name    string
        cmd     CreateOrderCommand
        setup   func(*MockRepo, *MockAccounts)
        wantErr string
    }{
        {
            name: "happy path",
            cmd:  CreateOrderCommand{AccountID: "acc-1", InstrumentID: "BTC", Quantity: 10},
            setup: func(repo *MockRepo, accts *MockAccounts) {
                accts.EXPECT().Get(gomock.Any(), "acc-1").Return(&Account{ID: "acc-1", Permissions: []string{"create_order"}}, nil)
                repo.EXPECT().Create(gomock.Any(), gomock.Any()).Return(&Order{ID: "ord-1"}, nil)
            },
        },
        {
            name:    "account not found",
            cmd:     CreateOrderCommand{AccountID: "missing"},
            setup:   func(repo *MockRepo, accts *MockAccounts) {
                accts.EXPECT().Get(gomock.Any(), "missing").Return(nil, ErrNotFound)
            },
            wantErr: "fetching account missing",
        },
        {
            name:    "permission denied",
            cmd:     CreateOrderCommand{AccountID: "acc-1"},
            setup:   func(repo *MockRepo, accts *MockAccounts) {
                accts.EXPECT().Get(gomock.Any(), "acc-1").Return(&Account{ID: "acc-1", Permissions: []string{}}, nil)
            },
            wantErr: "permission denied",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            ctrl := gomock.NewController(t)
            repo := NewMockRepo(ctrl)
            accts := NewMockAccounts(ctrl)
            tt.setup(repo, accts)

            svc := NewService(repo, accts)
            _, err := svc.CreateOrder(context.Background(), tt.cmd)

            if tt.wantErr != "" {
                require.ErrorContains(t, err, tt.wantErr)
            } else {
                require.NoError(t, err)
            }
        })
    }
}
```

## Why this structure?

- **Handler does ONLY: parse, validate, delegate, respond.** No business logic lives here. This means we can test all business logic without spinning up HTTP — the service is the unit under test.
- **Service accepts a Command struct, not raw request fields.** Commands are the internal API — they can come from HTTP handlers, queue consumers, or tests. This decouples "how work arrives" from "what the work is."
- **Errors are wrapped at every level with context.** `fmt.Errorf("creating order: %w", err)` builds a trace without logging at every level. Log once at the boundary (handler), propagate everywhere else.
- **Repository is an interface.** The service doesn't know if it's Postgres, DynamoDB, or a mock. This is what makes the table-driven tests above possible — swap the real repo for a mock, test the logic.
- **Table-driven tests with setup functions.** Each test case declares its own mock expectations. No shared setup that makes tests fragile or hard to read in isolation.
