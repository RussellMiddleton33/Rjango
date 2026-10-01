# Realtime

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

WebSockets, SSE, typed channels, authorization, and bounded backpressure belong to one realtime model. Ephemeral messages do not acquire durable-delivery guarantees by implication.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Version evolution and permission revalidation

Realtime commands invoke shared operations. Negotiate supported protocol versions; messages identify type/schema version. Unsupported versions produce explicit upgrade/reconnect behavior. Rolling deployments have a declared compatibility window. Ephemeral delivery has no durable replay guarantee; persisted streams/cursors require separate retention contracts.

Authorize connect, subscribe, publish/command and protected delivery. Revalidate credential expiry/refresh, logout, role/tenant/resource changes and delegation revocation. Bounded periodic rechecks cover missed invalidations; recheck queued protected messages and evict revoked subscriptions. Declare/validate maximum stale-authorization windows; sensitive streams use authoritative checks or fail closed if policy is unavailable. Topic names/prior authentication are insufficient. Bound per-subscriber bytes/count; slow-client policy drops ephemeral messages, requests resync or disconnects explicitly.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H8, C1). Claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment where they conflict.

- **Origin validation (cross-site WebSocket hijacking).** Browsers attach cookies to cross-site WebSocket handshakes, and CSRF tokens and CORS do not protect them. Every cookie-authenticated **WebSocket** handshake requires an `Origin` header matching the configured trusted origins (same-origin by default). Browsers always send it on WebSocket handshakes, so a missing or mismatched origin is rejected before upgrade. For cookie-authenticated **SSE** (`EventSource` issues an ordinary GET, which omits `Origin` when same-origin), a missing `Origin` is accepted only when `Sec-Fetch-Site` is `same-origin` or `none`. A present `Origin` must match the trusted list, and anything else is rejected. Bearer-token connections, where the token is presented in the first message or a subprotocol and never in the URL, are exempt from Origin checks but not from authorization. Tokens in query strings are rejected by default because they leak into logs.
- **Channels are operations.** Subscribing to a channel is a Query operation with a required policy (deny by default). Publishing commands are Command operations. Channel names derived from IDs (`venue:123`) are scoped by tenant from `Ctx` and are never authority in themselves.
- **Revocation source.** Auth publishes invalidation events (logout, revocation, role or tenant change) that connected nodes consume ([09](09-authentication.md)). The periodic recheck interval bounds the stale-authorization window when an event is missed. The window is a PROPOSED default of 60 seconds for ordinary streams and 0 (authoritative check per message) for streams declared sensitive.

## Open decisions and interpretation

See the retained lock/open list. Distributed broker choice, reconnect guarantees, authorization revalidation, and cross-node semantics need fuller design.



<!-- Source: iv section 83. -->
## Realtime

### Goal

Rjango should treat realtime communication as a first-class application capability.

Support:

```
WebSockets
Server-Sent Events
broadcast groups
pub/sub
presence
```

---

<!-- Source: iv section 84. -->
## WebSockets

Route-like definition:

```
WebSocket endpoint
```

must participate in:

```
routing
auth
permissions
AMG
observability
rate limits
```

WebSockets must not become a security side channel around HTTP authorization.

---

<!-- Source: iv section 85. -->
## SSE

Server-Sent Events should be first-class because many realtime cases only require server → client streaming.

Advantages:

```
simpler protocol
automatic browser reconnection
HTTP infrastructure compatibility
```

---

<!-- Source: iv section 86. -->
## Channels

Conceptual abstraction:

```
channel:venue:123
```

Consumers subscribe to logical channels rather than individual worker processes.

---

<!-- Source: iv section 87. -->
## Groups

Example:

```
organization:abc
venue:123
user:456
```

A connection may join multiple groups after authorization.

---

<!-- Source: iv section 88. -->
## Distributed Pub/Sub

Single process:

```
memory
```

Multi-instance production:

```
Redis or another distributed transport
```

Core realtime API must not depend directly on one broker.

---

<!-- Source: iv section 89. -->
## Presence

Presence should be optional.

Presence state is inherently ephemeral.

Examples:

```
user online
editing venue
cursor location
```

Rjango should clearly distinguish:

```
durable application state
```

from:

```
presence state
```

---

<!-- Source: iv section 90. -->
## Realtime Authentication

Connection handshake authenticates identity.

Long-lived connections introduce new concerns:

```
session revoked
permission changed
token expired
```

Rjango must define how revalidation works.

Potential policies:

```
connection lifetime authentication
periodic revalidation
event-time authorization
```

High-risk actions should authorize at message handling time, not only at initial socket connect.

---

<!-- Source: iv section 91. -->
## Realtime Message Schemas

Messages should use typed Rjango schemas.

Example concept:

```
ClientMessage
ServerMessage
```

This feeds:

```
validation
TypeScript generation
documentation
MCP
```

---

<!-- Source: iv section 92. -->
## Backpressure

Slow clients must not consume unbounded server memory.

Per-connection outbound buffers need limits.

Policies:

```
disconnect
drop old message
drop new message
coalesce
```

depending on channel semantics.

---

<!-- Source: iv section 93. -->
## Reconnect

Realtime systems should support:

```
connection ID
optional resume cursor
event sequence IDs
```

where applications need replay.

Rjango should not imply guaranteed replay unless the event backend is durable.

---

<!-- Source: iv section 94. -->
## Realtime Events vs Durable Events

Important distinction:

```
WebSocket broadcast
→ ephemeral transport

Domain event
→ application fact

Job
→ durable work request
```

They should interoperate but are not interchangeable.

---

<!-- Source: iv section 95. -->
## Realtime + Generated Clients

Typed event definitions should generate TypeScript types.

Example:

```
VenueUpdated
PresenceChanged
```

frontend receives strongly typed events.

---

<!-- Source: iv section 136. -->
## Testing — Realtime

Required:

```
authentication
authorization
disconnect
reconnect
backpressure
message validation
large fan-out
multiple processes
broker outage
permission changes
expired credentials
slow consumers
```

---

<!-- Source: iv section 145. -->
## Current Lock Status — Realtime

#### LOCK

- WebSocket + SSE
- authentication/authorization integration
- typed message schemas
- distributed transport abstraction
- backpressure
- groups/channels
- AMG metadata

#### OPEN

- default distributed broker
- presence API
- durable replay API
