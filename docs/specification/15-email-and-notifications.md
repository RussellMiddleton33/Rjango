# Email and notifications

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Email composition, provider transport, and durable delivery are separate concerns. Notification breadth remains open; side effects following database writes need transaction-aware dispatch.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H8, M1, M11). This topic had no Candidate 1 amendment; this section reconciles it with the [outbox](23-durability-and-message-contracts.md). Claims remain VALIDATION REQUIRED.

### Durable dispatch

Email, notifications and webhooks are written as outbox intents in the originating transaction (`tx.dispatch(SendEmail { .. })`). The "commit transaction, then dispatch email job" sequence in "Email and Jobs" below is SUPERSEDED: a crash between commit and dispatch would lose the email. Provider calls carry a stable message ID as a provider idempotency key where supported. Otherwise a delivery log keyed by message ID suppresses duplicates within the documented window.

### Egress-controlled outbound HTTP client (SSRF)

All framework-initiated outbound HTTP to destinations that are not fixed configuration uses `rjango::http::Client` with an **egress policy**. Examples are webhooks to customer URLs, URL ingestion, and link previews.

- **Schemes:** only `https` by default (`http` opt-in per destination class).
- **Blocked by default:** loopback, private (RFC 1918 and RFC 4193), link-local (including cloud metadata such as 169.254.169.254), multicast, unspecified and carrier-grade NAT ranges, for IPv4 and IPv6 including mapped forms.
- **Resolution checks:** the destination is checked **after DNS resolution and at connect time**, and the connection is pinned to the checked address to defeat DNS rebinding. Every redirect is re-checked; the redirect count is bounded.
- **Bounds and logging:** bounded connect, response and body-size limits. Response bodies are not reflected to callers. Outbound requests are logged with destination and policy decision.
- **Configured destinations:** fixed endpoints such as email providers and identity providers are allowlisted by name in configuration and bypass only the private-range rule they explicitly declare.

### Webhooks

Webhook payloads are signed with a per-endpoint secret over a timestamp plus body (illustrative: `Rjango-Signature: t=..., v1=...`). Receivers are documented to reject stale timestamps. Endpoint secrets are `Secret<T>` values stored encrypted at rest. Endpoints that fail continuously are disabled after a configured threshold, with audit. Webhook delivery runs as a job with at-least-once semantics, and payloads carry the envelope `message_id` for receiver deduplication.

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

> **SUPERSEDED (Candidate 2):** the email intent is written inside the transaction through the outbox, not dispatched after commit.

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
