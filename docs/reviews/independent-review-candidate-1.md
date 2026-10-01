# Rjango independent architecture review of Candidate 1

**Date:** 2026-09-30. **Baseline:** `ca0fab7614a19577a1c6f68ec487908234c941da`. **Reviewed:** all 50 documents, including history, preservation record, ADRs, Django guides and AI/MCP docs. **Audience lens:** experienced Django developers who do not yet know Rust. **Disposition:** every finding is addressed by a **Review Candidate 2 amendment** (see the [disposition crosswalk](#disposition-crosswalk)). Implementation planning and coding remain unauthorized.

Line references below are to the baseline commit; amendments since then shift line numbers. `spec/` means `docs/specification/`.

## Verdict

The foundations are sound and several teach Rust well: explicit database context, no hidden I/O, `get` returning an error versus `find` returning an `Option`, explicit loaded-state enums, the outbox and at-least-once contracts, and the split between database and wire types.

Candidate 1 was not yet a usable design for its primary audience:

1. **The everyday programming model was undefined.** The canonical examples contradicted the Candidate 1 amendments. Those amendments added a heavy Operation layer without a progressive path from simple to advanced use.
2. **The easiest way to read data skipped tenant and policy scoping.** Django developers would naturally write that unscoped code.
3. **Many items labelled "syntax, open" were semantic decisions.** In Rust, the API shape decides ownership, lifetimes, `Send`, cancellation and the compiler errors users see. Deferring them as "illustrative" understated what remained undecided.

## Critical

### C1 · The default data path bypassed tenant and policy scoping

**Evidence:**

- The canonical entry point was the unscoped `Venue::objects(&db)` (`spec/03:192`, `:201`).
- The ORM spec deferred tenancy (`spec/03:1634`), even though the amendment required scopes before every read, count, page, join and relation load (`spec/10:19-21`).
- Permission traits received an HTTP `request` (`spec/10:49-56`), so Jobs, MCP and CLI could not share them (`spec/22:11`).
- Routes without `#[permission]` had undefined behaviour.
- The flagship handler extracted `Identity` and never used it (`spec/20:494-517`).

**Correction:**

- Make the scoped database handle the default.
- Require an audited, distinctly named capability for unscoped access.
- Deny by default for every exposed surface.
- Use one policy object with `scope` and `check` forms.
- Use the `OperationContext` as policy input, not the HTTP request.

### C2 · No defined everyday path, and canonical examples broke the amendments

**Evidence:**

- `spec/22:11` makes every surface an adapter to an operation, but `spec/06:61-67` and `spec/03:199-207` showed handlers querying directly and returning models.
- `spec/20:494-534` broke five Candidate 1 rules:
  - it defined no operation;
  - its create was unscoped;
  - it used `publish_after_commit` naming against outbox semantics (`spec/23:9`);
  - it dispatched jobs through a non-transactional `&jobs` handle;
  - it extracted `Jobs` and `Identity` and never used them.
- The operation descriptor had about 11 facets and no defaults (`spec/22:17`).
- One tenant-scoped create needed about 15 concepts, against four in Django.

**Correction:**

- Add a progressive-complexity contract (Tier 0/1/2).
- Plain handlers become implicit operations with documented defaults.
- Replace the non-conforming example.

## High

### H1 · Transaction semantics were labelled syntax but decide correctness

**Evidence:** `spec/03:1001-1013`, `django/queries.md:80`, `spec/20:503`. Problems:

- A captured outer `db` inside the closure runs outside the transaction and can block on the transaction's own row lock until timeout.
- The closure committed implicitly, while `spec/22:26` requires an explicit commit with unknown-outcome handling.
- The closure hits higher-ranked lifetime and error-inference failures; `async |tx|` closures require Rust 1.85.
- Concurrent queries, streams, panics, savepoints and the `on_commit` equivalent were undefined.

**Correction:**

- `commit(self)` consumes the transaction; drop means rollback.
- Commit returns a typed outcome, including unknown.
- The transaction takes exclusive `&mut` access.
- The API uses a fixed `rjango::Error`.
- A dev-mode diagnostic fires when the pool handle is used while a transaction is open on the same task.

### H2 · Command cancellation defaults were missing

**Evidence:** Client disconnect drops handler futures (`spec/06:291-309`). Cancellation classes had no assigner or default (`spec/01:19`).

**Correction:**

- Commands run detached from the connection's lifetime, to completion or deadline, as tracked tasks.
- Queries cancel on disconnect.

### H3 · No compiler-diagnostics strategy; some promised checks were infeasible

**Evidence:**

- Cross-model "compile failures" (`spec/03:1753-1763`) are impossible from a single-type proc macro.
- Nothing addressed non-`Send` handler futures or Axum's opaque `Handler<_, _>` errors.
- Users implementing async traits would hit `Send` problems.
- Macro input errors erase generated items from rust-analyzer.

**Correction:** adopt a diagnostics contract:

- `#[diagnostic::on_unimplemented]` on user-facing traits and `#[diagnostic::do_not_recommend]` on internal blanket impls.
- A check for each handler argument, spanned to that argument.
- `Send` assertions with targeted messages.
- Annotated `async fn` extension points instead of user-implemented async traits.
- Error-tolerant macros.
- Rustdoc-visible generated items.
- trybuild snapshots as part of the API contract.
- Cross-type checks honestly reclassified as AMG/`rjango check` errors.

### H4 · The dev loop shaped the architecture but sat in the backlog

**Evidence:**

- Hot reload was a backlog item (`spec/21:12`).
- One model generated about eight types plus a SeaORM entity (`spec/03:289-297`).
- Every AMG consumer had to compile and run the app (`spec/02:1104-1149`).
- `rjango shell` had no design (`history:1032`).

**Correction:**

- Set rebuild and generated-code budgets.
- Add a metadata-only introspection mode with no external connections.
- Give crate-layout guidance.
- Give an explicit shell replacement story.

### H5 · No schema-compatibility model for rolling deploys

**Evidence:**

- Mixed binary and schema versions were never considered.
- Startup behaviour with pending or newer migrations was undefined.
- `rjango dev` auto-applied migrations (`spec/00:64`).

**Correction:**

- Classify every migration as expand, contract or breaking.
- Deploy-order plans.
- Compatibility windows recorded in the ledger.
- Readiness gating instead of production auto-apply.
- Dev auto-apply only against databases marked as development.

### H6 · The dynamic model-access layer was unspecified

**Evidence:** Admin (`spec/11:56-91`), query-string ordering (`spec/08:120-130`), historical models (`spec/04:415`) and MCP queries all need type-erased model access, while the AMG avoids runtime reflection (`spec/02:1007`).

**Correction:** a macro-generated `DynamicModel` contract that enforces scope and field policy inside the layer.

### H7 · Configuration layering let development settings reach production

**Evidence:**

- The base file applied to all profiles (`spec/18:80-94`).
- The historical template enabled MCP and set development in the base file (`history:1326-1339`).
- This contradicted `spec/10:593`.
- The default when no environment was set was unspecified.
- Production MCP data write was only a WARNING (`spec/10:682`).

**Correction:**

- An unset environment means production.
- Security-sensitive keys are not inherited from the base file into non-development profiles.
- Production MCP data write is CRITICAL.

### H8 · Security omissions on the new attack surfaces

- **WebSocket hijacking:** cookie-authenticated WebSocket/SSE handshakes did not require Origin validation.
- **SSRF:** webhooks and URL ingestion had no SSRF controls.
- **MCP prompt injection:** MCP outputs carrying attacker-controlled text had no untrusted-content handling.
- **`tests.run` is code execution:** it was presented as an ordinary scope (`spec/00:227`, `history:1220`).
- **Signed URLs:** they are bearer capabilities with no maximum TTL (`spec/14:190-201`).
- **Cache keys:** raw string keys (`spec/13:73-76`) contradicted the tenancy amendment.

### H9 · Loaded relations, the main Django mapping, were unspecified where users touch them

**Evidence:**

- The amendment removed relation fields (`spec/03:19`).
- The flagship `.with()` examples never showed the result type or how to read loaded data.

**Correction:**

- `.with()` returns `Loaded<M>`.
- Generated accessors are runtime-checked and return `Result<&[T], NotLoaded>`.
- No type-state generics.

### H10 · Model-to-API boilerplate had no ModelSerializer equivalent

**Evidence:**

- Schemas never derive automatically (`spec/07:19`).
- Resources were barely specified (`spec/08:57-80`).
- Conversions were written by hand (`spec/20:516`).
- Every schema was registered manually (`spec/02:1032-1037`, `:1167`).

**Correction:**

- Explicitly derived schema projections with compile-checked field lists.
- Root-only registration with transitive schema collection.
- Resources specified as generated operations.

## Medium

### M1 · Retained sketches contradicted amendments

| Conflict | Locations |
| --- | --- |
| `secrets.read` removed vs retained | review `:56`, `ai/mcp-architecture.md:31` vs `spec/10:574`, `spec/18:336` |
| RFC 9457 vs old envelope | `spec/06:21` vs `spec/06:431-447`, `spec/08:151-169` |
| `/users/:id` vs `/{id}` (Axum 0.8 rejects `:id`) | `spec/02:1502`, `:2458` vs `spec/06:61` |
| Handler returns model vs "never automatically a schema" | `spec/03:201` vs `spec/07:19`, `spec/02:1206` |
| Email dispatched after commit vs outbox | `spec/15:121-133` vs `spec/23:9` |
| `#[idempotent]` pluggable storage vs committed with effects | `spec/08:225-242` vs `spec/22:33` |
| Optimistic locking "later" vs version checks required | `spec/03:1674` vs `spec/22:24`, `spec/11:19`, MCP `:39` |
| Admin scoping "ideally" vs required | `spec/11:387` vs `spec/10:19` |
| Migration files shown as `.rs` vs open format vs stable IR | `spec/04:797` vs `:745`, `:19` |

### M2 · Fingerprints could not detect behavioural policy changes

`spec/02:27` claimed policy changes invalidate dependent contracts. Only declared metadata is hashed, and MCP stale-checks rely on that hash.

### M3 · The error model was deferred although everyday code depends on it

`spec/21:15` deferred it. Automatic `NotFound`→404 (`history:752`) leaks existence under scoping, and the spec had no domain-error or `?` conversion contract.

### M4 · Reusable apps versus static types

There was no swappable auth-user contract, and reusable crates could not reference host models (`spec/05:182-204`, `spec/09:62`).

### M5 · Identity stability

Route IDs were derived from function names (`spec/02:728` vs `:964`). Renames silently invalidated idempotency scopes and MCP previews (`spec/22:33`).

### M6 · Numeric types

- Framework counts conflicted with the `i64`-as-string default (`spec/07:23`, `spec/03:748`, `:1464`).
- Unsigned database types (`spec/02:502-508`) have no PostgreSQL mapping.

### M7 · Concept proliferation

- `Change<T>` (`spec/03:709`) duplicated `Patch<T>` (`spec/07:240`).
- Each model generated eight types.
- Macro forms were inconsistent.

### M8 · Version-skew claim

`spec/05:21` said skew is caught "before serving traffic". It actually surfaces as a confusing compile error.

### M9 · Hooks and validators could hide I/O; bulk operations bypassed invariants

- Hooks (`spec/03:1563-1580`, `spec/17:149-170`) and async validators (`spec/07:169`) could perform I/O.
- Bulk update and delete (`spec/03:735-771`) silently skipped hooks, `auto_now`, versions and audit.

### M10 · Jobs

- Dispatch used a non-transactional handle (`spec/12:141-145`).
- Job functions had no context or origin (`spec/12:55-61`).
- A relay was required even when the queue is PostgreSQL.

### M11 · Topic files never reconciled with Candidate 1

`spec/09`, `13`, `14` and `15` had no amendment. Password hashing on runtime threads and Django password-hash import were also unaddressed.

### M12 · ADR gaps

- There was no ADR for Operations or the outbox.
- ADR 0004 locked unverified major versions.
- ADRs 0006, 0007 and 0009 contained a pasted, partly irrelevant clarification paragraph.

## Low

- **L1.** Superseded examples were not marked where they appeared.
- **L2.** `#[deprecated(since, remove, replacement)]` (`spec/08:282`) collides with Rust's built-in attribute.
- **L3.** `first`/`last` had no ordering rule, and `require`/`one` had no multiple-row semantics.
- **L4.** A model without a primary key produced only a warning (`spec/02:1204`).
- **L5.** "v0.1" scoping language remained (`spec/02:344`, `spec/03:580`).
- **L6.** The dev north star showed MCP and migration auto-apply without guards (`spec/00:60-71`).

## Do the abstractions teach Rust?

| Abstraction | Effect on learning | Direction adopted |
| --- | --- | --- |
| `get` returns an error, `find` returns `Option` | Teaches `Result` vs `Option` | Keep |
| `edit()…save()` consuming the value | Teaches moves | Keep; extend to `commit(self)` |
| `Patch`, loaded-state enums | Teaches enums and exhaustive `match` | One `Patch<T>` type |
| Explicit `Db` parameter | Teaches capability-by-parameter | Default handle is scoped |
| Transaction closure | Hid lifetimes until they failed badly | Guard primitive plus an async-closure convenience |
| `Venue::F` / generated types | Invisible unless documented | Rustdoc-visible, error-tolerant macros |
| User-implemented async traits | Exposed `Send` complexity early | Annotated `async fn`s |
| Non-serialisable `Secret<T>` | Teaches trait bounds as enforcement | Keep; macro-generated `Debug` for sensitive fields |

## Requirements versus illustration versus unproven claims

**Moved from "illustrative syntax" to architecture decisions in Candidate 2:**

1. Transaction ownership and the commit model.
2. Loaded-relation representation.
3. How handlers relate to operations, and the context type.
4. Dispatch target.
5. Error type and conversions.
6. Default database-handle scoping.
7. Extension model.
8. Migration artifact format.

**Correctly locked already:** explicit context, no hidden I/O, immutable AMG, outbox with at-least-once delivery, explicit migrations, MCP capability split, separate database and wire types.

**Unproven claims now relabelled VALIDATION REQUIRED or corrected:**

- The SeaORM 2.x / SQLx 0.9 pairing.
- Cross-model compile-time checks.
- Policy fingerprint closure.
- Skew detected before serving traffic.
- Automatic JOIN/batch/data-loader choice.
- "No `Arc`/locks for developers".
- Admin derived from the AMG without a dynamic layer.
- The MCP token-saving figures.

## Validation-required questions

1. Can Django developers with no Rust build a tenant-scoped CRUD endpoint with admin and tests from the tutorial alone? Measure concepts, compile errors and time.
2. What is the incremental rebuild time for a 30-model, 100-route reference app against the proposed budgets?
3. Do the transaction guard and `atomic` convenience compile on the chosen MSRV with clean inference and good errors for the ten most common mistakes?
4. Can macros detect non-`Send` handler futures and point at the offending `.await`?
5. What do error messages and the runtime cost look like for runtime-checked `Loaded<M>` accessors on nested loads?
6. Which policies translate to SQL scopes? Does a property test show that scope and check agree?
7. Can SeaORM 2 and SQLx 0.9 share one transaction so raw SQL runs inside ORM transactions?
8. How do Hyper and Axum cancel on HTTP/1.1 and HTTP/2 disconnects, and what does shielding commands cost?
9. Across the matrix of old and new binaries against old and new schemas under SeaORM column lists, which combinations break?
10. Does rust-analyzer keep completing `Venue::F` and `Venue::R` while the macro input contains an error?
11. What throughput does a PostgreSQL queue acting as the outbox achieve, compared with a relay?
12. Can Django password hashes be imported and rehashed on login?
13. Is RLS with transaction-local tenant settings correct on pooled connections?

## Disposition crosswalk

After the amendments were written, a whole-system seam review on a different model checked them as a set. It found 11 cross-file defects and several nits in the new text, all corrected:

- the Tier 0 example was really Tier 1;
- `NewVenue` omissions were undefined;
- policy forms had no signatures;
- unknown-commit errors had two names;
- `SET LOCAL` outside a transaction is a no-op;
- the SSE Origin rule rejected same-origin requests;
- the claim that hooks cannot perform I/O was too strong;
- the renamed-crate path claim was backwards;
- the derived-schema mechanism was unstated;
- integer Django user IDs need remapping to `UserId`;
- `Ctx` was not tied to `OperationContext`.

| Finding | Candidate 2 amendment location |
| --- | --- |
| C1 | [10](../specification/10-authorization-and-security.md), [03](../specification/03-models-and-orm.md), [22](../specification/22-application-operations-and-services.md), [ADR 0012](../adr/0012-scoped-data-access-and-default-deny.md) |
| C2 | [22](../specification/22-application-operations-and-services.md), [20](../specification/20-cross-system-invariants.md), [06](../specification/06-http-and-routing.md), [ADR 0010](../adr/0010-application-operations.md) |
| H1 | [03](../specification/03-models-and-orm.md), [Django queries](../django/queries.md), [ADR 0013](../adr/0013-transaction-ownership.md) |
| H2 | [01](../specification/01-runtime-architecture.md), [06](../specification/06-http-and-routing.md), [22](../specification/22-application-operations-and-services.md) |
| H3 | [24](../specification/24-developer-experience-and-diagnostics.md), [03](../specification/03-models-and-orm.md), [ADR 0015](../adr/0015-diagnostics-and-developer-loop.md) |
| H4 | [24](../specification/24-developer-experience-and-diagnostics.md), [02](../specification/02-application-metadata-graph.md), [21](../specification/21-design-backlog.md) |
| H5 | [04](../specification/04-migrations.md), [17](../specification/17-events-and-lifecycle.md) |
| H6 | [03](../specification/03-models-and-orm.md), [11](../specification/11-admin.md), [08](../specification/08-api-framework.md), [04](../specification/04-migrations.md) |
| H7 | [18](../specification/18-configuration.md), [10](../specification/10-authorization-and-security.md) |
| H8 | [16](../specification/16-realtime.md), [15](../specification/15-email-and-notifications.md), [14](../specification/14-storage.md), [13](../specification/13-caching.md), [MCP](../ai/mcp-architecture.md), [10](../specification/10-authorization-and-security.md) |
| H9 | [03](../specification/03-models-and-orm.md), [Django queries](../django/queries.md), [ADR 0014](../adr/0014-loaded-relations.md) |
| H10 | [07](../specification/07-schemas-and-validation.md), [08](../specification/08-api-framework.md), [02](../specification/02-application-metadata-graph.md), [05](../specification/05-app-system.md) |
| M1 | Inline supersession notes in 02, 03, 04, 06, 08, 10, 11, 15, 18, 20 and history |
| M2 | [02](../specification/02-application-metadata-graph.md), [MCP](../ai/mcp-architecture.md) |
| M3 | [24](../specification/24-developer-experience-and-diagnostics.md), [06](../specification/06-http-and-routing.md) |
| M4 | [05](../specification/05-app-system.md), [09](../specification/09-authentication.md) |
| M5 | [02](../specification/02-application-metadata-graph.md) |
| M6 | [07](../specification/07-schemas-and-validation.md), [02](../specification/02-application-metadata-graph.md) |
| M7 | [03](../specification/03-models-and-orm.md), [07](../specification/07-schemas-and-validation.md) |
| M8 | [05](../specification/05-app-system.md) |
| M9 | [03](../specification/03-models-and-orm.md), [17](../specification/17-events-and-lifecycle.md), [07](../specification/07-schemas-and-validation.md) |
| M10 | [12](../specification/12-jobs-and-scheduling.md), [23](../specification/23-durability-and-message-contracts.md), [ADR 0011](../adr/0011-transactional-outbox.md) |
| M11 | [09](../specification/09-authentication.md), [13](../specification/13-caching.md), [14](../specification/14-storage.md), [15](../specification/15-email-and-notifications.md) |
| M12 | [ADR index](../adr/README.md), [ADR 0004](../adr/0004-seaorm-sqlx.md) |
| L1–L6 | Inline notes; [03](../specification/03-models-and-orm.md) (L3, L4), [08](../specification/08-api-framework.md) (L2), [02](../specification/02-application-metadata-graph.md) (L5), [00](../specification/00-vision-and-principles.md) (L6) |
