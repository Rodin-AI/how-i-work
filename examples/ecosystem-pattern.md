# Example: Ecosystem Pattern — Error Handling in Go

This is a complete ecosystem pattern file. It documents how authoritative open-source projects handle a specific concern — grounded in real source code, not opinions. Your version should cite the specific projects and versions you analyzed.

---

# Error Handling in Go

## Sources analyzed

- kubernetes/kubernetes (v1.29)
- etcd-io/etcd (v3.5)
- cockroachdb/cockroach (v23.2)

## The pattern

All three projects wrap errors with context at each level:

```go
if err != nil {
    return fmt.Errorf("fetching pod %s: %w", name, err)
}
```

They never:
- Return bare `err` without wrapping
- Use `errors.New` for dynamic messages (that's for sentinel errors)
- Log AND return (pick one — the caller decides)

## Where they disagree

Kubernetes uses `utilruntime.HandleError()` for non-fatal errors in controllers. Etcd propagates everything. CockroachDB has a custom `errors` package with stack traces.

## Recommendation for our repos

Follow the kubernetes pattern: wrap at every level, log at the boundary, propagate in the middle. Use `fmt.Errorf("context: %w", err)` everywhere.
