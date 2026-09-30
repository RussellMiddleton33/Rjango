# Storage and files

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Storage exposes logical file handles and backend capabilities, with streaming I/O, untrusted upload metadata, private defaults, and explicit cleanup semantics.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

See the retained lock/open list. Public file-field types, lifecycle details, and provider capability contracts remain open.



<!-- Source: iv section 59. -->
## Storage & Files

### Goal

Provide one storage abstraction for:

```
local development
S3-compatible object storage
cloud providers
custom backends
```

Application code should not hardcode filesystem paths.

---

<!-- Source: iv section 60. -->
## Storage Backend

Conceptual trait:

```
StorageBackend
```

Capabilities:

```
write
read
stream
delete
exists
metadata
copy/move where supported
signed URL
```

Capabilities may vary by backend.

The abstraction should expose capability differences rather than pretending every backend is identical.

---

<!-- Source: iv section 61. -->
## Storage Handles

Application code should store logical file references, not arbitrary local paths.

Example:

```
uploads/venues/123/map.glb
```

plus storage backend identity if required.

---

<!-- Source: iv section 62. -->
## Streaming Upload

Preferred:

```
HTTP stream
    ↓
validation/checksum
    ↓
storage stream
```

not:

```
HTTP
→ full RAM buffer
→ storage
```

---

<!-- Source: iv section 63. -->
## Direct-to-Object-Storage Uploads

Large uploads should support:

```
client
  ↓
signed upload URL
  ↓
S3
```

then application verifies/finalizes.

This reduces application-server bandwidth.

---

<!-- Source: iv section 64. -->
## Upload State

Potential lifecycle:

```
Pending
Uploaded
Verified
Available
Quarantined
Deleted
```

Useful when antivirus/media-processing workflows are involved.

---

<!-- Source: iv section 65. -->
## Checksums

Storage API should support checksums.

Benefits:

```
integrity
deduplication
upload verification
cache validation
```

---

<!-- Source: iv section 66. -->
## Content Type

Client-provided content type is untrusted metadata.

Rjango may retain:

```
declared MIME
detected MIME
```

when available.

---

<!-- Source: iv section 67. -->
## Filename Safety

Original filenames are metadata only.

Storage key should generally be generated.

Prevent:

```
../../something
```

path traversal.

---

<!-- Source: iv section 68. -->
## Signed URLs

Support:

```
signed read URL
signed upload URL
expiration
content constraints
```

Signing secrets remain in the backend/provider.

---

<!-- Source: iv section 69. -->
## Public vs Private Objects

Explicit storage visibility:

```
private
public
```

Default should favor private for uploaded user content unless configured otherwise.

---

<!-- Source: iv section 70. -->
## Deletion Semantics

Deleting a DB row does not necessarily imply immediate external file deletion.

Storage cleanup may need durable background jobs.

The framework should support:

```
mark deleted
commit DB
dispatch cleanup job
```

rather than risking transactional mismatch.

---

<!-- Source: iv section 71. -->
## Model/File Integration

Potential model type:

```
StoredFile
```

rather than simply:

```
String path
```

It can carry:

```
key
backend
size
checksum
content type
metadata
```

Exact field integration remains OPEN.

---

<!-- Source: iv section 72. -->
## Storage Security Hooks

Support callbacks/integrations for:

```
virus scanning
media validation
image transcoding
document sanitization
```

Framework must not falsely claim uploaded files are safe merely because extension/MIME checks pass.

---

<!-- Source: iv section 134. -->
## Testing — Storage

Required:

```
streaming
large files
empty files
invalid filenames
path traversal attempts
checksum mismatch
provider failure
partial upload
delete failure
signed URL expiry
private/public access
```

---

<!-- Source: iv section 143. -->
## Current Lock Status — Storage

#### LOCK

- backend abstraction
- streaming
- local + object storage
- private-by-default capability
- signed URLs
- path safety
- checksum support
- background cleanup compatibility

#### OPEN

- exact model field representation
- multipart/resumable API surface
- scanning integration standard
