# Admin: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/11-admin.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

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
