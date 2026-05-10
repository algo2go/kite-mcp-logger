# kite-mcp-logger

[![Go Reference](https://pkg.go.dev/badge/github.com/algo2go/kite-mcp-logger.svg)](https://pkg.go.dev/github.com/algo2go/kite-mcp-logger)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Context-aware structured logger port for the algo2go ecosystem.
Defines the `Logger` interface (over `log/slog`) plus three
implementations: `SlogAdapter` (production wrapper), `Noop` (silent
for tests/init), and `Capture` (in-memory accumulator for assertions).

Used by [`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
across 100+ files for ctx-threaded structured logging — the
foundational logging primitive for every MCP tool handler, broker
adapter, scheduler tick, audit-log writer, and OAuth callback.

## Why a separate module?

A `Logger` interface is the cleanest cross-cutting primitive — every
algo2go consumer (broker dashboards, monitoring, future broker
adapters, trading bots) needs structured logging with a standard port.
Hosting the interface + adapters as a module:

- Standardizes the ctx-threaded logging contract across consumers
- Lets unrelated projects adopt `Capture` for table-driven tests
  without pulling in kite-mcp-server
- Reduces the dep-graph weight for users who only need logging

## Stability promise

**v0.x — unstable.** The `Logger` interface signatures may evolve.
Pin `v0.1.0` deliberately. v1.0 ships only after the public API
(interface methods + adapter constructors) is reviewed for stability
and at least one external consumer ships against it.

## Install

```bash
go get github.com/algo2go/kite-mcp-logger@v0.1.0
```

## Public API

### Interface (port.go)

```go
type Logger interface {
    Info(ctx context.Context, msg string, args ...any)
    Warn(ctx context.Context, msg string, args ...any)
    Error(ctx context.Context, msg string, args ...any)
    Debug(ctx context.Context, msg string, args ...any)
    With(args ...any) Logger
}
```

### Adapters

- `NewSlogAdapter(*slog.Logger) Logger` — production wrapper
- `NewNoop() Logger` — silent (tests, init paths)
- `NewCapture() *CaptureLogger` — in-memory accumulator with assertion
  helpers (`AssertContains`, `AssertCount`, `Records()`)

## Reference consumer

[`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
— used in 100+ files including:
- `app/lifecycle.go` — startup/shutdown logging with structured fields
- `kc/usecases/*.go` — every use case threads ctx + Logger
- `kc/audit/store.go` — audit-log persistence with structured records
- `kc/papertrading/engine.go` — paper-trade lifecycle logging
- `kc/alerts/evaluator.go` — alert evaluation tracing

## License

MIT — see [LICENSE](LICENSE).

## Authors

Original design: [Sundeepg98](https://github.com/Sundeepg98) (Zerodha
Tech). Multi-module promotion (2026-05-10): algo2go contributors.
