# ADR 0009: AI-native, not AI-dependent; explicit MCP boundaries

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/10-authorization-and-security.md) · [Status legend](../specification/README.md#status-legend)

## Context

Agents need structured understanding and safe tools while applications must remain usable without AI and without accidentally broadening access.

## Decision

Make AI/MCP a first-class interface to shared application structure while requiring no model for ordinary application operation. Separate metadata access, business-data read/write, migration generation/application, command execution, secret access and destructive actions. Use explicit identities, scoped permissions, risk/approval policy and audit; disable production MCP by default.

## Consequences and boundaries

Metadata inspection never grants data access or authority to execute. Admin actions and jobs do not automatically become MCP tools. Source-changing workflows produce reviewable edits and rebuild metadata rather than mutating the graph. Production exposure must be deliberately constrained.

## Alternatives and rationale

An unrestricted administrative agent interface or application execution that requires an AI model conflicts with the design. Exact MCP transport, schemas and approval mechanics are still open.

## Evidence still required

Capability isolation, environment defaults, authentication, authorization, secret redaction, audit and abuse scenarios need validation.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.

## Candidate 1 clarification

The [architecture review](../reviews/architecture-review-candidate-1.md) retains this foundational decision and records its strengthened boundaries. Observed AMG evidence remains a separate overlay; migrations use phase/step recovery and checksums; core MCP never exposes raw secrets and defaults to zero production application-data access. These amendments supersede conflicting earlier sketches, with validation still required.
