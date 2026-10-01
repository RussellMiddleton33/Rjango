# Architecture decision records

[Master specification](../specification/README.md)

These ADRs preserve decisions already agreed in the conversation. SPEC-LOCKED and VALIDATION REQUIRED can coexist: agreement does not establish implementation evidence. Exact syntax and open choices remain proposed.

| ADR | Decision | Status |
| --- | --- | --- |
| [0001](0001-tokio-runtime.md) | Tokio multi-thread runtime | SPEC-LOCKED · VALIDATION REQUIRED |
| [0002](0002-axum-tower.md) | Axum HTTP and Tower middleware | SPEC-LOCKED · VALIDATION REQUIRED |
| [0003](0003-postgresql-first.md) | PostgreSQL-first database design | SPEC-LOCKED · VALIDATION REQUIRED |
| [0004](0004-seaorm-sqlx.md) | SeaORM 2.x over SQLx 0.9 with direct escape hatch | SPEC-LOCKED · VALIDATION REQUIRED |
| [0005](0005-explicit-database-context.md) | Explicit database and transaction context | SPEC-LOCKED · VALIDATION REQUIRED |
| [0006](0006-immutable-amg.md) | Immutable, deterministic Application Metadata Graph | SPEC-LOCKED · VALIDATION REQUIRED |
| [0007](0007-explicit-production-migrations.md) | Explicit production migrations | SPEC-LOCKED · VALIDATION REQUIRED |
| [0008](0008-no-hidden-network-io.md) | No hidden network I/O | SPEC-LOCKED · VALIDATION REQUIRED |
| [0009](0009-ai-native-and-mcp-security.md) | AI-native, not AI-dependent; explicit MCP boundaries | SPEC-LOCKED · VALIDATION REQUIRED |
| [0010](0010-application-operations.md) | Application operations as the universal entry contract | SPEC-LOCKED · VALIDATION REQUIRED |
| [0011](0011-transactional-outbox.md) | Transactional outbox and transaction-owned durable effects | SPEC-LOCKED · VALIDATION REQUIRED |
| [0012](0012-scoped-data-access-and-default-deny.md) | Scoped data access by default; deny-by-default policy | SPEC-LOCKED · VALIDATION REQUIRED |
| [0013](0013-transaction-ownership.md) | Transactions own their connection; commit consumes | SPEC-LOCKED · VALIDATION REQUIRED |
| [0014](0014-loaded-relations.md) | Loaded relations as explicit values with runtime-checked accessors | SPEC-LOCKED · VALIDATION REQUIRED |
| [0015](0015-diagnostics-and-developer-loop.md) | Diagnostics contract and development-loop budgets | SPEC-LOCKED (budgets PROPOSED) · VALIDATION REQUIRED |

ADRs 0010–0015 come from the [independent review](../reviews/independent-review-candidate-1.md). ADR 0004 locks SeaORM as the engine but not the specific major versions, and ADRs 0005 and 0008 carry Candidate 2 refinements.

When superseding a decision, retain the old ADR, mark it SUPERSEDED, and link its replacement. VALIDATED claims must link reproducible evidence with versions, scope and limitations.
