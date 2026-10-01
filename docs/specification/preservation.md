# Preservation record and source crosswalk

[Master specification](README.md)

**Scope:** Architecture/specification material from “Rust Django Equivalent Explained,” recovered through the existing master design, AMG v0.1, ORM v0.1, and Parts III–IV, plus the runtime, documentation, evidence and Django-adoption discussion. Captured 2026-09-30. No private chat transcript or account information is included.

## Editorial rules

- Topic files preserve the substantive existing design, examples, open questions, and test requirements. Numbered source sections are identified in HTML comments; headings are normalized for standalone reading.
- Later detailed specifications take precedence over early master examples. Explicit DB context supersedes ambient lookup; explicit registration/exposure and security boundaries govern all generated interfaces.
- SPEC-LOCKED means architecture agreement. No library compatibility, benchmark, implementation or test is newly claimed VALIDATED. Historical “PROVISIONALLY VALIDATED” terminology is normalized to SPEC-LOCKED / VALIDATION REQUIRED.
- Source statements such as “should,” “potential,” “open,” “future,” and proposed APIs retain their uncertainty. A page's settled principles do not lock all illustrative syntax.
- Earlier implementation phases, tickets and sequencing are SUPERSEDED by the later requirement to finish the specification before implementation planning. Only those obsolete planning sections are excluded, as identified below.
- Historical examples of commands, output, tests passing and feature availability are illustrative. The design status lists mean “specified to some degree,” not implemented, fully resolved, or released.
- This documentation capture adds navigation, provenance, decision records and scaffolding. It does not add framework code, dependency manifests or a new implementation plan.

## Source section crosswalk

Every numbered section of the five existing specifications is accounted for below. Repeated locations indicate intentional contextual preservation, especially the historical master. Narrative runtime and quality requirements are consolidated into the runtime and quality pages; Django and AI requirements have dedicated scaffolding.

### Initial master specification

| Source section | Durable location or disposition |
| --- | --- |
| 1. Vision | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 2. Core Philosophy | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 3. High-Level Architecture | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 4. Runtime Architecture | [01-runtime-architecture](01-runtime-architecture.md); [initial-architecture](history/initial-architecture.md) |
| 5. Blocking Work | [01-runtime-architecture](01-runtime-architecture.md); [initial-architecture](history/initial-architecture.md) |
| 6. Workspace Architecture | [initial-architecture](history/initial-architecture.md) |
| 7. Rjango Application Structure | [initial-architecture](history/initial-architecture.md) |
| 8. Application Boot | [01-runtime-architecture](01-runtime-architecture.md); [initial-architecture](history/initial-architecture.md) |
| 9. Models | [initial-architecture](history/initial-architecture.md) |
| 10. Query API | [initial-architecture](history/initial-architecture.md) |
| 11. Migrations | [initial-architecture](history/initial-architecture.md) |
| 12. Routing | [initial-architecture](history/initial-architecture.md) |
| 13. Automatic API Generation | [initial-architecture](history/initial-architecture.md) |
| 14. Validation and Schemas | [initial-architecture](history/initial-architecture.md) |
| 15. Error Model | [initial-architecture](history/initial-architecture.md) |
| 16. Authentication | [initial-architecture](history/initial-architecture.md) |
| 17. Permissions | [initial-architecture](history/initial-architecture.md) |
| 18. OpenAPI | [initial-architecture](history/initial-architecture.md) |
| 19. Frontend Client Generation | [initial-architecture](history/initial-architecture.md) |
| 20. Admin | [initial-architecture](history/initial-architecture.md) |
| 21. Background Jobs | [initial-architecture](history/initial-architecture.md) |
| 22. Real-Time | [initial-architecture](history/initial-architecture.md) |
| 23. The Rjango CLI | [initial-architecture](history/initial-architecture.md) |
| 24. `rjango check` | [initial-architecture](history/initial-architecture.md) |
| 25. AI Architecture | [initial-architecture](history/initial-architecture.md) |
| 26. Native MCP | [initial-architecture](history/initial-architecture.md) |
| 27. MCP Security | [initial-architecture](history/initial-architecture.md) |
| 28. Agent-Friendly Architecture | [initial-architecture](history/initial-architecture.md) |
| 29. Observability | [initial-architecture](history/initial-architecture.md) |
| 30. Configuration | [initial-architecture](history/initial-architecture.md) |
| 31. Testing | [initial-architecture](history/initial-architecture.md) |
| 32. Framework Security | [initial-architecture](history/initial-architecture.md) |
| 33. Performance Philosophy | [initial-architecture](history/initial-architecture.md) |
| 34. Escape Hatches | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 35. Development Phases | SUPERSEDED early sequencing/milestones/tickets; no implementation plan retained |
| 36. First Development Milestone | SUPERSEDED early sequencing/milestones/tickets; no implementation plan retained |
| 37. Initial Engineering Tickets | SUPERSEDED early sequencing/milestones/tickets; no implementation plan retained |
| 38. v0.1 Definition | SUPERSEDED early sequencing/milestones/tickets; no implementation plan retained |
| 39. v1.0 Definition | [initial-architecture](history/initial-architecture.md) |
| 40. What Rjango Is Not | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 41. Product Identity | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |
| 42. Architectural North Star | [00-vision-and-principles](00-vision-and-principles.md); [initial-architecture](history/initial-architecture.md) |

### AMG specification

| Source section | Durable location or disposition |
| --- | --- |
| 1. Purpose | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 2. Core Principle | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 3. Three Information Planes | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 4. Graph Structure | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 5. Stable Metadata IDs | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 6. Qualified Names | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 7. Initial Core Node Types | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 8. Application Nodes | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 9. Model Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 10. Fields | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 11. Type System | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 12. Relationship Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 13. Constraints | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 14. Index Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 15. Schema Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 16. Route Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 17. Handler Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 18. Source References | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 19. Permission Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 20. Job Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 21. Settings Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 22. Edge Types | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 23. Typed Graph + Specialized Registries | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 24. Registration Model | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 25. Macro Design | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 26. Build Pipeline | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 27. Validation | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 28. Warnings | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 29. Deterministic Fingerprints | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 30. Why Fingerprints Matter | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 31. Metadata Versioning | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 32. Serialized Format | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 33. Canonical Serialization | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 34. Query API | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 35. Generic Graph Querying | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 36. CLI Introspection | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 37. Graph Exploration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 38. Migration Integration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 39. Migration Operations | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 40. Migration Safety | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 41. OpenAPI Integration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 42. TypeScript Generation | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 43. Admin Integration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 44. `rjango check` Integration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 45. MCP Integration | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 46. MCP Responses Should Reference Metadata IDs | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 47. MCP Context Efficiency | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 48. AI Change Workflow | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 49. Metadata Exposure Policies | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 50. Sensitive Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 51. MCP Authorization | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 52. Extension Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 53. Extension Namespace Rules | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 54. Provenance | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 55. Immutability | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 56. Performance | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 57. Internal Storage | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 58. Development Hot Reload | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 59. Graph Diff | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 60. Dependency Impact Analysis | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 61. Public HTTP Exposure | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 62. Rjango Metadata API | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 63. Initial Crate Location | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 64. Trait-Based Metadata | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 65. MetadataFragment | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 66. Important Architectural Rule | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 67. Phase 0 Implementation Order | SUPERSEDED early implementation order |
| 68. First Prototype | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 69. First MCP Prototype | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 70. Long-Term Potential | [02-application-metadata-graph](02-application-metadata-graph.md) |
| 71. Architectural North Star | [02-application-metadata-graph](02-application-metadata-graph.md) |

### ORM specification

| Source section | Durable location or disposition |
| --- | --- |
| 1. Objective | [03-models-and-orm](03-models-and-orm.md) |
| 2. Architectural Stack | [03-models-and-orm](03-models-and-orm.md) |
| 3. Critical Rule: Rjango Owns Its Public API | [03-models-and-orm](03-models-and-orm.md) |
| 4. Explicit Database Context | [03-models-and-orm](03-models-and-orm.md) |
| 5. Model Declaration | [03-models-and-orm](03-models-and-orm.md) |
| 6. Generated Model Components | [03-models-and-orm](03-models-and-orm.md) |
| 7. Fields API | [03-models-and-orm](03-models-and-orm.md) |
| 8. Typed Queries | [03-models-and-orm](03-models-and-orm.md) |
| 9. QuerySet Mental Model | [03-models-and-orm](03-models-and-orm.md) |
| 10. QuerySet Operations | [03-models-and-orm](03-models-and-orm.md) |
| 11. Primary-Key Convenience | [03-models-and-orm](03-models-and-orm.md) |
| 12. `first()` Semantics | [03-models-and-orm](03-models-and-orm.md) |
| 13. Complex Filtering | [03-models-and-orm](03-models-and-orm.md) |
| 14. Django Q-Object Equivalent | [03-models-and-orm](03-models-and-orm.md) |
| 15. Creation | [03-models-and-orm](03-models-and-orm.md) |
| 16. Builder Creation | [03-models-and-orm](03-models-and-orm.md) |
| 17. Updates | [03-models-and-orm](03-models-and-orm.md) |
| 18. Explicit Changesets | [03-models-and-orm](03-models-and-orm.md) |
| 19. Bulk Update | [03-models-and-orm](03-models-and-orm.md) |
| 20. Delete | [03-models-and-orm](03-models-and-orm.md) |
| 21. Relationships | [03-models-and-orm](03-models-and-orm.md) |
| 22. Relationship Types | [03-models-and-orm](03-models-and-orm.md) |
| 23. Loading Relationships | [03-models-and-orm](03-models-and-orm.md) |
| 24. Nested Loading | [03-models-and-orm](03-models-and-orm.md) |
| 25. N+1 Policy | [03-models-and-orm](03-models-and-orm.md) |
| 26. Development N+1 Detection | [03-models-and-orm](03-models-and-orm.md) |
| 27. Transactions | [03-models-and-orm](03-models-and-orm.md) |
| 28. Nested Transactions | [03-models-and-orm](03-models-and-orm.md) |
| 29. Transaction Cancellation | [03-models-and-orm](03-models-and-orm.md) |
| 30. Raw SQL Escape Hatch | [03-models-and-orm](03-models-and-orm.md) |
| 31. Compile-Checked SQL | [03-models-and-orm](03-models-and-orm.md) |
| 32. PostgreSQL-First Strategy | [03-models-and-orm](03-models-and-orm.md) |
| 33. Database Portability | [03-models-and-orm](03-models-and-orm.md) |
| 34. Connection Pool | [03-models-and-orm](03-models-and-orm.md) |
| 35. Pool Instrumentation | [03-models-and-orm](03-models-and-orm.md) |
| 36. Query Instrumentation | [03-models-and-orm](03-models-and-orm.md) |
| 37. Slow Query Detection | [03-models-and-orm](03-models-and-orm.md) |
| 38. Database Errors | [03-models-and-orm](03-models-and-orm.md) |
| 39. Constraint Mapping | [03-models-and-orm](03-models-and-orm.md) |
| 40. Migrations | [04-migrations](04-migrations.md) |
| 41. Migration Command | [04-migrations](04-migrations.md) |
| 42. Migration Plan | [04-migrations](04-migrations.md) |
| 43. Destructive Migration Safety | [04-migrations](04-migrations.md) |
| 44. Migration Files | [04-migrations](04-migrations.md) |
| 45. Migration Determinism | [04-migrations](04-migrations.md) |
| 46. Rename Detection | [04-migrations](04-migrations.md) |
| 47. Schema Drift | [04-migrations](04-migrations.md) |
| 48. Development Schema Sync | [04-migrations](04-migrations.md) |
| 49. Metadata Integration | [03-models-and-orm](03-models-and-orm.md) |
| 50. MCP | [03-models-and-orm](03-models-and-orm.md) |
| 51. MCP Query Inspection | [03-models-and-orm](03-models-and-orm.md) |
| 52. MCP Database Access Is Separate | [03-models-and-orm](03-models-and-orm.md) |
| 53. Django Mapping | [models](../django/models.md) |
| 54. Django `select_related` | [queries](../django/queries.md) |
| 55. Django `prefetch_related` | [queries](../django/queries.md) |
| 56. Django `save()` | [queries](../django/queries.md) |
| 57. Django `atomic` | [queries](../django/queries.md) |
| 58. Pagination | [03-models-and-orm](03-models-and-orm.md) |
| 59. Streaming | [03-models-and-orm](03-models-and-orm.md) |
| 60. Aggregation | [03-models-and-orm](03-models-and-orm.md) |
| 61. Partial Selections | [03-models-and-orm](03-models-and-orm.md) |
| 62. Custom Query DTOs | [03-models-and-orm](03-models-and-orm.md) |
| 63. Model Hooks | [03-models-and-orm](03-models-and-orm.md) |
| 64. Domain Events | [03-models-and-orm](03-models-and-orm.md) |
| 65. Soft Delete | [03-models-and-orm](03-models-and-orm.md) |
| 66. Multi-Tenancy | [03-models-and-orm](03-models-and-orm.md) |
| 67. Optimistic Locking | [03-models-and-orm](03-models-and-orm.md) |
| 68. Database Generated IDs | [03-models-and-orm](03-models-and-orm.md) |
| 69. Enums | [03-models-and-orm](03-models-and-orm.md) |
| 70. Custom Types | [03-models-and-orm](03-models-and-orm.md) |
| 71. Compile-Time Diagnostics | [03-models-and-orm](03-models-and-orm.md) |
| 72. Runtime Validation | [03-models-and-orm](03-models-and-orm.md) |
| 73. Required Public API Tests | [03-models-and-orm](03-models-and-orm.md) |
| 74. Coverage Standard | [03-models-and-orm](03-models-and-orm.md) |
| 75. Coverage Is Not Enough | [03-models-and-orm](03-models-and-orm.md) |
| 76. ORM Prototype Gate | [03-models-and-orm](03-models-and-orm.md) |
| 77. Query Prototype Gate | [03-models-and-orm](03-models-and-orm.md) |
| 78. SQL Inspection Tests | [03-models-and-orm](03-models-and-orm.md) |
| 79. Performance Benchmark Matrix | [03-models-and-orm](03-models-and-orm.md) |
| 80. Performance Acceptance Goal | [03-models-and-orm](03-models-and-orm.md) |
| 81. N+1 Verification | [03-models-and-orm](03-models-and-orm.md) |
| 82. Concurrency Test | [03-models-and-orm](03-models-and-orm.md) |
| 83. Cancellation Test | [03-models-and-orm](03-models-and-orm.md) |
| 84. Migration Round-Trip Tests | [03-models-and-orm](03-models-and-orm.md) |
| 85. Property Tests | [03-models-and-orm](03-models-and-orm.md) |
| 86. Fuzzing | [03-models-and-orm](03-models-and-orm.md) |
| 87. Mutation Testing | [03-models-and-orm](03-models-and-orm.md) |
| 88. Documentation Tests | [03-models-and-orm](03-models-and-orm.md) |
| 89. Human Documentation Requirement | [03-models-and-orm](03-models-and-orm.md) |
| 90. AI Documentation Requirement | [03-models-and-orm](03-models-and-orm.md) |
| 91. Documentation Version Awareness | [03-models-and-orm](03-models-and-orm.md) |
| 92. Error Documentation | [03-models-and-orm](03-models-and-orm.md) |
| 93. Stability Rule | [03-models-and-orm](03-models-and-orm.md) |
| 94. Initial Crates | [03-models-and-orm](03-models-and-orm.md) |
| 95. Implementation Order | SUPERSEDED early implementation order |
| 96. Validation Stages | [03-models-and-orm](03-models-and-orm.md) |
| 97. Decisions We Are Comfortable Locking Now | [03-models-and-orm](03-models-and-orm.md) |
| 98. Decisions That Must Remain Open Until Prototype Testing | [03-models-and-orm](03-models-and-orm.md) |
| 99. First Acceptance Prototype | [03-models-and-orm](03-models-and-orm.md) |
| 100. North Star | [03-models-and-orm](03-models-and-orm.md) |

### Part III

| Source section | Durable location or disposition |
| --- | --- |
| 1. Migrations & Schema Management | [04-migrations](04-migrations.md) |
| 2. Migration Safety Classification | [04-migrations](04-migrations.md) |
| 3. Migration Plan | [04-migrations](04-migrations.md) |
| 4. Rename Detection | [04-migrations](04-migrations.md) |
| 5. Stable Field Identity | [04-migrations](04-migrations.md) |
| 6. Atomicity | [04-migrations](04-migrations.md) |
| 7. Data Migrations | [04-migrations](04-migrations.md) |
| 8. Historical Models | [04-migrations](04-migrations.md) |
| 9. Raw SQL | [04-migrations](04-migrations.md) |
| 10. Reversibility | [04-migrations](04-migrations.md) |
| 11. Migration Squashing | [04-migrations](04-migrations.md) |
| 12. Migration Branching | [04-migrations](04-migrations.md) |
| 13. Drift Detection | [04-migrations](04-migrations.md) |
| 14. Development Schema Sync | [04-migrations](04-migrations.md) |
| 15. Database Lock Awareness | [04-migrations](04-migrations.md) |
| 16. Migration Metadata + MCP | [04-migrations](04-migrations.md) |
| 17. Migration Testing Standard | [04-migrations](04-migrations.md) |
| 18. Django Mapping — Migrations | [04-migrations](04-migrations.md); [migrations](../django/migrations.md) |
| 19. Migration Decisions | [04-migrations](04-migrations.md) |
| 20. Rjango Applications / Modules | [05-app-system](05-app-system.md) |
| 21. App Definition | [05-app-system](05-app-system.md) |
| 22. App Identity | [05-app-system](05-app-system.md) |
| 23. App Dependencies | [05-app-system](05-app-system.md) |
| 24. App Lifecycle | [05-app-system](05-app-system.md) |
| 25. Reusable Apps | [05-app-system](05-app-system.md) |
| 26. Reusable App Isolation | [05-app-system](05-app-system.md) |
| 27. App Configuration | [05-app-system](05-app-system.md) |
| 28. Application Metadata | [05-app-system](05-app-system.md) |
| 29. Django Mapping — Apps | [05-app-system](05-app-system.md); [apps](../django/apps.md) |
| 30. HTTP Architecture | [06-http-and-routing](06-http-and-routing.md) |
| 31. Handler Design | [06-http-and-routing](06-http-and-routing.md) |
| 32. Explicit Request Context | [06-http-and-routing](06-http-and-routing.md) |
| 33. Router Metadata | [06-http-and-routing](06-http-and-routing.md) |
| 34. Nested Routers | [06-http-and-routing](06-http-and-routing.md) |
| 35. Route Names | [06-http-and-routing](06-http-and-routing.md) |
| 36. Middleware | [06-http-and-routing](06-http-and-routing.md) |
| 37. Middleware Ordering | [06-http-and-routing](06-http-and-routing.md) |
| 38. Timeouts | [06-http-and-routing](06-http-and-routing.md) |
| 39. Cancellation | [06-http-and-routing](06-http-and-routing.md) |
| 40. Request Bodies | [06-http-and-routing](06-http-and-routing.md) |
| 41. File Uploads | [06-http-and-routing](06-http-and-routing.md) |
| 42. Responses | [06-http-and-routing](06-http-and-routing.md) |
| 43. Streaming | [06-http-and-routing](06-http-and-routing.md) |
| 44. Content Negotiation | [06-http-and-routing](06-http-and-routing.md) |
| 45. HTTP Errors | [06-http-and-routing](06-http-and-routing.md) |
| 46. Request IDs | [06-http-and-routing](06-http-and-routing.md) |
| 47. Reverse Proxy Awareness | [06-http-and-routing](06-http-and-routing.md) |
| 48. Health Endpoints | [06-http-and-routing](06-http-and-routing.md) |
| 49. Schemas & Serialization | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 50. Schema Type Information | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 51. Input vs Output | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 52. Sensitive Fields | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 53. Validation | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 54. Validation Ordering | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 55. Nested Schemas | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 56. Partial Updates | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 57. Schema Versioning | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 58. Serialization Naming | [07-schemas-and-validation](07-schemas-and-validation.md) |
| 59. API Framework | [08-api-framework](08-api-framework.md) |
| 60. Resource APIs | [08-api-framework](08-api-framework.md) |
| 61. Explicit Exposure | [08-api-framework](08-api-framework.md) |
| 62. Filtering | [08-api-framework](08-api-framework.md) |
| 63. Ordering | [08-api-framework](08-api-framework.md) |
| 64. Pagination | [08-api-framework](08-api-framework.md) |
| 65. API Error Contract | [08-api-framework](08-api-framework.md) |
| 66. OpenAPI | [08-api-framework](08-api-framework.md) |
| 67. Generated SDKs | [08-api-framework](08-api-framework.md) |
| 68. Idempotency | [08-api-framework](08-api-framework.md) |
| 69. API Versioning | [08-api-framework](08-api-framework.md) |
| 70. Deprecation | [08-api-framework](08-api-framework.md) |
| 71. Authentication Architecture | [09-authentication](09-authentication.md) |
| 72. Identity | [09-authentication](09-authentication.md) |
| 73. User Model | [09-authentication](09-authentication.md) |
| 74. Password Handling | [09-authentication](09-authentication.md) |
| 75. Sessions | [09-authentication](09-authentication.md) |
| 76. JWT | [09-authentication](09-authentication.md) |
| 77. API Keys | [09-authentication](09-authentication.md) |
| 78. OAuth/OIDC | [09-authentication](09-authentication.md) |
| 79. Passkeys | [09-authentication](09-authentication.md) |
| 80. Multi-Factor Authentication | [09-authentication](09-authentication.md) |
| 81. Authorization | [10-authorization-and-security](10-authorization-and-security.md) |
| 82. Roles | [10-authorization-and-security](10-authorization-and-security.md) |
| 83. Object-Level Authorization | [10-authorization-and-security](10-authorization-and-security.md) |
| 84. Permission Composition | [10-authorization-and-security](10-authorization-and-security.md) |
| 85. Authorization + AMG | [10-authorization-and-security](10-authorization-and-security.md) |
| 86. Service Accounts | [10-authorization-and-security](10-authorization-and-security.md) |
| 87. Agent Identity | [10-authorization-and-security](10-authorization-and-security.md) |
| 88. Impersonation | [10-authorization-and-security](10-authorization-and-security.md) |
| 89. Security Architecture | [10-authorization-and-security](10-authorization-and-security.md) |
| 90. CSRF | [10-authorization-and-security](10-authorization-and-security.md) |
| 91. CORS | [10-authorization-and-security](10-authorization-and-security.md) |
| 92. Security Headers | [10-authorization-and-security](10-authorization-and-security.md) |
| 93. Trusted Hosts | [10-authorization-and-security](10-authorization-and-security.md) |
| 94. Trusted Proxies | [10-authorization-and-security](10-authorization-and-security.md) |
| 95. Rate Limiting | [10-authorization-and-security](10-authorization-and-security.md) |
| 96. Request Limits | [10-authorization-and-security](10-authorization-and-security.md) |
| 97. Secrets | [10-authorization-and-security](10-authorization-and-security.md) |
| 98. Secret Sources | [10-authorization-and-security](10-authorization-and-security.md) |
| 99. SQL Injection | [10-authorization-and-security](10-authorization-and-security.md) |
| 100. XSS | [10-authorization-and-security](10-authorization-and-security.md) |
| 101. Open Redirects | [10-authorization-and-security](10-authorization-and-security.md) |
| 102. File Security | [10-authorization-and-security](10-authorization-and-security.md) |
| 103. Error Leakage | [10-authorization-and-security](10-authorization-and-security.md) |
| 104. Audit Log | [10-authorization-and-security](10-authorization-and-security.md) |
| 105. MCP Security Boundary | [10-authorization-and-security](10-authorization-and-security.md) |
| 106. Production MCP | [10-authorization-and-security](10-authorization-and-security.md) |
| 107. MCP Confirmation Policy | [10-authorization-and-security](10-authorization-and-security.md) |
| 108. Supply-Chain Security | [10-authorization-and-security](10-authorization-and-security.md) |
| 109. Unsafe Rust Policy | [10-authorization-and-security](10-authorization-and-security.md) |
| 110. Security Diagnostics | [10-authorization-and-security](10-authorization-and-security.md) |
| 111. Django Transition Documentation | [web-and-security](../django/web-and-security.md) |
| 112. Human Documentation Requirements for These Systems | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 113. AI Documentation Requirements | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 114. Cross-System Invariants Established So Far | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 115. Current Design Status | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 116. Quality Standard Applied to Every Section | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 117. Architectural Result So Far | [20-cross-system-invariants](20-cross-system-invariants.md) |

### Part IV

| Source section | Durable location or disposition |
| --- | --- |
| 1. Cross-System Principle | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 2. Rjango Admin | [11-admin](11-admin.md) |
| 3. Admin Is a Projection | [11-admin](11-admin.md) |
| 4. Explicit Admin Registration | [11-admin](11-admin.md) |
| 5. Default Admin Experience | [11-admin](11-admin.md) |
| 6. Admin Views | [11-admin](11-admin.md) |
| 7. List Configuration | [11-admin](11-admin.md) |
| 8. Search | [11-admin](11-admin.md) |
| 9. Filters | [11-admin](11-admin.md) |
| 10. Admin Forms | [11-admin](11-admin.md) |
| 11. Read-Only Fields | [11-admin](11-admin.md) |
| 12. Relationship Editing | [11-admin](11-admin.md) |
| 13. Bulk Actions | [11-admin](11-admin.md) |
| 14. Admin Actions | [11-admin](11-admin.md) |
| 15. Admin Permissions | [11-admin](11-admin.md) |
| 16. Object-Level Admin Permissions | [11-admin](11-admin.md) |
| 17. Admin Audit History | [11-admin](11-admin.md) |
| 18. Admin Authentication | [11-admin](11-admin.md) |
| 19. Admin UI Architecture | [11-admin](11-admin.md) |
| 20. Admin Customization | [11-admin](11-admin.md) |
| 21. Admin + MCP | [11-admin](11-admin.md) |
| 22. Coming from Django — Admin | [11-admin](11-admin.md); [admin](../django/admin.md) |
| 23. Background Jobs | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 24. Job Definition | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 25. Job Metadata | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 26. Durable Queue Semantics | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 27. Idempotency | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 28. Dispatch | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 29. Transaction-Aware Dispatch | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 30. Job Backends | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 31. Job Workers | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 32. Worker Concurrency | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 33. Job Timeouts | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 34. Retries | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 35. Backoff | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 36. Dead-Letter State | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 37. Scheduled Jobs | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 38. Cron-Style Scheduling | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 39. Overlap Policy | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 40. Job Cancellation | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 41. Job Progress | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 42. Job Observability | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 43. Job Testing | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 44. Django Mapping — Jobs | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 45. Caching | [13-caching](13-caching.md) |
| 46. Cache Backends | [13-caching](13-caching.md) |
| 47. Cache Namespaces | [13-caching](13-caching.md) |
| 48. Typed Cache API | [13-caching](13-caching.md) |
| 49. Cache Serialization Versioning | [13-caching](13-caching.md) |
| 50. TTL | [13-caching](13-caching.md) |
| 51. Cache-Aside Helpers | [13-caching](13-caching.md) |
| 52. Stampede Protection | [13-caching](13-caching.md) |
| 53. Negative Caching | [13-caching](13-caching.md) |
| 54. Cache Invalidation | [13-caching](13-caching.md) |
| 55. Query Caching | [13-caching](13-caching.md) |
| 56. Per-Request Cache | [13-caching](13-caching.md) |
| 57. Sensitive Data in Cache | [13-caching](13-caching.md) |
| 58. Cache Observability | [13-caching](13-caching.md) |
| 59. Storage & Files | [14-storage](14-storage.md) |
| 60. Storage Backend | [14-storage](14-storage.md) |
| 61. Storage Handles | [14-storage](14-storage.md) |
| 62. Streaming Upload | [14-storage](14-storage.md) |
| 63. Direct-to-Object-Storage Uploads | [14-storage](14-storage.md) |
| 64. Upload State | [14-storage](14-storage.md) |
| 65. Checksums | [14-storage](14-storage.md) |
| 66. Content Type | [14-storage](14-storage.md) |
| 67. Filename Safety | [14-storage](14-storage.md) |
| 68. Signed URLs | [14-storage](14-storage.md) |
| 69. Public vs Private Objects | [14-storage](14-storage.md) |
| 70. Deletion Semantics | [14-storage](14-storage.md) |
| 71. Model/File Integration | [14-storage](14-storage.md) |
| 72. Storage Security Hooks | [14-storage](14-storage.md) |
| 73. Email & Notifications | [15-email-and-notifications](15-email-and-notifications.md) |
| 74. Email Interface | [15-email-and-notifications](15-email-and-notifications.md) |
| 75. Email Backends | [15-email-and-notifications](15-email-and-notifications.md) |
| 76. Email Address Types | [15-email-and-notifications](15-email-and-notifications.md) |
| 77. Development Email Backend | [15-email-and-notifications](15-email-and-notifications.md) |
| 78. Email Templates | [15-email-and-notifications](15-email-and-notifications.md) |
| 79. Email and Jobs | [15-email-and-notifications](15-email-and-notifications.md) |
| 80. Delivery Tracking | [15-email-and-notifications](15-email-and-notifications.md) |
| 81. Notification Abstraction | [15-email-and-notifications](15-email-and-notifications.md) |
| 82. Webhooks | [15-email-and-notifications](15-email-and-notifications.md) |
| 83. Realtime | [16-realtime](16-realtime.md) |
| 84. WebSockets | [16-realtime](16-realtime.md) |
| 85. SSE | [16-realtime](16-realtime.md) |
| 86. Channels | [16-realtime](16-realtime.md) |
| 87. Groups | [16-realtime](16-realtime.md) |
| 88. Distributed Pub/Sub | [16-realtime](16-realtime.md) |
| 89. Presence | [16-realtime](16-realtime.md) |
| 90. Realtime Authentication | [16-realtime](16-realtime.md) |
| 91. Realtime Message Schemas | [16-realtime](16-realtime.md) |
| 92. Backpressure | [16-realtime](16-realtime.md) |
| 93. Reconnect | [16-realtime](16-realtime.md) |
| 94. Realtime Events vs Durable Events | [16-realtime](16-realtime.md) |
| 95. Realtime + Generated Clients | [16-realtime](16-realtime.md) |
| 96. Events | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 97. Event Categories | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 98. Domain Events | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 99. Event Dispatch Semantics | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 100. Transactional Domain Events | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 101. Event Listeners | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 102. Hidden Side Effects | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 103. Model Lifecycle Hooks | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 104. Event Cycles | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 105. Event Failure Policy | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 106. Application Lifecycle | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 107. Startup Hooks | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 108. Graceful Shutdown | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 109. Configuration | [18-configuration](18-configuration.md) |
| 110. Primary Configuration | [18-configuration](18-configuration.md) |
| 111. Layering | [18-configuration](18-configuration.md) |
| 112. Environment Profiles | [18-configuration](18-configuration.md) |
| 113. Environment Variables | [18-configuration](18-configuration.md) |
| 114. Typed Application Settings | [18-configuration](18-configuration.md) |
| 115. Required Settings | [18-configuration](18-configuration.md) |
| 116. Secret Type | [18-configuration](18-configuration.md) |
| 117. Configuration Introspection | [18-configuration](18-configuration.md) |
| 118. Configuration Schema | [18-configuration](18-configuration.md) |
| 119. Invalid Configuration | [18-configuration](18-configuration.md) |
| 120. Dynamic Configuration | [18-configuration](18-configuration.md) |
| 121. Feature Flags | [18-configuration](18-configuration.md) |
| 122. Configuration + MCP | [18-configuration](18-configuration.md) |
| 123. Configuration + Admin | [18-configuration](18-configuration.md) |
| 124. Documentation Architecture Across These Systems | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 125. AI Documentation | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 126. Django Transition — Jobs | [services](../django/services.md) |
| 127. Django Transition — Caching | [services](../django/services.md) |
| 128. Django Transition — Storage | [services](../django/services.md) |
| 129. Django Transition — Email | [services](../django/services.md) |
| 130. Django Transition — Signals | [services](../django/services.md) |
| 131. Testing — Admin | [11-admin](11-admin.md) |
| 132. Testing — Jobs | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 133. Testing — Cache | [13-caching](13-caching.md) |
| 134. Testing — Storage | [14-storage](14-storage.md) |
| 135. Testing — Email | [15-email-and-notifications](15-email-and-notifications.md) |
| 136. Testing — Realtime | [16-realtime](16-realtime.md) |
| 137. Testing — Events | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 138. Testing — Configuration | [18-configuration](18-configuration.md) |
| 139. Fault-Injection Standard | [19-quality-and-documentation](19-quality-and-documentation.md) |
| 140. Current Lock Status — Admin | [11-admin](11-admin.md) |
| 141. Current Lock Status — Jobs | [12-jobs-and-scheduling](12-jobs-and-scheduling.md) |
| 142. Current Lock Status — Cache | [13-caching](13-caching.md) |
| 143. Current Lock Status — Storage | [14-storage](14-storage.md) |
| 144. Current Lock Status — Email/Notifications | [15-email-and-notifications](15-email-and-notifications.md) |
| 145. Current Lock Status — Realtime | [16-realtime](16-realtime.md) |
| 146. Current Lock Status — Events | [17-events-and-lifecycle](17-events-and-lifecycle.md) |
| 147. Current Lock Status — Configuration | [18-configuration](18-configuration.md) |
| 148. New Cross-System Invariant: Durable Side Effects | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 149. New Cross-System Invariant: Side-Effect Visibility | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 150. New Cross-System Invariant: Explicit Durability | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 151. New Cross-System Invariant: Auth Everywhere | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 152. New Cross-System Invariant: Observable by Default | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 153. New Cross-System Invariant: No Hidden Network I/O | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 154. Rjango's Emerging Programming Model | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 155. Architectural North Star | [20-cross-system-invariants](20-cross-system-invariants.md) |
| 156. Status After Part IV | [21-design-backlog](21-design-backlog.md) |

## Review Candidate 1 reconciliation — 2026-09-30

The [durable architecture review](../reviews/architecture-review-candidate-1.md) maps all 22 amendments to authoritative topic sections and records source truncation. Earlier conversation sections remain preserved; conflicting migration atomicity, relation-state, fingerprint and MCP-secret sketches are superseded by Candidate 1 amendments. No original section is deleted and no experimental validation is implied. Part V remains incompletely captured.
