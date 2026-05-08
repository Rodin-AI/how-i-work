# Example: Conventions Doc (Go)

This is what a finished conventions doc looks like. Yours should reflect YOUR repo's actual patterns — this uses Go as an example.

---

# Conventions

## Package Structure
- One package per directory
- Package name matches directory name: `internal/orders/validator.go` → package `validator`
- Public API packages in `pkg/`, internal packages in `internal/`
- One struct per file when the struct has methods

## Error Handling
- Always wrap errors with context: `fmt.Errorf("fetching order %s: %w", id, err)`
- Never return bare `err` without wrapping
- Log at the boundary (HTTP handler, queue consumer), not at the call site
- Use sentinel errors for expected conditions: `var ErrNotFound = errors.New("not found")`

## Testing
- Unit tests: `<file>_test.go` in the same package
- Integration tests: `_test.go` with build tag `//go:build integration`
- Table-driven tests for any function with >2 cases
- Use `testify/require` for assertions (not `assert` — fail fast)

## Naming
- Interfaces: verb phrases (`OrderValidator`, `PositionFetcher`)
- Constructors: `NewOrderService(...)` not `CreateOrderService(...)`
- Boolean methods: `IsValid()`, `HasPosition()` not `CheckValid()`
- Unexported helpers: `doThing()` not `_doThing()` or `helperDoThing()`
