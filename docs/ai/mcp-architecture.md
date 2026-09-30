# MCP architecture and security boundaries

[AI documentation](README.md) · [AMG](../specification/02-application-metadata-graph.md) · [Security](../specification/10-authorization-and-security.md)

**Decision:** SPEC-LOCKED boundaries. **Interface details:** PROPOSED. **Evidence:** VALIDATION REQUIRED.

## Purpose and planes

Expose versioned, structured understanding of the application to agents without making application execution dependent on AI. Definition metadata describes structure; runtime bindings resolve executable behavior; operational resources report state. Business records and secret values are not graph metadata. Public, Internal, DevelopmentOnly and Restricted exposure classifications govern what may be published.

Intended resources describe apps, models, schemas, routes, permissions, migration state, jobs, configuration schemas, docs and diagnostics. Use stable IDs, deterministic serialization and explicit metadata schema versions. Production source paths and sensitive structures may require filtering. Exact resource URIs, tool names and protocol schemas are still design proposals.

## Capability separation

| Capability | Boundary |
| --- | --- |
| Inspect metadata | Filter structural visibility by identity, environment and exposure policy. |
| Read application data | Require independent data-read authorization and applicable object/tenant policy. |
| Modify data | Require separate write permissions, validation and audit. |
| Generate migrations | Produce reviewable artifacts; generation is not permission to apply them. |
| Apply migrations | Independently authorize environment, operation risk and execution. |
| Run commands, jobs or admin actions | Independently expose and authorize each capability; registration in one interface does not automatically expose it through MCP. |
| Read secrets | Separate explicit capability; normal introspection reveals schema/provenance and redacted state only. |
| Perform destructive actions | Enforce risk/approval policy and retain audit evidence. |

## Identity, transport and production

Agents use explicit identities/service accounts and scoped permissions. Treat authenticated MCP transport as a security boundary with resource limits, audit, and explicit tool policy. Production MCP is disabled by default and enabled deliberately with constrained access. Read-only metadata inspection must never imply database mutation or arbitrary shell execution. A human's authority is not automatically delegated to an agent.

## Reviewable mutation

The graph is immutable. Changes occur through source edits, metadata rebuild/validation, reviewable diffs and the separately authorized execution paths. Destructive or production changes require their applicable policy checks; no tool may silently widen its capability. Data-access tooling remains distinct from model introspection and query explanation.

## Remaining design

Complete transport/authentication mechanisms, capability schemas, confirmation UX, remote trust model, resource quotas, audit retention, version negotiation and Django translation resources remain unspecced. The graph's original MCP sections and ORM data-access boundaries are retained in their canonical topic documents.
