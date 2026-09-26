# Functional-oriented Rust server style

Canonical fleet rule: https://github.com/ORESoftware/ores-middleware/blob/main/FUNCTIONAL-STYLE.md

**Build values; do not mutate caller-owned state.** Avoid `&mut T` parameters except `&mut self` on state owners and APIs that force mutation. Prefer complete `T`/`Option<T>`/`Result<T,E>` returns, consuming builders, iterator pipelines, and explicit `state -> new state` transitions.

## Pure core, imperative shell
Keep `main.rs` a composition root. Load config and initialize runtime/telemetry at the edge, then wire immutable application state, adapters, routes, and lifecycle. Put deterministic transformation and policy in pure functions/modules. Keep `http` thin; keep DB/queue/fs/clock/random/network work in `adapters`; keep `server` for orchestration.

Split mixed handlers into `parse -> pure validation/decision -> effect -> pure response mapping`. Extract modules for ownership/testability/reuse/effect boundaries, not just file length.

## Mutable hot paths
When an API forces mutation or measurement shows copying/allocation is wrong, confine it and add:
```rust
// HOT-PATH (imperative by design): <what>, <why>, <where mutation is confined>,
// <what immutable value callers receive>.
```
Use `TODO(measure)` for unmeasured performance claims.

## Review
Return values instead of filling caller-owned slots/collections; keep validation/policy testable without Tokio/Axum/DB; make effects explicit; split large handlers into named stages; require non-obvious mutation to be state ownership or documented `HOT-PATH`; constructors return complete values.
