# Email and notifications

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Email composition, provider transport, and durable delivery are separate concerns. Notification breadth remains open; side effects following database writes need transaction-aware dispatch.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

See the retained lock/open list. Notification channels, templating choices, and backend guarantees remain open.



<!-- Source: iv section 73. -->
## Email & Notifications

### Goal

Rjango should make transactional communication simple while separating:

```
message composition
delivery provider
durability
```

---

<!-- Source: iv section 74. -->
## Email Interface

Conceptual:

```
Email::new()
    .to(...)
    .subject(...)
    .text(...)
    .html(...)
```

Exact API remains open.

---

<!-- Source: iv section 75. -->
## Email Backends

Potential providers:

```
SMTP
AWS SES
Postmark
Resend
SendGrid
custom
```

Provider differences should be encapsulated behind a standard delivery interface with provider-specific extensions where needed.

---

<!-- Source: iv section 76. -->
## Email Address Types

Use typed values where possible:

```
EmailAddress
Mailbox
```

rather than arbitrary strings everywhere.

---

<!-- Source: iv section 77. -->
## Development Email Backend

Rjango should provide an excellent local-development experience.

Possibilities:

```
console backend
in-memory test inbox
local web inbox
```

A developer should not need to actually send email to test password reset flows.

---

<!-- Source: iv section 78. -->
## Email Templates

Templates should be separate from mail delivery.

Support:

```
text
HTML
```

with shared template context.

Templates should be testable independently.

---

<!-- Source: iv section 79. -->
## Email and Jobs

Production email should normally be job-backed.

Request:

```
create account
   ↓
commit transaction
   ↓
dispatch email job
   ↓
return HTTP response
```

Users should not wait on SMTP/provider latency unnecessarily.

---

<!-- Source: iv section 80. -->
## Delivery Tracking

Provider integrations may expose:

```
queued
sent
delivered
bounced
complained
```

These states should not be assumed universally available.

---

<!-- Source: iv section 81. -->
## Notification Abstraction

Potential future common abstraction:

```
Notification
```

channels:

```
email
SMS
push
in-app
webhook
```

But do not force radically different channels into an overly generic abstraction prematurely.

**STATUS: OPEN FOR 1.0 SCOPE**

Email itself is definitely required.

---

<!-- Source: iv section 82. -->
## Webhooks

Outbound webhooks fit near notification infrastructure but deserve explicit semantics:

```
signed payload
retry
idempotency
delivery log
dead-letter behavior
```

Likely job-backed.

---

<!-- Source: iv section 135. -->
## Testing — Email

Required:

```
composition
template rendering
headers
invalid addresses
provider timeout
provider retry
job dispatch
secret redaction
test inbox
```

---

<!-- Source: iv section 144. -->
## Current Lock Status — Email/Notifications

#### LOCK

- pluggable email backend
- development backend
- templates
- job integration
- typed addresses
- observability

#### OPEN

- generic notification abstraction
- exact template technology
- which providers receive first-party adapters
