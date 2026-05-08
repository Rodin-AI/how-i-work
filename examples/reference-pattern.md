# Example: Reference Pattern — Adding a New API Endpoint

This is a complete reference pattern document. It's the blueprint an agent follows when it needs to "add a new endpoint." Your version should reflect YOUR project's real architecture — this is a starting template.

---

# Pattern: Adding a New API Endpoint

Endpoints are the entry point for all mutations. Every state change goes through an HTTP handler → service → repository.

## Files to create

1. `internal/orders/handler.go` — HTTP handler (or add a method to existing)
2. `internal/orders/service.go` — business logic
3. `internal/orders/service_test.go` — tests
4. `api/openapi.yaml` — update the spec

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

- Happy path: valid request → order created, correct response
- Validation: rejects missing fields, invalid account ID
- Permissions: returns 403 when account lacks permission
- Conflict: returns 409 on duplicate idempotency key (if applicable)

## Why this structure?

- **Handler does ONLY: parse, validate, delegate, respond.** No business logic. This lets us test business logic without HTTP.
- **Service accepts a Command struct, not raw request fields.** Commands are the internal API — they can come from HTTP, queues, or tests.
- **Errors are wrapped at every level with context.** `fmt.Errorf("creating order: %w", err)` gives a full trace without logging everywhere.
- **Repository is an interface.** Service doesn't know if it's Postgres, DynamoDB, or a mock.
