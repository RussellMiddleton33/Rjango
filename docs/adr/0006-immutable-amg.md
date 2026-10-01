# ADR 0006: Immutable, deterministic Application Metadata Graph

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/02-application-metadata-graph.md) · [Status legend](../specification/README.md#status-legend)

## Context

Models, migrations, APIs, admin, docs, checks and MCP need one structural description without separate drifting registries.

## Decision

Build the definition graph from code-generated descriptors and explicit registration through collect, normalize, resolve, validate, fingerprint and freeze stages. Keep the resulting graph immutable. Separate runtime bindings and operational state; exclude business records and secret values.

## Consequences and boundaries

Use stable identities, deterministic serialization, versioned metadata, typed registries and generic graph traversal. Hot reload rebuilds a new snapshot. Source edits produce graph changes; direct graph mutation does not define application behavior.

## Alternatives and rationale

Runtime reflection/global import side effects, independently maintained feature schemas and a mutable universal graph were not selected.

## Evidence still required

Determinism, duplicate/reference/cycle diagnostics, fingerprint stability, extension evolution, memory costs and schema compatibility require evidence.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.

## Candidate 1 clarification

The [architecture review](../reviews/architecture-review-candidate-1.md) retains this decision. Observed evidence is a separate overlay bound to a definition fingerprint, and projection-specific fingerprints replace a single application hash.

## Candidate 2 clarification

From the [independent review](../reviews/independent-review-candidate-1.md):

- Cross-type validation happens when the AMG is built (at startup or in metadata-only introspection mode), not at compile time.
- Applications register roots, and referenced schemas are collected transitively.
- Exposed identities are recorded in a checked-in `rjango.ids.lock`.
- Fingerprints detect declared change only, plus annotated policy bodies, never behavioural equivalence.

See [02](../specification/02-application-metadata-graph.md).
