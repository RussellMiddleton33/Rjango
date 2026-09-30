# Quality and documentation requirements

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III, Part IV.

The design requires meaningful tests and excellent documentation for humans and agents to ship with every feature. This file records quality requirements; it does not report completed tests.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

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
