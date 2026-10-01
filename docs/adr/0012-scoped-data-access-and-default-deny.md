# ADR 0012: Scoped data access by default and deny-by-default policy

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidate 2.

[ADR index](README.md) · [Authorization](../specification/10-authorization-and-security.md) · [ORM](../specification/03-models-and-orm.md) · Refines [ADR 0005](0005-explicit-database-context.md)

## Context

Candidate 1 required tenant and policy scoping on every read and write path, but the canonical query entry point was unscoped, the ORM deferred tenancy, and routes without permissions had undefined exposure. The easiest code was the insecure code, and Django habits (open views, unscoped managers) would reproduce it.

## Decision

- **Scoped handle by default.** The default database handle (`ctx.db()` or the `Db` extractor) is scoped to the actor and tenant. Model tenant keys and default policies apply automatically to every QuerySet terminal, relation load, stream and bulk operation.
- **Unscoped access is a capability.** `SystemDb` requires a declared, audited capability.
- **Deny by default.** Every exposed surface declares a policy or the explicit marker `public`; missing declarations fail AMG validation.
- **One policy, two forms.** A policy has a `scope` predicate and a `check` object decision, and receives the operation context, not the HTTP request.

## Consequences and boundaries

Explicit context (ADR 0005) is retained; explicit does not mean unscoped. Tenant keys are set from context, never from payloads. Row-level security is optional defense in depth with transaction-local settings. Policies that cannot produce a scope cannot protect collections.

## Alternatives and rationale

- **Opt-in scoping:** rejected, because it is easy to forget.
- **Task-local ambient tenant:** rejected, because it is hidden state, conflicts with ADR 0005 and is hard to test.
- **RLS as the only mechanism:** rejected, because of portability, poor diagnostics and pooled-session hazards.

## Evidence still required

SQL translation coverage for policies; scope/check agreement property tests; overhead; RLS interaction; developer comprehension of `public` versus policy.
