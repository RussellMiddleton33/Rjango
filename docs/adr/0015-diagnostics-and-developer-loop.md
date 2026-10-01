# ADR 0015: Diagnostics contract and development-loop budgets

**Decision status:** SPEC-LOCKED (budgets PROPOSED).
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidate 2.

[ADR index](README.md) · [Canonical specification](../specification/24-developer-experience-and-diagnostics.md)

## Context

The primary audience is Django developers who do not know Rust. Compiler errors from macros, traits and async `Send` bounds, slow rebuilds and the lack of a REPL are the main adoption risks. Candidate 1 stated goals but no mechanism, and promised compile-time checks a proc macro cannot perform.

## Decision

- **Diagnostics are API.** `#[diagnostic::on_unimplemented]` on user-facing traits; per-argument and `Send` assertions in handler macros; error-tolerant macros that keep IDE completion working; shallow public types; reviewed trybuild message snapshots.
- **Extension points are annotated functions,** not user-implemented async traits.
- **Honest classification.** Cross-type checks are AMG startup/`rjango check` errors, not compile errors.
- **Error model.** One framework error type at Tier 0, derived domain errors at Tier 1, and status mapping at the adapter.
- **Development loop.** Proposed rebuild budgets; a metadata-only introspection mode with no external connections; a dev server that restarts after a successful build; `db shell` and `run <script>` replace `manage.py shell`.
- **Version skew.** Lockstep exact versions plus a Cargo `links` guard.

## Consequences and boundaries

Message text changes are API-impact items. Generated code volume is budgeted. Live code patching is not promised.

## Alternatives and rationale

- **Relying on rustc and Axum default messages:** rejected for the primary audience.
- **Type-state APIs for maximal compile-time checking:** rejected where they degrade messages.
- **An embedded REPL:** not feasible for compiled Rust apps.

## Evidence still required

Rebuild times on the reference app; newcomer usability study; `Send` span quality; rust-analyzer behaviour; skew detection.
