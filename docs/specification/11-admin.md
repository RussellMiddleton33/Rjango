# Admin

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Admin is an explicitly enabled projection of shared model metadata with separate admin configuration, object-level authorization, redacted audit history, and extensible views.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Mutation modes

Admin explicitly declares ReadOnly, DirectCRUD or OperationBacked per action/resource. ReadOnly has no write path. DirectCRUD is deliberate simple-model exposure and still enforces validation, scopes, field permissions, concurrency, transactions and audit. Domain transitions/durable effects use OperationBacked commands with explicit form mappings; no fallback direct write may bypass invariants. Bulk mutations declare one-database atomicity or explicit per-item outcomes/recovery. UI visibility never substitutes for server-side policy.


## Open decisions and interpretation

See the retained lock/open list. UI technology, detailed visual design, extension packaging, and final action syntax are not settled.



<!-- Source: iv section 2. -->
## Rjango Admin

### Goal

Rjango Admin should become one of the framework's defining productivity features.

Django demonstrated how powerful it is when:

```
model definition
        ↓
working internal administration interface
```

Rjango should preserve that advantage while improving:

- type safety
- permissions
- auditing
- API separation
- extensibility
- agent introspection
- modern frontend UX

---

<!-- Source: iv section 3. -->
## Admin Is a Projection

Admin must not maintain another independent description of application models.

Instead:

```
AMG model metadata
        +
Admin-specific metadata
        ↓
Admin UI
```

For example AMG already knows:

```
Venue
 ├── id
 ├── name
 ├── slug
 ├── active
 └── floors
```

Admin metadata adds:

```
list columns
search fields
filters
editable fields
actions
display labels
layout
```

---

<!-- Source: iv section 4. -->
## Explicit Admin Registration

A model should not automatically become administratively accessible merely because it exists.

Conceptually:

```
#[rjango::admin(Venue)]
pub struct VenueAdmin {
    // ...
}
```

or an equivalent typed builder.

**Exact syntax remains OPEN.**

Architectural rule:

> Admin exposure is explicit.

---

<!-- Source: iv section 5. -->
## Default Admin Experience

Minimal registration should provide sensible CRUD automatically.

Given:

```
Venue
```

Rjango may derive:

```
List
Create
Detail
Edit
Delete
Relationships
Search
Filtering
Pagination
```

subject to permissions.

---

<!-- Source: iv section 6. -->
## Admin Views

Core view types:

```
List
Detail
Create
Update
Delete
History
```

Later:

```
Dashboard
CustomPage
Wizard
Report
```

Admin should allow custom pages without requiring users to fork the entire frontend.

---

<!-- Source: iv section 7. -->
## List Configuration

Admin metadata should support:

```
columns
search
filters
ordering
pagination
date navigation
relationship summaries
computed columns
```

Example concept:

```
list_display = [
    Venue::F.name,
    Venue::F.slug,
    Venue::F.active,
    Venue::F.created_at,
]
```

Exact syntax remains open.

---

<!-- Source: iv section 8. -->
## Search

Admin search should translate only explicitly allowed fields into queries.

Never allow arbitrary field names from the browser to become raw SQL.

Potential supported search strategies:

```
exact
contains
case-insensitive contains
prefix
PostgreSQL full-text
```

---

<!-- Source: iv section 9. -->
## Filters

Filters should derive from field metadata where sensible:

```
bool
enum
date
foreign key
numeric range
```

Custom filters can expose typed query logic.

---

<!-- Source: iv section 10. -->
## Admin Forms

Admin forms derive from:

```
model fields
schema validation
relationships
custom admin configuration
```

But admin mutation schemas should still be explicit enough to prevent internal-only fields from becoming editable accidentally.

---

<!-- Source: iv section 11. -->
## Read-Only Fields

Example:

```
created_at
updated_at
audit actor
generated IDs
computed values
```

may be rendered but not editable.

Metadata should express this explicitly.

---

<!-- Source: iv section 12. -->
## Relationship Editing

Support:

```
foreign key selector
one-to-one
inline has-many
many-to-many
```

But potentially expensive relationship widgets should not automatically load enormous datasets.

For large relations:

```
search/autocomplete
```

should replace giant dropdowns.

---

<!-- Source: iv section 13. -->
## Bulk Actions

Examples:

```
archive
activate
export
send notification
```

Bulk actions need:

```
authorization
confirmation
audit trail
result summary
failure handling
```

Destructive actions must not be single-click accidental operations.

---

<!-- Source: iv section 14. -->
## Admin Actions

Conceptually:

```
#[rjango::admin_action]
async fn archive_venues(...) -> Result<...>
```

Admin action metadata includes:

```
name
description
required permission
risk classification
supports bulk
async/job-backed
```

---

<!-- Source: iv section 15. -->
## Admin Permissions

Admin access must use normal Rjango authorization.

Not:

```
is_admin = true
→ unlimited everything
```

Instead policies can differentiate:

```
view Venue
create Venue
edit Venue
delete Venue
run Export action
view audit log
```

---

<!-- Source: iv section 16. -->
## Object-Level Admin Permissions

Admin must support object-level checks.

Example:

```
User can edit venues belonging to Organization A
but not Organization B
```

List queries should ideally be scoped before objects are returned rather than fetching forbidden objects and hiding them afterward.

---

<!-- Source: iv section 17. -->
## Admin Audit History

Admin mutations should automatically produce audit events.

Example:

```
Actor:
user:123

Action:
admin.update

Resource:
venues.Venue:456

Changes:
name:
  old: "Center"
  new: "The Center"

Request:
req_abc123
```

Sensitive fields must be redacted.

---

<!-- Source: iv section 18. -->
## Admin Authentication

Production admin should strongly support:

```
MFA
SSO
passkeys
shorter sessions
re-authentication for sensitive actions
```

The framework can recommend stricter security policies for admin than for normal end-user sessions.

---

<!-- Source: iv section 19. -->
## Admin UI Architecture

The Admin frontend should be decoupled from application business APIs.

Preferred architecture:

```
Admin UI
   │
   ▼
Rjango Admin API
   │
   ▼
Admin metadata + permissions
   │
   ▼
Models/services
```

This avoids requiring public application APIs to expose internal admin functionality.

---

<!-- Source: iv section 20. -->
## Admin Customization

Three levels:

#### Level 1

Metadata configuration.

#### Level 2

Custom fields/widgets/actions.

#### Level 3

Custom admin pages/components.

This provides escape hatches without forcing every project to maintain a custom SPA.

---

<!-- Source: iv section 21. -->
## Admin + MCP

Read tools:

```
inspect_admin
inspect_admin_model
inspect_admin_permissions
```

Agents should be able to explain:

```
Why isn't this field editable?

What permissions protect Venue deletion?

Which admin actions are destructive?
```

Admin actions themselves do **not** automatically become MCP actions.

---

<!-- Source: iv section 22. -->
## Coming from Django — Admin

```
Django ModelAdmin
→ Rjango Admin metadata

list_display
→ list columns

search_fields
→ search metadata

list_filter
→ filters

readonly_fields
→ read-only metadata

admin actions
→ typed audited actions
```

Major Rjango difference:

> Admin permissions, AMG metadata and audit trails are designed together instead of being loosely connected systems.

---

<!-- Source: iv section 131. -->
## Testing — Admin

Required:

```
permissions
object permissions
CRUD
search
filters
relationship editing
audit trail
sensitive field redaction
bulk actions
destructive confirmation
large datasets
authentication
CSRF
```

---

<!-- Source: iv section 140. -->
## Current Lock Status — Admin

#### LOCK

- Admin derived from AMG
- explicit model exposure
- normal authorization system
- audit events
- object-level permissions
- separate internal admin API
- safe bulk-action handling

#### OPEN

- frontend technology
- exact configuration syntax
- customizable widget API
