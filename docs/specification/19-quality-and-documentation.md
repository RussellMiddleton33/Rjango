# Quality and documentation requirements

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III, Part IV.

The design requires meaningful tests and excellent documentation for humans and agents to ship with every feature. This file records quality requirements; it does not report completed tests.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Strengthened evidence requirements

Retain 100% practically testable framework-owned line coverage with justified exclusions; meaningful branch/mutation/adversarial tests remain independently required. Evidence must include:

- Equivalent outcomes/denials across all six operation adapters and no Admin bypass.
- Provenance/completeness, stale overlays, deterministic projection hashes and unrelated-change isolation.
- Relation unloaded/empty/null/value states, partial-row safety and no hidden I/O.
- Real PostgreSQL mixed phases, concurrent migrators/lock loss, crash at every DDL/ledger boundary, partial objects, checksums, IR upgrades and historical replay.
- Outbox commit/rollback, relay crashes before/after acknowledgement, duplicates, poison messages, dedup expiry, ordering/retention; isolated databases and multiple connections for commit/locking tests rather than outer rollback fixtures.
- Old/new producer/consumer and realtime protocol matrices, payload upgrades, unknown versions, replay and revoked origins.
- Cancellation at acquisition/query/commit/dispatch, unknown outcome reconciliation, pool recovery, lease expiry, shutdown deadlines and saturated-resource backpressure.
- Tenant leakage through counts/exports/joins/caches/streams, delegation attenuation/revocation, MCP token/audience rejection, stale fingerprints and idempotency conflicts.
- Problem Details redaction/status/type stability, TypeScript numeric/date precision round trips, Secret<T> serialization rejection/leak tests.
- Macro/runtime skew/features, plugin trust boundaries, replica lag/failover and no accidental cross-database atomicity promise.

Fake backends cannot establish real durability or backend compatibility. Evidence records dependency versions, environment, fault injection, result and limits. No prototype/test result is claimed by this documentation amendment.

### Shared canonical human/AI source

One versioned semantic source feeds human/agent reference, CLI explain, error docs and MCP documentation where practical. Preserve IDs, versions, evidence status and source links. Human tutorials add pedagogy without redefining contracts; generated references cannot replace tutorials/how-to/explanation. Drift checks compare schemas/errors/commands; eventual runnable examples compile against supported versions. Until then, label design illustrations.


## Review Candidate 2 amendment

**Status:** evidence requirements added from the [independent review](../reviews/independent-review-candidate-1.md). No result is claimed.

Additional evidence:

- **Developer usability.**
  - A recorded study of Django developers who do not know Rust completing the Tier 0 tutorial, recording concepts met, compiler errors met and time taken ([22](22-application-operations-and-services.md)).
  - A compile-fail message suite reviewed by Rust newcomers.
  - rust-analyzer completion under macro input errors.
  - Rebuild-time budgets on the reference application ([24](24-developer-experience-and-diagnostics.md)).
- **Default-deny and scoping.**
  - Every registered surface has a policy or `public`.
  - Every QuerySet terminal, relation load, bulk operation, stream, Admin dynamic query and resource filter is scoped.
  - Using `SystemDb` without declaration fails.
  - Scope/check agreement property tests per policy.
- **Transactions.**
  - `RJG-DB-TX-OUTSIDE` detection.
  - Compile-fail tests for concurrent use of `&mut tx` and use after commit.
  - The commit-outcome-unknown path.
  - Rollback on drop, panic and cancellation.
- **Cancellation.** Command shielding across HTTP/1.1 and HTTP/2 disconnects; Query cancellation; shielded-task drain during shutdown.
- **Rolling deploys.** The old/new binary × old/new schema matrix; readiness gating outside the compatibility window; development-only auto-apply.
- **Security.**
  - WebSocket/SSE Origin rejection.
  - SSRF egress policy against private, metadata, IPv6-mapped and redirect/DNS-rebinding cases.
  - Signed-URL TTL limits.
  - MCP untrusted-content labelling and refusal of code-execution capabilities in production.
  - Unset environment resolves to production.
  - Base-file security keys are ignored outside development.
- **Data access.** `Loaded<M>` NotLoaded/LoadedEmpty/LoadedNull/value accessors; DynamicModel field and scope enforcement; `bulk_update` audit/version/`auto_now` semantics; Django password-hash import and rehash.

Test fixtures that wrap each test in an outer rollback cannot verify commit, outbox or locking behaviour; those use isolated databases, as the Candidate 1 amendment requires.

## Open decisions and interpretation

Testing-framework design, documentation architecture/tooling, versioned agent resources, and the full Django curriculum remain partially specified.

## Conversation-wide quality contract

The framework targets **100% line coverage of practically testable framework-owned code**, with meaningful branch coverage wherever practical, especially security, permissions, migrations, validation, parsing, configuration, and errors. Exclusions require a documented technical reason, owner, scope, and explanation of why a meaningful test is not possible. Casual coverage ignores are unacceptable. This is a requirement, not a current coverage report.

Coverage alone is insufficient. Public APIs need positive, negative, edge and failure cases; macros need compile-pass and compile-fail diagnostics; database behavior needs real PostgreSQL integration coverage; concurrency and cancellation need stress and resource-cleanup checks. Metadata, fingerprints, schema diffs, migrations and generated outputs require deterministic golden/snapshot checks. Property, fuzz, mutation, adversarial security, compatibility/semver and performance regression testing apply where meaningful. A reported bug requires a regression test.

Every feature ships implementation, appropriate tests, human documentation, agent documentation, a runnable example, and changelog/API-impact information together. Documentation examples must eventually compile or run against their stated version. This capture preserves sketches before such validation exists.

Architecture decisions must record the problem, constraints, alternatives, tradeoffs, evidence and reasons for rejection. A failed experiment should change the design rather than be hidden to defend an earlier preference. Experimental claims require reproducible results and tested versions.

## Human and agent documentation as product requirements

Installation and first use should be approachable out of the box. Human documentation needs tutorials, task guides, explanations and reference, with clear prerequisites, complete examples, troubleshooting and stable error-code links. Agents need concise version-aware contracts, metadata schemas, conventions, command/tool permissions, examples, and known limitations. Both should derive from shared definitions where possible and must not drift.

The documentation should support three entry paths: new backend developers, experienced Rust developers, and developers coming from Django. Every significant feature with an obvious Django counterpart needs the counterpart and the point where the analogy stops. A Django concept coverage index must distinguish designed, implemented, different and unsupported capabilities. No design example should masquerade as shipped functionality.

See [Django guide scaffolding](../django/README.md), [agent guide](../ai/agent-guide.md), and the [remaining design inventory](21-design-backlog.md).

<!-- Source: iii section 112. -->
## Human Documentation Requirements for These Systems

Every subsystem ships documentation containing:

```
5-minute quick start

concept explanation

API reference

production guidance

common recipes

failure modes

security considerations

performance considerations

Django comparison

troubleshooting

complete runnable example
```

Examples are CI-tested.

---

<!-- Source: iii section 113. -->
## AI Documentation Requirements

Every subsystem exposes version-aware structured knowledge describing:

```
concepts
public APIs
constraints
examples
anti-patterns
security rules
error codes
migration guidance
Django equivalents
```

An agent must be able to ask:

```
How do I add a Rjango app?

How do I create a data migration?

Can this route accept 100MB?

How do I require object-level permission?

Is this endpoint protected against CSRF?

What's the Rjango equivalent of DRF Serializer?
```

without searching arbitrary blog posts.

---

<!-- Source: iii section 116. -->
## Quality Standard Applied to Every Section

Nothing represented here should be considered proven merely because this specification says it is desirable.

For each testable behavior, the eventual validation standard remains:

```
100% practically testable Rjango-owned line coverage

meaningful branch coverage

compile-pass tests

compile-fail tests

unit tests

integration tests

property tests

fuzz tests

mutation tests

concurrency tests

security/adversarial tests

documentation tests

compatibility tests

performance regression tests
```

Every bug becomes a regression test.

Every user-facing feature receives human and AI documentation.

Every appropriate feature receives a Django migration/explanation page.

<!-- Source: iv section 124. -->
## Documentation Architecture Across These Systems

Every subsystem documented through four layers:

### Tutorial

Example:

```
Send your first background job
```

### How-to

Example:

```
Use Redis as your production cache
```

### Concepts

Example:

```
Why Rjango jobs are at-least-once
```

### Reference

Example:

```
RetryPolicy API
```

This mirrors a strong documentation architecture instead of mixing every audience onto one page.

---

<!-- Source: iv section 125. -->
## AI Documentation

AI-facing docs need explicit semantic metadata.

Example job documentation:

```
{
  "concept": "job retries",
  "framework_version": "1.x",
  "guarantee": "at-least-once",
  "requires_idempotency": true,
  "related": [
    "transactional outbox",
    "retry policy",
    "dead-letter jobs"
  ]
}
```

Agents should retrieve conceptual rules, not merely source snippets.

---

<!-- Source: iv section 139. -->
## Fault-Injection Standard

These systems especially require deliberate failure simulation.

Examples:

```
Redis dies during request

worker dies after external side effect

S3 accepts partial upload then connection drops

SMTP provider times out

PostgreSQL commits but queue broker is unreachable

realtime broker disconnects

shutdown begins while job is running
```

A framework that works only on happy paths is not production infrastructure.
