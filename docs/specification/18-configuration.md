# Configuration and environments

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Typed settings merge through deterministic layers with provenance, startup validation, and secret redaction. Dynamic reload and administrative mutation are separate explicit capabilities.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Secret values and resource budgets

Secret<T> is nonserializable by default: ordinary serialization must not reveal contents. Debug/display, telemetry, errors, AMG, docs and core MCP are redacted. Exposure requires a narrowly named deliberate application action, never implicit conversion/blanket serialization. Redaction is not memory zeroization; copying/retention/zeroization require separate validation. Secret introspection exposes existence/source/rotation health only. Validate resource/shutdown budgets and reject unsupported unbounded or contradictory settings.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H7, M1). Claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Environment selection fails safe

- The active environment comes from `RJANGO_ENV` or an explicit CLI flag.
- **If it is unset, the environment is `production`.** The application never falls back to development.
- `rjango dev` and `rjango test` set `development` and `test` explicitly.
- A production process whose configuration enables a development-only capability refuses to start with a CRITICAL diagnostic. Development-only capabilities include debug error pages, development MCP, `db sync`, development email inboxes and auto-applied migrations.

### Security-sensitive keys are not inherited from the base file

The layering below still applies, with one exception. Keys classified **security-sensitive** in the configuration schema, when set in the base `rjango.toml`, apply only to the `development` and `test` profiles. Every other profile (production, staging, custom) uses the framework's secure default unless the key is set in that profile's file, environment variables or runtime override. Security-sensitive keys include:

- `mcp.*`
- debug and error detail
- CORS origins
- cookie `Secure`/`SameSite`
- trusted proxies and hosts
- metadata HTTP exposure
- development tools
- rate-limit disablement
- realtime trusted origins

`rjango config show` displays ignored base values with the reason. This implements "No inheritance from development permissions" ([10](10-authorization-and-security.md)). The historical template in [initial architecture](history/initial-architecture.md), which sets `environment = "development"` and `[mcp] enabled = true` in the base file, is SUPERSEDED. The generated project puts development values in `rjango.development.toml`.

### Secrets and MCP

"`secrets.read` remains an extraordinarily privileged separate capability" in "Configuration + MCP" below is SUPERSEDED. Core MCP has no raw-secret capability ([MCP](../ai/mcp-architecture.md)). `Secret<T>` implements `Deserialize` (so it can be loaded) but not `Serialize`; values are revealed only by an explicitly named method in application code.

### Reusable-app configuration

App configuration structs from [05](05-app-system.md) are part of the configuration schema under the app's namespace, with code-supplied defaults overridable per environment.

## Open decisions and interpretation

See the retained lock/open list. Live reload classes, secret refresh, feature flags, and configuration mutation need fuller design.



<!-- Source: iv section 109. -->
## Configuration

### Goal

Configuration should be:

- typed
- validated
- layered
- environment-aware
- secret-safe
- inspectable
- deterministic

---

<!-- Source: iv section 110. -->
## Primary Configuration

Proposed default file:

```
rjango.toml
```

Example:

```
[app]
name = "acme"

[server]
host = "127.0.0.1"
port = 8000

[database]
url_env = "DATABASE_URL"

[jobs]
enabled = true

[cache]
backend = "redis"

[mcp]
enabled = false
```

---

<!-- Source: iv section 111. -->
## Layering

Recommended precedence:

```
framework defaults
       ↓
rjango.toml
       ↓
environment profile
       ↓
environment variables
       ↓
explicit runtime override
```

Every final value should be traceable to its source without exposing secret contents.

---

<!-- Source: iv section 112. -->
## Environment Profiles

Conceptually:

```
development
test
staging
production
```

But environments should be configurable rather than a closed enum if applications need custom profiles.

---

<!-- Source: iv section 113. -->
## Environment Variables

Variables should map predictably.

Example concept:

```
RJANGO_SERVER_PORT
RJANGO_DATABASE_MAX_CONNECTIONS
```

Secrets can be referenced through application-specific environment names.

---

<!-- Source: iv section 114. -->
## Typed Application Settings

Applications may define:

```
#[rjango::settings]
pub struct BillingSettings {
    pub tax_rate: Decimal,
    pub invoice_prefix: String,
}
```

Validation occurs at startup.

---

<!-- Source: iv section 115. -->
## Required Settings

Missing required configuration should fail before serving traffic.

Bad:

```
server starts
first request
→ crash because API key absent
```

Good:

```
startup validation
→ clear error
```

---

<!-- Source: iv section 116. -->
## Secret Type

Conceptually:

```
Secret<T>
```

Features:

```
redacted Debug
redacted Display
excluded from AMG values
excluded from logs
explicit reveal operation
```

---

<!-- Source: iv section 117. -->
## Configuration Introspection

Command concept:

```
rjango config show
```

Output:

```
server.port
  value: 8000
  source: rjango.toml

database.url
  value: [REDACTED]
  source: environment:DATABASE_URL
```

This can make deployment debugging dramatically easier.

---

<!-- Source: iv section 118. -->
## Configuration Schema

Rjango should be able to serialize a machine-readable configuration schema.

Used by:

```
IDE
docs
MCP
deployment tooling
validation
```

---

<!-- Source: iv section 119. -->
## Invalid Configuration

Errors should point directly to:

```
key
source
expected type
actual value
constraint
```

Example:

```
RJG-CONFIG-014

database.max_connections must be greater than 0.

Source:
rjango.production.toml

Found:
0
```

---

<!-- Source: iv section 120. -->
## Dynamic Configuration

Not every setting should be hot-reloadable.

Classify settings:

```
Static
Reloadable
SecretReloadable
```

Examples:

```
server port
→ static

log level
→ potentially reloadable

API provider key
→ potentially secret reloadable
```

Exact runtime reload capabilities remain OPEN.

---

<!-- Source: iv section 121. -->
## Feature Flags

Application feature flags should not be confused with Cargo feature flags.

Runtime application flag system may eventually support:

```
percentage rollout
organization scope
user scope
environment
```

But this is not necessarily a core Rjango 1.0 requirement.

**STATUS: OPTIONAL / PLUGIN CANDIDATE**

---

<!-- Source: iv section 122. -->
## Configuration + MCP

Safe tools:

```
inspect_configuration_schema
inspect_configuration_sources
validate_configuration
```

By default they return:

```
secret present: true
```

not:

```
secret value: abc123
```

> **SUPERSEDED (Candidates 1 and 2):** core MCP has no `secrets.read` capability.

`secrets.read` remains an extraordinarily privileged separate capability.

---

<!-- Source: iv section 123. -->
## Configuration + Admin

Admin should not automatically expose framework configuration editing.

Infrastructure configuration belongs outside the normal application admin unless explicitly implemented.

---

<!-- Source: iv section 138. -->
## Testing — Configuration

Required:

```
precedence
environment overrides
invalid values
missing values
secret redaction
profile behavior
Unicode
malformed TOML
unknown keys
deprecated settings
```

---

<!-- Source: iv section 147. -->
## Current Lock Status — Configuration

#### LOCK

- typed settings
- TOML default configuration
- environment overrides
- profiles
- startup validation
- secret type/redaction
- introspection
- machine-readable config schema

#### OPEN

- exact environment file layout
- dynamic reload scope
- runtime feature flags
