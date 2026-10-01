# Rjango master specification

**Master status: REVIEW CANDIDATE 2 — NOT FROZEN.** Independent-review corrections are incorporated as architectural direction; implementation planning and coding remain unauthorized.

Read the [independent review and disposition crosswalk](../reviews/independent-review-candidate-1.md), then the [first architecture review](../reviews/architecture-review-candidate-1.md), [Application Operations / Services](22-application-operations-and-services.md), [core durability/message contracts](23-durability-and-message-contracts.md) and [developer experience and diagnostics](24-developer-experience-and-diagnostics.md).

**Precedence:**

1. Sections titled "Review Candidate 2 amendment".
2. Sections titled "Review Candidate 1 amendment".
3. Retained sketches, many now carrying inline SUPERSEDED notes.

The conforming everyday programming model is in [20](20-cross-system-invariants.md). An independent review has been completed and incorporated; remaining design work and experimental validation remain open.

Rjango is an async-first, type-safe Rust web framework design combining Django-style productivity with modern APIs, explicit security boundaries and structured tooling for humans and agents. It is **AI-native but never AI-dependent**.

This is the durable source of truth for the architecture designed so far. It preserves the earlier conversation through Part IV and incorporates the latest adversarial review and all 22 requested amendments. Part V is not yet fully captured. **This is a design snapshot, not an implemented framework, validated dependency set, or implementation roadmap.** No implementation planning or coding is authorized by this candidate.

## Status legend

| Status | Meaning |
| --- | --- |
| **REVIEW CANDIDATE 1** | Document-set maturity: first-review amendments integrated for independent review; neither frozen nor experimentally validated. |
| **REVIEW CANDIDATE 2** | Document-set maturity: independent-review direction integrated; neither frozen nor experimentally validated. |
| **PROPOSED** | Candidate design, illustrative syntax, alternative or unresolved question. |
| **SPEC-LOCKED** | Agreed architectural direction or requirement; changing it requires an explicit recorded decision. It need not be implemented or experimentally proven. |
| **VALIDATION REQUIRED** | The claim needs supporting prototype, test, benchmark, compatibility or operational evidence. May accompany SPEC-LOCKED. |
| **VALIDATED** | A specific claim has linked, reproducible evidence with tested versions, scope and limitations. It does not validate an entire subsystem by association. |
| **SUPERSEDED** | Replaced by a later decision; retain its history and identify the authoritative replacement. |

No runtime behavior or stack compatibility is marked VALIDATED by this capture. The earlier “PROVISIONALLY VALIDATED” ORM label described an ecosystem review, not completed prototypes. The target SeaORM 2.x / SQLx 0.9 combination still requires verification.

## Specifications

The settled principles below are SPEC-LOCKED; examples and explicitly open decisions remain PROPOSED. Evidence remains VALIDATION REQUIRED. Each page identifies its scope and unresolved choices.

| Specification | Design coverage / remaining boundary |
| --- | --- |
| [00 · Vision and principles](00-vision-and-principles.md) | The final 1.0 scope and release contract remain open. Early release milestones in historical discussion are superseded by the requirement to finish the design first. |
| [01 · Runtime architecture](01-runtime-architecture.md) | Public blocking-helper syntax, numeric cancellation/shutdown budgets, runtime tuning, and the full operational/performance contract require further design or validation. |
| [02 · Application Metadata Graph](02-application-metadata-graph.md) | Exact storage representation, macro/public trait ergonomics, extension evolution, rename-stable identities, and complete feature-node coverage remain open. Graph examples are structural sketches. |
| [03 · Models and ORM](03-models-and-orm.md) | The final locks and open decisions are preserved below. In particular, model macro form, `F`/`R` syntax, creation/builders, partial selections, loaded-state representation, and backend wrapping require evidence before syntax is frozen. |
| [04 · Migrations and schema management](04-migrations.md) | Migration file representation, exact commands, opaque field identities, rename UX, and detailed backend support remain open. A safety label is a review aid, not proof an operation is harmless. |
| [05 · Applications and modules](05-app-system.md) | Compile-time trusted plugins and macro/runtime compatibility requirements are specified; exact package/version policy remains open; app descriptor and lifecycle signatures are illustrative. |
| [06 · HTTP and routing](06-http-and-routing.md) | Exact routing/middleware syntax and full proxy/deployment guidance remain open. Examples express contracts, not a usable framework API. |
| [07 · Schemas and validation](07-schemas-and-validation.md) | Detailed validator extension APIs, schema evolution compatibility, and exact derive/macro syntax remain open. |
| [08 · API framework](08-api-framework.md) | Idempotency storage, versioning defaults, final resource syntax, SDK scope, and precise compatibility rules remain open. |
| [09 · Authentication](09-authentication.md) | Provider choices and detailed credential recovery, OIDC, passkey, MFA, session-revocation and key-lifecycle flows require fuller design. Inclusion in the specification is not availability. |
| [10 · Authorization and security](10-authorization-and-security.md) | Concrete policy syntax, transport/auth details, approval protocols, security threat model, and operational controls require further design and adversarial validation. |
| [11 · Admin](11-admin.md) | See the retained lock/open list. UI technology, detailed visual design, extension packaging, and final action syntax are not settled. |
| [12 · Jobs and scheduling](12-jobs-and-scheduling.md) | See the retained lock/open list. Backend defaults, scheduling leadership, payload evolution and exact APIs require validation and fuller design. |
| [13 · Caching](13-caching.md) | See the retained lock/open list. Backend defaults and distributed stampede/invalidation behavior remain open. |
| [14 · Storage and files](14-storage.md) | See the retained lock/open list. Public file-field types, lifecycle details, and provider capability contracts remain open. |
| [15 · Email and notifications](15-email-and-notifications.md) | See the retained lock/open list. Notification channels, templating choices, and backend guarantees remain open. |
| [16 · Realtime](16-realtime.md) | See the retained lock/open list. Distributed broker choice, reconnect guarantees, maximum revalidation windows and cross-node guarantees require validation. |
| [17 · Events and lifecycle](17-events-and-lifecycle.md) | See the retained lock/open list. Listener execution/failure defaults, cycle safeguards, and exact lifecycle signatures remain open. |
| [18 · Configuration and environments](18-configuration.md) | See the retained lock/open list. Live reload classes, secret refresh, feature flags, and configuration mutation need fuller design. |
| [19 · Quality and documentation requirements](19-quality-and-documentation.md) | Shared requirements; validation remains outstanding. |
| [20 · Cross-system invariants](20-cross-system-invariants.md) | Shared requirements; validation remains outstanding. |
| [21 · Remaining design inventory](21-design-backlog.md) | Outstanding design areas; no execution sequence. |
| [22 · Application Operations / Services](22-application-operations-and-services.md) | Shared business boundary for HTTP, Admin, Realtime, CLI, Jobs and MCP; ergonomics/evidence open. |
| [23 · Durability and message contracts](23-durability-and-message-contracts.md) | Core outbox, versioned envelopes, at-least-once delivery and recovery. PostgreSQL queue-as-outbox is the default; broker choices and evidence remain open. |
| [24 · Developer experience and diagnostics](24-developer-experience-and-diagnostics.md) | Teaching Rust through the framework, diagnostics contract, error model, dev loop budgets, introspection mode, shell replacement and version skew. Budgets are PROPOSED; evidence is open. |

## Supporting documentation

- [Architecture decision records](../adr/README.md): fifteen major settled decisions with rationale, boundaries and evidence gaps.
- [Coming from Django](../django/README.md): models, queries, migrations, apps, admin, application services, web and security.
- [AI and agent documentation](../ai/README.md): agent guide and MCP architecture/security scaffolding.
- [Preservation record and section crosswalk](preservation.md): every source section accounted for, with explicit supersessions.
- [Initial architecture history](history/initial-architecture.md): earlier rationale and developer-platform outlines; later topic specifications take precedence.

## Cross-system decisions to preserve

Tokio is the multi-thread async runtime; Axum/Tower supplies HTTP and middleware. PostgreSQL is first, with SeaORM as the ORM engine over SQLx (2.x/0.9 is a version target, not a lock) and a direct SQLx escape hatch.

Candidate 2 adds the following decisions:

- every entry point is an operation, with a progressive Tier 0/1/2 contract;
- deny-by-default policy, with scoped data access as the default;
- transactions own their connection, and commit consumes;
- durable effects go through the transaction;
- loaded relations are `Loaded<M>` values;
- commands are shielded from client disconnect;
- an unset environment means production;
- migrations carry expand/contract tags and gate readiness;
- diagnostics are API.

Database context is explicit. The AMG is immutable and deterministic, separate from runtime bindings and operational data. Production migrations are explicit. No property access hides network I/O. AI operation is optional and MCP permissions are separated from metadata visibility.

Jobs use at-least-once delivery with idempotency and transaction-aware dispatch. Domain events, durable jobs and ephemeral realtime messages have distinct contracts. Admin and public APIs require explicit exposure. Human docs, agent docs and meaningful tests are first-class feature requirements.

## Remaining unspecced or partially specced areas

The [remaining design inventory](21-design-backlog.md) tracks CLI/DX; dev server and hot reload; testing; observability; errors; human documentation architecture; Django transition; full AI/MCP; agent documentation; plugins; templates/forms/static assets; internationalization/timezones; advanced database features; deployment/operations; performance; compatibility/MSRV/deprecation; governance; reference apps; and the final 1.0 contract.

These are coverage gaps, not promised features or implementation tasks. Backend, syntax and policy questions inside already-written topics remain open as well.
