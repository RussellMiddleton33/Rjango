# ADR 0001: Tokio multi-thread runtime

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/01-runtime-architecture.md) · [Status legend](../specification/README.md#status-legend)

## Context

Rjango needs async I/O, safe concurrency, multi-core scheduling and one application model for web requests and workers.

## Decision

Use Tokio as the runtime foundation, multi-threaded by default. Requests use lightweight async execution with shared, concurrency-safe resources. Isolate blocking/CPU work explicitly; coordinate timeout, cancellation and graceful shutdown through framework APIs.

## Consequences and boundaries

Rjango does not build an async runtime. A request must not require its own OS thread, and low-level scheduler/locking details should not dominate common application code.

## Alternatives and rationale

A custom runtime or a primarily synchronous request/worker model would duplicate mature infrastructure and undermine the async-first design. Other runtimes were not experimentally compared in this conversation.

## Evidence still required

Cancellation, blocking isolation, shared resource safety, graceful draining and load behavior require reproducible validation.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
