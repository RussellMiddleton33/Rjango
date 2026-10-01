# ADR 0010: Application operations as the universal entry contract

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidates 1 and 2.

[ADR index](README.md) · [Canonical specification](../specification/22-application-operations-and-services.md) · [Independent review](../reviews/independent-review-candidate-1.md)

## Context

HTTP, Admin, Realtime, CLI, Jobs and MCP must share authorization, transaction, durability and audit semantics. Candidate 1 introduced shared operations but left plain handlers and the everyday developer path undefined. Its canonical examples bypassed the contract.

## Decision

Every entry point is an operation. Annotated handlers, admin actions, channel commands, CLI commands, jobs and MCP tools are implicit operations whose descriptors are synthesized from signatures and documented defaults. Named operations are reused across adapters by explicit exposure. A three-tier progressive-complexity contract defines what developers write at each stage. Tier 0 targets six concepts or fewer for a tenant-scoped create endpoint.

## Consequences and boundaries

No adapter path bypasses policy, scoping, cancellation or audit. Escape-hatch Axum routes are reported as unmanaged. Descriptor facets have defaults; developers declare them only at Tier 2. Operations are a semantic contract, not CQRS or event sourcing.

## Alternatives and rationale

- **Handlers as a separate, lighter path:** rejected, because it creates an authorization bypass.
- **Mandatory explicit descriptors:** rejected, because of the boilerplate burden on the primary audience.

## Evidence still required

Tier 0 usability with Django developers who do not know Rust; descriptor inference accuracy; implicit-operation overhead; adapter parity tests.
