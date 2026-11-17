# toolspec

`github.com/PivotLLM/toolspec` is the shared, transport-neutral **tool contract**
between tool packages and the hosts that run them. It is deliberately
dependency-free (standard library only) so it can be imported by tool packages,
by a host's registry/bridge, and by external software such as Maestro as a standalone module without import cycles.

## Model

A tool package implements `ToolProvider` and returns `ToolDefinition`s with
**bare** names (e.g. `read`). The host/aggregator that mounts the provider
supplies the namespace (e.g. `file`); the published name is `<namespace>_<bare>`
(e.g. `file_read`). Namespacing lives in the aggregator, not the tool package.

```go
type ToolProvider interface {
    RegisterTools(deps Deps) []ToolDefinition
}
```

- `ToolDefinition` — one tool: name, parameters (or a raw JSON Schema), handler,
  and flags (`SessionScoped`, `Async`, `PrimaryOnly`, `DefaultAllow`).
- `ToolHandler` — `func(call *ToolCall) (*Result, error)`.
- `ToolCall` — per-invocation bundle (ctx, args, agent/session/channel, async `Notify`).
- `Result` — model-facing `ForLLM`, optional `ForUser`, plus `Silent`/`IsError`/`Async`/`Media`.
- `Deps` — dependency injection handed to a provider at construction.

## Stability

This module is the contract that drop-in tool packages compile against, so it is
intended to be small and stable. Breaking changes are versioned.
