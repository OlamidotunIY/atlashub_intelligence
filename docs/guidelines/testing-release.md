# Atlas Intelligence — test architecture and delivery gates

**Status:** Planned. Separate Python CI/CD (no Turborepo/Nx).

## Test pyramid
- **Domain unit:** tool/risk/action state machine, tenant scoping, conversation retention, permission fallback, voice risk consent/revocation, cost/attempt limits.
- **Application with fake ports:** submit turn, fetch authorized Java evidence, propose action, cancel, approve, execute and truthfully surface outcomes; reject zero-org business action; allow authorized nonbusiness conversation.
- **Contract:** Python-generated versioned protobuf vs Java tool gateway, authentication context schema, error/deadline handling and backwards-compatible field additions. Bidirectional streaming audio event versioning.
- **Integration:** real self-hosted inference endpoint (staging), Java fake tool gateway and selected sandbox workflows, Postgres/Redis only if chosen, secure gRPC/mTLS and rotating short-lived tokens.
- **Adversarial:** prompt injection via product titles/emails/pages, cross-tenant tool/data exfiltration, org spoofing, fake approval, replay/race of high-risk commands, hallucinated transfer confirmation, infinite tool loops, model serving SSRF.
- **Speech:** noisy/accented/changed voice ASR quality, interruption/barge-in, diarization, streaming latency, speaker spoof replay/clone attack resistance, false rejection/acceptance population/environment testing and strong nonvoice fallback.
- **Operational:** GPU CPU/VRAM budgets, inference concurrency/time-to-first-token, gateway p95/p99, cancellation, queue overload, failclosed Java unreachable, degraded read-only fallback and observability.

## CI gates
Formatting/lint + strict type checks, pytest unit/contract, proto generation diff, dependency/security scanning, mock-model deterministic tests in standard CI. GPU/voice tests in separate controlled staging workflow with explicit resources; don't require a developer laptop to run expensive models. Containers/cloud provider and model weights pinned with license/security review. Secrets injected at runtime only.

## Definition of done
Feature has documented bounded context and port contracts, safety/identity test cases, Java authority test, redacted audit, rollback/degraded state, model version/eval baseline and docs. No production rollout solely because chatbot responds naturally or voice resembles enrolled speaker.

## Release rules
Version prompt, model, tool schema and protobuf independently. Canary risk evaluation, reproducible evaluation suites, model rollback and auditable tool execution logs. Java/AI deployments can roll independently if schema compatibility preserved. Never expose an unapproved high-risk tool before step-up/permissions tests.
