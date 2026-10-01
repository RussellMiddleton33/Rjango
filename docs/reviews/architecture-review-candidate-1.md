# Rjango 1.0 adversarial architecture review — Candidate 1

> **Superseded as the latest review:** the [independent review of Candidate 1](independent-review-candidate-1.md) produced Review Candidate 2. Its amendments take precedence where they conflict with this disposition.

**Date:** 2026-09-30. **Verdict:** VIABLE, AMENDMENTS INCORPORATED AT SPEC LEVEL; VALIDATION REQUIRED. **Master status:** REVIEW CANDIDATE 1, not frozen. **Implementation planning/coding:** not authorized by this review.

## Source and authority

This durable disposition incorporates the latest review in the referenced conversation “Rust Django Equivalent Explained” (conversation ID 6abd4192-6288-83ea-aa1b-4d7f1c6480e9), retrieved on this date, and the user's explicit 22-amendment instruction. The retrieval truncates individual messages at 20,000 characters; it supplies the review through the beginning of MCP idempotency, not a complete archival transcript. The remaining requirements are grounded in the user's current instruction. Part V is also truncated and is not represented here as fully preserved. No external dependency compatibility or protocol revision claim from the conversation is treated as experimentally validated.

The current master and topic amendments are authoritative. Earlier sketches remain historical where explicitly superseded; SPEC-LOCKED describes agreed direction, never proof of implementation. This document is an architecture assessment and disposition, not a delivery plan.

## Verdict by subsystem

The Rust/Tokio/Axum/Tower direction, PostgreSQL-first strategy, Rjango-owned public API, explicit I/O, immutable AMG, separate model/schema and identity/permission contracts, explicit production migrations and optional AI interfaces survive the conceptual review. No fatal architectural flaw was identified; this is not a proof that the proposed stack works.

| Area | Verdict and adopted correction |
| --- | --- |
| Application architecture | Shared Operations / Services close the largest structural gap between six execution surfaces. |
| Runtime | Cancellation outcome classification, shutdown deadlines and universal resource bounds required. |
| AMG | Provenance/completeness and projection hashes required; observed facts stay outside definitions. |
| ORM | Row state/relationship metadata/loaded values separated; public ergonomics unvalidated. |
| Migrations | Major correction: mixed phase/step transactions, partial-state recovery, locks/checksums/stable IR. |
| Jobs/events | Core outbox and versioned envelopes; at-least-once delivery remains explicit. |
| Authorization | Query scoping, tenant isolation and attenuated delegation required everywhere. |
| Realtime | Protocol evolution and ongoing permission revalidation required. |
| MCP | Separate local/remote trust; no raw secrets or broad default production data access. |
| HTTP/clients | Problem Details plus separate database/wire types and precision-safe client mapping. |
| Admin/plugins | Explicit mutation modes; trusted compile-time Rust extensions with no sandbox claim. |
| Multiple databases | No cross-database ACID; explicit replica consistency mechanisms. |
| Quality/docs | Shared canonical source and failure-oriented evidence matrix strengthened. |
| 1.0 scope | High remaining breadth risk; Candidate 1 is not scope completion or freeze. |

## Amendment crosswalk

| Request | Durable specification |
| --- | --- |
| 1 · Operations/Services | [22](../specification/22-application-operations-and-services.md) |
| 2–3 · AMG evidence/projection fingerprints | [02](../specification/02-application-metadata-graph.md) |
| 4 · Relations separate from stored/loaded state | [03](../specification/03-models-and-orm.md) |
| 5 · Migration phases/recovery/locks/checksums/IR | [04](../specification/04-migrations.md) |
| 6–7 · Outbox/envelopes/protocol evolution | [23](../specification/23-durability-and-message-contracts.md), [12](../specification/12-jobs-and-scheduling.md), [16](../specification/16-realtime.md) |
| 8 · Cancellation/shutdown | [01](../specification/01-runtime-architecture.md), [17](../specification/17-events-and-lifecycle.md) |
| 9–10 · Query scopes/tenancy/actor/subject/delegation | [10](../specification/10-authorization-and-security.md) |
| 11 · Realtime revalidation | [16](../specification/16-realtime.md) |
| 12 · MCP auth/secrets/data/preconditions/idempotency | [MCP](../ai/mcp-architecture.md) |
| 13 · RFC 9457 | [06](../specification/06-http-and-routing.md), [08](../specification/08-api-framework.md) |
| 14 · Wire/database types and TypeScript precision | [07](../specification/07-schemas-and-validation.md) |
| 15 · Admin mutation modes | [11](../specification/11-admin.md) |
| 16–17 · Plugins/macro-runtime contract | [05](../specification/05-app-system.md) |
| 18 · Cross-database/replica restrictions | [03](../specification/03-models-and-orm.md) |
| 19 · Bounds/backpressure | [01](../specification/01-runtime-architecture.md) |
| 20 · Secret nonserialization | [18](../specification/18-configuration.md) |
| 21–22 · Canonical docs/testing | [19](../specification/19-quality-and-documentation.md) |

## Explicit supersessions

Migration-wide atomicity flags no longer express the full execution contract. Relationship declaration sketches do not lock fields into persisted rows. One application hash cannot replace projection hashes. After-commit dispatch without durable intent is insufficient. Metadata effect graphs are not assumed exhaustive. Raw core secrets.read is removed; broad production database reads are not default capabilities. Plugins are trusted code rather than security-isolated extensions. Replica routing offers no unconditional read-your-writes guarantee. Historical cross-system-review-pending language refers to the earlier capture; this first review is complete, independent review and evidence remain open.

## Remaining risks and unresolved validation questions

- **Stack and ergonomics:** Does the selected SeaORM/SQLx pairing support shared pools/transactions and the desired macros without exposing unstable internals? Which loaded relation and operation-context types remain ergonomic across async lifetimes? No compatibility version has been proven here.
- **Migration longevity:** What stable IR/version/checksum rules survive years of upgrades? How do custom data code, lock/session loss and ambiguous DDL completion recover without unsafe automatic repair?
- **Durability:** Which queue/broker/relay design satisfies retention, leasing, ordering and dedup windows under rolling upgrades? How are unknown commits and external effects reconciled?
- **Policy:** Which authorization predicates can translate to database scopes? How do tenant constraints and raw SQL stay safe? What exact audit fail-closed policy and delegation revocation mechanism are practical?
- **Realtime/MCP:** What bounded revocation window is acceptable? Which remote MCP protocol/auth revision and resource/token validation rules will be supported? How do retry keys and relevant fingerprints remain valid across deploys? Pin and verify normative versions before claiming conformance.
- **Resource budgets:** What measured defaults meet realistic workloads without deadlocks, starvation, unbounded buffers or unacceptable shutdown latency?
- **Compatibility/types:** What macro/runtime version policy, MSRV, wire precision/timezone rules and client support windows can be guaranteed?
- **Scope:** Developer-platform, operations, ecosystem and release-contract gaps in [21](../specification/21-design-backlog.md) remain. The retrieved Part V excerpt is not a substitute for a complete durable specification of those areas.

## Review acceptance boundary

All 22 requested corrections are now represented as design requirements, with subsystem ownership and open evidence identified. Independent review should challenge interactions, missing failure states, misleading historical examples and scope promises. Freeze requires resolving design-critical questions and recording evidence for any claim marked VALIDATED. Neither this verdict nor a documentation commit authorizes implementation planning or coding.
