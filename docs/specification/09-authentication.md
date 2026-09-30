# Authentication

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Authentication establishes an identity through interchangeable mechanisms. The user model is replaceable; credentials, sessions, service accounts, and agents have explicit security semantics.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

Provider choices and detailed credential recovery, OIDC, passkey, MFA, session-revocation and key-lifecycle flows require fuller design. Inclusion in the specification is not availability.



<!-- Source: iii section 71. -->
## Authentication Architecture

Authentication answers:

> Who is making this request?

Authorization answers:

> What are they allowed to do?

Never combine them into one abstraction.

---

<!-- Source: iii section 72. -->
## Identity

Core:

```
Identity
```

Possible principals:

```
Anonymous
User
ServiceAccount
APIKeyPrincipal
Agent
```

Authentication produces an identity.

Authorization evaluates that identity against requested actions/resources.

---

<!-- Source: iii section 73. -->
## User Model

Rjango should provide a strong default user implementation while allowing replacement.

Default conceptual fields:

```
id
email
password_hash
active
verified
created_at
updated_at
```

Do not force:

```
username
first_name
last_name
```

on all applications.

Email-first is the better modern default.

---

<!-- Source: iii section 74. -->
## Password Handling

Passwords:

- never stored
- never logged
- never returned through schemas
- hashed using a modern memory-hard password hash
- configuration versioned so hash parameters can be upgraded

Authentication should support transparent rehashing after login when policy changes.

---

<!-- Source: iii section 75. -->
## Sessions

Browser-oriented authentication uses secure opaque session identifiers.

Session state may live in:

```
database
Redis
other session backend
```

Cookies should default:

```
HttpOnly = true
Secure = true in production
SameSite = appropriate secure default
```

Session rotation after privilege changes/login is required.

---

<!-- Source: iii section 76. -->
## JWT

JWT support should exist for appropriate use cases but **not** be presented as universally superior to sessions.

Rjango documentation must explain:

- revocation tradeoffs
- lifetime
- refresh tokens
- audience
- issuer
- signing algorithms
- key rotation

---

<!-- Source: iii section 77. -->
## API Keys

First-class service authentication.

Stored server-side as hashes where possible.

Expose:

```
prefix
created time
last used
scopes
expiration
revocation state
```

Raw key shown only at creation.

---

<!-- Source: iii section 78. -->
## OAuth/OIDC

Rjango should support standards-first integration rather than building proprietary social login protocols.

Core abstractions should work with:

```
Google
Microsoft
GitHub
Okta
Auth0
enterprise identity providers
```

using OAuth/OIDC configuration.

---

<!-- Source: iii section 79. -->
## Passkeys

WebAuthn/passkeys belong in the 1.0 identity design even if not the first authentication mechanism implemented.

Identity architecture must not assume:

```
password exists
```

for every user.

---

<!-- Source: iii section 80. -->
## Multi-Factor Authentication

Pluggable factors:

```
TOTP
passkey/security key
recovery codes
```

SMS may be supported but should not be framed as the strongest factor.
