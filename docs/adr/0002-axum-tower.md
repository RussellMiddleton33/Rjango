# ADR 0002: Axum HTTP and Tower middleware

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/06-http-and-routing.md) · [Status legend](../specification/README.md#status-legend)

## Context

HTTP routing and middleware should use mature Rust infrastructure while presenting one coherent application model.

## Decision

Build HTTP services and routing on Axum and compose middleware through Tower. Rjango owns approachable handlers, typed request context, route metadata, names, policy, diagnostics and exposure rules.

## Consequences and boundaries

Keep explicit escape hatches while maintaining AMG metadata coherence. Define deterministic middleware order and preserve streaming, cancellation and backpressure.

## Alternatives and rationale

Inventing a new HTTP stack would spend effort outside Rjango’s intended value. The conversation did not record comparative benchmarks against other HTTP frameworks.

## Evidence still required

Typed handler integration, middleware order, streaming/cancellation, metadata extraction and performance remain unverified.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
