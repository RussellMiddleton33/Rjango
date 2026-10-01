# MCP architecture and security boundaries

[Master specification](../specification/README.md) · [Operations](../specification/22-application-operations-and-services.md) · [Security](../specification/10-authorization-and-security.md)

**Status:** REVIEW CANDIDATE 2. Boundaries adopted; transport/schema/version details and conformance remain VALIDATION REQUIRED.

## Planes and exposure

Definition metadata, runtime bindings and operational observations are distinct. Business records and secret values are not AMG metadata. Filter Public/Internal/DevelopmentOnly/Restricted resources by identity/environment; redact production source paths and sensitive structures. Inspection includes provenance, completeness, schema version and relevant projection fingerprints. Metadata visibility never grants execution/data authority.

## Local versus remote authentication

Local development stdio uses the local OS user/process trust boundary, explicit environment selection and allowlisted capabilities. Local ownership does not authorize production access. Avoid credentials in arguments/logs and never implicitly reuse an unrestricted developer shell identity for remote privileges.

Remote MCP requires TLS and explicit network authentication/authorization aligned with a pinned, verified supported MCP authorization revision. Validate issuer, audience/resource, expiry and scopes/capabilities; document discovery, refresh and revocation behavior. Reject arbitrary upstream bearer-token passthrough to downstream services. Use separate least-privilege downstream credentials; prevent confused-deputy escalation and cross-resource token reuse. OAuth/OIDC integration must follow the supported MCP contract rather than an invented privileged session. Exact revision/provider support remains open; no conversation-derived protocol revision is claimed validated.

Production MCP is disabled by default. Enabling transport starts with zero application-data access and explicit metadata visibility policy. Rate/concurrency/body/time limits, audit and tenant/delegation checks apply to each call.

## Capabilities

| Capability | Required boundary |
| --- | --- |
| Metadata inspection | Filter/redact structural resources independently of data. |
| Application-approved query | Explicit scoped operation; tenant/object/field policy and bounded output. |
| Mutation/action/job | Separate explicit exposure, shared operation authorization, preconditions and audit. |
| Migration generation | Reviewable artifact only; no permission to apply. |
| Migration execution | Independent environment/risk/approval policy and migration lock/checksum safeguards. |
| Secret diagnostics | Existence/source/rotation/validation state only; never raw values. |
| Local development raw database tooling | Explicit development-only capability and environment guard; no broad production default. |

Core Rjango exposes **no raw secrets.read tool/capability**. Applications needing exceptional secret tools own a separate privileged boundary outside core MCP. Production data tools prefer approved queries, sanitized summaries and aggregates; registration of a model never enables arbitrary production SQL. Shell/command execution is separately exposed and constrained, not implied by MCP authentication.

## Actor/subject and delegation

Record the executing agent/service actor, human/service subject and ordered attenuated delegation chain. Authority is limited by every link, tenant, environment, expiry/revocation and current policy. Neither a launch by a human nor a tool description grants delegation. Audit operation/tool, decision/result, correlation/idempotency IDs and relevant fingerprints without credentials or sensitive payloads.

## Stale-fingerprint and idempotency protections

Reviewable mutations bind expected operation/auth/MCP projection fingerprints, target environment/resource, canonical input digest and required concurrency preconditions. Re-evaluate current authorization and those preconditions immediately before execution. Stale contracts reject with structured diagnostics and require reinspection/replanning; an old preview is not approval. A matching fingerprint does not prove unchanged business row state, so row versions/ETags/locks remain necessary.

Retryable commands require scoped idempotency keys. Persist claims/outcomes with the operation's database changes where feasible. Concurrent retries do not execute twice; same key/different input conflicts. Unknown outcome requires reconciliation, not blind replay. Retention/expiry and in-progress response are declared. Returning cached sensitive outcomes still requires current authorization. MCP transport retries do not provide business exactly-once semantics. Destructive/production operations enforce applicable approval policy; neither tokens nor previews silently broaden capability.

## Review Candidate 2 amendment

From the [independent review](../reviews/independent-review-candidate-1.md) (H8, M2, L6). Claims remain VALIDATION REQUIRED.

### Untrusted content and prompt injection

Tool and resource results that contain application data, logs, error messages, user-supplied strings, file contents or third-party responses are **untrusted content**.

- **Labelling.** Results label such fields structurally (illustrative: `{"untrusted": true, "source": "logs", "content": ...}`), separate from framework-authored fields, and escape them so they cannot impersonate framework metadata.
- **No instruction-following.** Rjango never executes instructions found in data. A tool result never grants or broadens capability.
- **Approval channel.** Privileged and destructive actions require approval bound to the specific action digest and delivered through a channel outside the agent conversation: the MCP client's confirmation UI, CLI confirmation, or an approval record by an authorized human. An agent-produced "the user approved" is never accepted.
- **Rate limits.** Inspection tools that return untrusted content (logs, recent errors) have their own capability and are rate-limited.

### Code execution is its own capability class

`tests.run`, `run_check` when it compiles code, `rjango run`, migration data steps and shell execution all execute project code, including build scripts and proc macros, with the invoking user's authority. They form the **CodeExecution** capability class:

- available only to local development MCP under the local OS trust boundary;
- never enabled in production;
- never implied by metadata or migration-generation permission;
- each invocation audited.

The static portion of `rjango check` in metadata-only mode ([24](../specification/24-developer-experience-and-diagnostics.md)) is not code execution against data, but it still runs the compiled application binary and is classified accordingly.

### Development MCP binding

`rjango dev` exposes development MCP over stdio, or on loopback only with a per-session token. It defaults to metadata read and migration generation. Data access requires an explicit local grant and a development-marked database ([04](../specification/04-migrations.md)). A development MCP connected to a database whose ledger is not development-marked refuses data tools, whatever the local configuration says.

### Fingerprints are not behavioural proof

Stale-fingerprint protection detects changes to declared metadata and annotated policy bodies only ([02](../specification/02-application-metadata-graph.md)). Execution always re-runs current authorization.

## Open validation

Pin protocol/auth versions, transport/resource schemas, capability grammar, credential storage, delegation revocation, retention, limits and confirmation binding. Test local-vs-production guards, wrong issuer/audience, expiry/revocation, passthrough rejection, tenant leakage, stale metadata/row state, duplicate/conflicting calls, cancellation and sanitized diagnostics. No unvalidated protocol claim authorizes implementation.
