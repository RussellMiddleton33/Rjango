# Remaining design inventory

[Master specification](README.md) · [Preservation record](preservation.md)

**Status:** PROPOSED / partially specified. **Evidence:** VALIDATION REQUIRED.

The existing design covers the major application systems, but the complete Rjango 1.0 specification is not finished. The following are design questions and coverage gaps, not an implementation plan or execution order. The final cross-system review must precede any implementation planning.

| Area | Preserved starting point | Remaining design boundary |
| --- | --- | --- |
| CLI and developer experience | Historical master §§23–24; CLI/inspection examples across AMG | Complete command taxonomy, output/exit contracts, scaffolding, extensions and diagnostics. |
| Development server and hot reload | AMG snapshot rebuild; initial project experience | Rebuild lifecycle, watching, process coordination and reload failure behavior. |
| Testing framework | [Quality requirements](19-quality-and-documentation.md), subsystem tests | Fixtures, app/client/database isolation, fake backends, time control and harness API. |
| Observability | Query, pool, HTTP, job and service instrumentation requirements | Unified trace/log/metric conventions, exporters, redaction, overhead and operational interfaces. |
| Errors and diagnostics | Stable codes, structured envelopes, source context | Shared taxonomy, chaining, localization, machine schema and redaction contracts. |
| Human documentation architecture | Tutorial/how-to/explanation/reference requirements | Information architecture, versioning, generation, search, build and example validation. |
| Django transition system | [Django scaffolding](../django/README.md) | Full concept coverage, curriculum, verified translations and migration guidance. |
| Complete AI/MCP architecture | [MCP boundaries](../ai/mcp-architecture.md) and AMG resources | Transport, authentication, tool/resource schemas, permissions, review and compatibility details. |
| Agent documentation system | [Agent guide](../ai/agent-guide.md) | Version-aware retrieval, machine contracts, generated references and drift checks. |
| Plugin and ecosystem system | Explicit apps; namespaced AMG extensions | Distribution, discovery, dependency/version rules, capability trust and stable extension APIs. |
| Templates, forms and static assets | API-first, not API-only principle | Rendering, validation integration, escaping, asset handling and customization contracts. |
| Internationalization, localization and timezones | Schedule timezone/DST requirement | Translation catalogs, locale negotiation, formatting, timezone-aware data and error messages. |
| Advanced database capabilities | PostgreSQL-first and raw escape hatch; ORM open decisions | Multi-database routing, replicas, locks, extensions, full-text search, tenancy, soft deletion and optimistic concurrency. |
| Deployment and operations | Explicit migrations, lifecycle, readiness and worker separation | Process/container contract, config/secrets operations, scaling, recovery and operational guides. |
| Performance contract | Existing ORM benchmark matrix; historical philosophy | Measurable budgets, representative workloads, environments and regression acceptance thresholds. |
| Compatibility and versioning | Stable Rjango API; schema-versioned AMG | MSRV, dependency policy, semver, deprecation/support windows and upgrade guarantees. |
| Open-source governance | Open-source intent | License/governance policy, RFCs, contribution review, security reporting and release process. |
| Reference applications | Early example-app concepts | Representative end-to-end designs and the behaviors they must demonstrate. |
| Final Rjango 1.0 contract | Initial scope aspirations | Explicit inclusion/exclusion, coherence review, unresolved tradeoffs and evidence criteria. |

## Topics designed but not fully resolved

Every topic page includes open questions. Particularly significant ones include model macro and query-field syntax; relationship loaded-state representation; migration file and field identity schemes; backend choices for queues, caching and distributed realtime; admin UI technology; notification breadth; dynamic configuration; and complete authorization semantics across async effects.

Existing prototypes, benchmark matrices and test scenarios in the ORM specification are retained as evidence requirements. They have not been run by this capture and are not a roadmap. Earlier engineering tickets, phases and sequencing were superseded by the user's later design-first instruction.

## Original design-status record

<!-- Source: iv section 156. -->
## Status After Part IV

At this point the specification covers the primary runtime/application systems:

```
✓ Core philosophy
✓ Async/runtime architecture
✓ Application Metadata Graph
✓ Models/ORM
✓ Migrations/schema evolution
✓ App/module system
✓ HTTP/routing
✓ Schemas/validation
✓ API framework
✓ Authentication
✓ Authorization
✓ Security
✓ Admin
✓ Jobs/scheduling
✓ Caching
✓ Storage/files
✓ Email
✓ Realtime
✓ Events/lifecycle
✓ Configuration/environments
```

The remaining major 1.0 design areas are primarily the **developer-platform and operational layers**:

```
CLI & developer experience
Development server / hot reload
Testing framework
Observability
Error/diagnostic system
Documentation architecture
Django-transition system
AI/MCP architecture in full detail
Plugin/ecosystem system
Templates/forms/static files
Internationalization/timezones
Advanced database capabilities
Deployment/operations
Performance contract
Compatibility/versioning
Open-source governance
Reference applications
Final Rjango 1.0 contract
```

No implementation roadmap should be created until those areas and the final cross-system architecture review are complete.
