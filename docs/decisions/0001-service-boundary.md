# ADR 0001 — Python AI service, Java authoritative business operations

**Status:** Proposed  
**Context:** Atlas Hub runs a Java/Spring Boot modular monolith with DDD/CQRS/hexagonal boundaries. AI needs independent model compute, Python ML tooling, streaming speech and managed model releases.

## Decision
Build an independently deployable Python service for model inference, conversation lifecycle, orchestration, speech and speaker-risk inference. Keep the existing Java backend authoritative for authenticated identity, tenant permissions, approvals, HR, commerce, inventory, accounting and external integration state.

Use versioned internal gRPC/Protocol Buffers for typed Java–Python exchanges. Expose public clients through managed Java/API and appropriate browser/mobile streaming gateways. Model-planned actions must execute only via authorized Java commands after current-session permissions, business constraints and approval checks.

Self-hostable open-weight models are preferred for inference independence. The system does not require training a general foundational model from zero.

## Consequences
Positive: independent GPU scaling, strong contract boundaries, auditability and Java business invariants.
Tradeoffs: cross-language schema lifecycle, deployment orchestration, observability, network latency and tool execution security.
Mitigations: contract testing, typed errors, timeouts, idempotency, signed delegated identities, traces and benchmarks.

## Alternatives
- **Python code embedded inside Java monolith:** simpler deployment but poor separation for evolving ML/speech infrastructure.
- **REST between Java and Python:** viable and easier debugging, but less natural for typed bidirectional streaming. Remains acceptable if gRPC complexity outweighs measured benefit.
- **Voice authentication:** rejected as sole authentication due to spoofing and accessibility risks.

Review after prototype and security/latency evaluation.
