# Authorization and security

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Authorization applies across every execution surface. Permission to inspect structure does not grant access to records, secrets, code execution, or destructive operations.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Query scoping and tenant isolation

Apply authorization scopes before list/count/aggregate/pagination/export execution; fetch-all-and-filter leaks counts/existence and breaks pagination. Compose scopes with filters and mutation object checks. Validate writable IDs/foreign keys inside the same tenant/transaction. Bulk writes, joins, relation loading, search, Admin, Jobs and MCP share scoping. Unsupported policy translation fails closed or requires an explicitly reviewed bounded alternative.

Tenant derives from trusted authenticated context, never client payload alone. Boundaries cover constraints/foreign keys, database routing, cache keys, storage, outbox and realtime channels. Row-level security adds defense in depth. Reset pooled tenant/session state safely. Raw SQL needs explicit scoped review; cross-tenant privilege is distinct and audited.

### Actor, subject and delegation chain

Actor is the executing human/agent/service; subject is the identity on whose behalf it acts. Ordered delegation links record issuer/recipient, scope, tenant, environment/resource, expiry and revocation reference. Effective authority is bounded by every link and current policy; delegation never amplifies privilege. Workers record execution service plus origin and revalidate where required. Audit includes operation/transport, correlation/message/idempotency IDs, policy decision/result and relevant fingerprints, never credentials. Revoked/expired chains fail closed.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (C1, H7, H8, M1). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Deny by default

Every exposed route, operation, resource action, admin resource/action, realtime channel/command, CLI command and MCP tool declares a policy or the explicit marker `public`. A missing declaration is an AMG validation error that stops startup and fails `rjango check`. `public` is itself AMG metadata, so "which endpoints can anonymous users reach?" is answered exactly. Django's open-by-default view convention is deliberately not copied.

### One policy, two forms

A policy is one named definition with two forms that must agree:

- `scope(ctx) -> Predicate<M>`: a query predicate applied before list/count/aggregate/pagination/export/search/relation loading/bulk writes.
- `check(ctx, &M) -> Decision`: an object decision for single-object reads and mutations.

Policies receive the [operation context](22-application-operations-and-services.md) and optionally the resource, never the HTTP request. A policy that cannot produce a `scope` can only protect single-object operations; using it on a collection is an AMG validation error unless the operation declares a reviewed bounded alternative. The test harness provides scope/check agreement property tests for registered policies. Their strength depends on the data generators supplied, so agreement is VALIDATION REQUIRED, not guaranteed.

Policies are annotated functions rather than implementations of async traits ([24](24-developer-experience-and-diagnostics.md)). There are three shapes (illustrative):

- **Context-only:** `#[rjango::policy] async fn can_create_venue(ctx: &Ctx) -> Decision`, for creates and other operations without an existing object. It may also take the validated input: `(ctx: &Ctx, input: &CreateVenue)`.
- **Object policy with both forms:** `#[rjango::policy(model = Venue, scope = venue_scope)] async fn can_edit_venue(ctx: &Ctx, venue: &Venue) -> Decision`, paired with `fn venue_scope(ctx: &Ctx) -> Predicate<Venue>`. The macro registers both as one policy and checks they target the same model.
- **Scope-only:** `#[rjango::policy(model = Venue, scope_only)] fn visible_venues(ctx: &Ctx) -> Predicate<Venue>`. The object check is derived by evaluating the predicate against the row's key.

### Route permission attributes are exposure, not business authority

`#[permission(...)]` on a route (sections below) is SUPERSEDED as the business authorization mechanism. Adapter attributes declare exposure and transport requirements (authentication mode, CSRF, origin, rate limits). Business authorization lives on the operation's policy and applies identically across all six adapters.

### Scoped data access is the default

The database handle given to handlers and operations (`ctx.db()` or the `Db` extractor) is a **scoped handle** bound to actor and tenant. Models that declare a tenant key (illustrative: `#[rjango::model(tenant = organization_id)]`) or a default policy have their scope applied automatically by every QuerySet terminal, relation load, `select_as`, stream, bulk update and bulk delete through that handle. Writes validate the tenant key and every writable foreign key inside the same tenant. A tenant-owned query through a context without a tenant fails closed.

Unscoped access requires the named capability `ctx.system_db(reason)`. The operation must declare it (Tier 2). Every use is audited with its reason, and `rjango check` lists every operation that holds it. Migrations, framework maintenance jobs and explicitly cross-tenant operations use it. Raw SQL through a scoped handle requires either a tenant-binding helper or the unscoped capability. The [ORM](03-models-and-orm.md) amendment defines the handle types. Explicit context (ADR 0005) is retained; explicit does not mean unscoped ([ADR 0012](../adr/0012-scoped-data-access-and-default-deny.md)).

Row-level security remains optional defense in depth. When it is enabled, the scoped handle runs every query inside an explicit transaction it opens itself, and sets the tenant with `set_config('rjango.tenant', $1, true)` (the transaction-local equivalent of `SET LOCAL`) as the first statement. `SET LOCAL` outside an explicit transaction block has no effect in PostgreSQL. Pooled session state is never relied on. The cost of the extra round trip is VALIDATION REQUIRED.

### Security defaults corrected

- `secrets.read` in the MCP diagram below is SUPERSEDED: core MCP exposes no raw secret capability ([MCP](../ai/mcp-architecture.md)).
- "No inheritance from development permissions" is enforced by [configuration](18-configuration.md): security-sensitive keys in the base file do not apply to non-development profiles.
- `MCP database.write enabled in production` and any production MCP data-read capability are **CRITICAL** checks, not warnings. Production code-execution capabilities are refused.
- Cookie-authenticated WebSocket and SSE handshakes require Origin validation ([16](16-realtime.md)).
- All framework-initiated outbound HTTP (webhooks, URL ingestion, provider callbacks) uses the egress-controlled client; configured identity-provider and service hosts are explicit allowlist entries in that client's policy, defined in [15](15-email-and-notifications.md) to prevent SSRF.
- Impersonation uses the actor/subject delegation chain; the subject never replaces the actor in audit.

### Validation required

Which policy forms translate to SQL for PostgreSQL; scope/check agreement under property tests; overhead of automatic scoping; RLS with `SET LOCAL` on pooled connections; developer comprehension of `public` versus policy declarations.

## Open decisions and interpretation

Concrete policy syntax, transport/auth details, approval protocols, security threat model, and operational controls require further design and adversarial validation.



<!-- Source: iii section 81. -->
## Authorization

> **Candidate 2 note:** route-level `#[permission]` below is adapter exposure only; business authorization is the operation policy, and permission inputs are the operation context, not the request. See the amendment above.

Basic permission:

```
#[permission(IsAuthenticated)]
```

More expressive:

```
#[permission(CanEditVenue)]
```

Permission traits receive:

```
identity
request
route metadata
optional resource
```

---

<!-- Source: iii section 82. -->
## Roles

Built-in role support:

```
Owner
Admin
Member
```

should be application-defined rather than globally hardcoded.

Role assignment can be scoped.

Example:

```
Russ
  Admin
  of Organization A

Russ
  Member
  of Organization B
```

---

<!-- Source: iii section 83. -->
## Object-Level Authorization

Required for serious applications.

Example:

```
CanEdit<Venue>
```

checks:

```
identity
+
specific Venue
```

Rjango should support resource authorization without requiring every application to reinvent the pattern.

---

<!-- Source: iii section 84. -->
## Permission Composition

Conceptually:

```
IsAuthenticated AND CanEditVenue
```

or:

```
IsAdmin OR IsVenueOwner
```

Exact Rust syntax remains OPEN until ergonomics are tested.

Metadata representation should support boolean permission trees regardless of surface syntax.

---

<!-- Source: iii section 85. -->
## Authorization + AMG

Route:

```
route:venues.update
```

links:

```
protected_by
permission:venues.CanEditVenue
```

This lets Rjango answer:

```
Which endpoints can anonymous users reach?

Which routes can mutate User?

Which routes require admin access?
```

That is extremely valuable for security review and AI agents.

---

<!-- Source: iii section 86. -->
## Service Accounts

Machine-to-machine access should use explicit service principals, not fake user records.

Service account:

```
identity
credentials
scopes
audit metadata
expiration
```

---

<!-- Source: iii section 87. -->
## Agent Identity

Because Rjango is AI-forward, agents should eventually be able to authenticate as explicit principals.

Example:

```
Agent:
deployment-assistant

Capabilities:
read schema
run tests
generate migration

Not allowed:
production database writes
secrets
user impersonation
```

AI actions become attributable.

---

<!-- Source: iii section 88. -->
## Impersonation

Admin/support impersonation, if enabled, must be explicit and auditable.

Audit record:

```
actor:
admin-user

acting_as:
customer-user

reason:
support case

started:
...

ended:
...
```

No silent identity swapping.

---

<!-- Source: iii section 89. -->
## Security Architecture

Rjango's security goal is:

> Secure defaults with explicit escape hatches.

The framework cannot make applications secure automatically, but it can eliminate entire categories of dangerous defaults.

---

<!-- Source: iii section 90. -->
## CSRF

Cookie-authenticated state-changing browser requests require CSRF protection.

Rjango should not apply CSRF blindly to:

```
Bearer-token APIs
```

where the threat model differs.

The framework understands authentication mode and applies the appropriate policy.

---

<!-- Source: iii section 91. -->
## CORS

Default:

```
same-origin
```

not:

```
*
```

Applications explicitly configure trusted origins.

Credentials plus wildcard origins must be rejected where invalid/insecure.

---

<!-- Source: iii section 92. -->
## Security Headers

Production defaults should include sensible support for:

```
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
frame protections
HSTS where properly deployed
```

But CSP cannot be safely universal without application awareness.

Rjango should provide strong configuration and diagnostics rather than a single magical policy.

---

<!-- Source: iii section 93. -->
## Trusted Hosts

Host-header validation should be available and production-recommended.

Example:

```
[security]
allowed_hosts = [
    "api.example.com"
]
```

---

<!-- Source: iii section 94. -->
## Trusted Proxies

Configured explicitly.

Example:

```
[server.proxy]
trusted = [
    "10.0.0.0/8"
]
```

Only then can forwarded client IP/scheme headers influence security decisions.

---

<!-- Source: iii section 95. -->
## Rate Limiting

Scopes:

```
global
IP
identity
API key
route
custom
```

Algorithms may be backed by:

```
memory
Redis
other distributed stores
```

Responses should use standard HTTP semantics.

---

<!-- Source: iii section 96. -->
## Request Limits

Framework safeguards:

```
maximum headers
maximum header size
maximum body
multipart limits
upload limits
JSON depth where practical
request timeout
```

Routes can selectively override.

---

<!-- Source: iii section 97. -->
## Secrets

Typed secret wrapper:

```
Secret<String>
```

Debug representation:

```
Secret([REDACTED])
```

Secrets must automatically redact from:

```
logs
error reports
metadata
MCP
debug output
```

unless explicitly and narrowly accessed by authorized application code.

---

<!-- Source: iii section 98. -->
## Secret Sources

Support:

```
environment variables
mounted files
secret managers/plugins
```

Rjango configuration references secret names, not necessarily values.

AMG knows:

```
DATABASE_URL
required
secret
```

without knowing the value.

---

<!-- Source: iii section 99. -->
## SQL Injection

Normal ORM/query APIs must bind parameters.

Raw SQL APIs should make parameter binding straightforward.

Dangerous interpolation should trigger documentation/compiler/lint warnings where detectable.

---

<!-- Source: iii section 100. -->
## XSS

JSON APIs largely avoid direct HTML rendering concerns.

If Rjango later provides templates:

```
escaping enabled by default
```

Raw/unescaped HTML must require explicit intent.

---

<!-- Source: iii section 101. -->
## Open Redirects

Redirect helpers should distinguish:

```
internal route redirect
```

from:

```
arbitrary external URL
```

External redirects should be explicit.

---

<!-- Source: iii section 102. -->
## File Security

Uploads require:

- generated storage names
- filename sanitization
- MIME awareness
- size limits
- no implicit executable serving
- configurable scanning hooks
- storage outside executable/source directories

Do not trust client filenames.

---

<!-- Source: iii section 103. -->
## Error Leakage

Development:

```
rich diagnostics
source locations
backtraces
SQL fingerprints
```

Production:

```
stable public error
request ID
safe context
```

No internal source path, stack, SQL value or secret leakage.

---

<!-- Source: iii section 104. -->
## Audit Log

Rjango should define a standard audit-event abstraction.

Example:

```
AuditEvent {
    actor,
    action,
    resource,
    result,
    request_id,
    timestamp,
    metadata,
}
```

High-value uses:

```
login
logout
password reset
permission change
API key creation
admin actions
MCP privileged operation
impersonation
migration execution
```

---

<!-- Source: iii section 105. -->
## MCP Security Boundary

MCP must not become a privileged side door.

The permission model should look like:

```
Agent Identity
      │
      ▼
MCP Authorization
      │
      ├── metadata.read
      ├── tests.run
      ├── migrations.generate
      ├── database.read
      ├── database.write
      └── secrets.read
```

These scopes are independent.

> **SUPERSEDED (Candidates 1 and 2):** `secrets.read` is not a core MCP capability; core MCP reports secret existence/source/rotation health only. `tests.run` is a code-execution capability ([MCP](../ai/mcp-architecture.md)).

---

<!-- Source: iii section 106. -->
## Production MCP

Default:

```
[mcp]
enabled = false
```

If enabled, production MCP must require authenticated transport and explicitly configured capabilities.

No inheritance from development permissions.

---

<!-- Source: iii section 107. -->
## MCP Confirmation Policy

Some tool operations should support human approval boundaries.

Example classification:

```
READ
inspect model

LOCAL WRITE
generate source migration

ENVIRONMENT WRITE
run migration

DESTRUCTIVE WRITE
drop production table
```

The MCP schema should communicate operation risk.

A client can then enforce approval workflows.

---

<!-- Source: iii section 108. -->
## Supply-Chain Security

Because Rjango will itself be infrastructure, the project spec should require:

```
dependency auditing
locked CI dependencies
security advisories
minimal feature activation
release provenance
SBOM capability
signed/reproducible release strategy where practical
```

Dependencies should be periodically reviewed for necessity.

---

<!-- Source: iii section 109. -->
## Unsafe Rust Policy

Framework-owned unsafe Rust should default to:

```
forbidden
```

unless there is a documented and reviewed technical reason.

Any unsafe block requires:

```
safety invariant documentation
tests
review justification
```

Ideally Rjango itself contains zero unsafe Rust and relies on lower-level audited crates for unsafe primitives.

---

<!-- Source: iii section 110. -->
## Security Diagnostics

`rjango check` should report things like:

```
CRITICAL
production debug mode enabled

CRITICAL
session cookies not Secure

WARNING
CORS accepts unexpected origin pattern

CRITICAL (Candidate 2; formerly WARNING)
MCP database.write enabled in production

WARNING
trusted proxy configuration accepts all networks

WARNING
password reset token lifetime unusually long
```

Checks need stable IDs.

Example:

```
RJG-SEC-014
```
