# Atlas Intelligence

> **Status:** Documentation-first architecture. This repository does not yet contain a runnable AI service.

Python service for Atlas Hub's conversational and voice-enabled business assistant, including self-hosted model inference, authorized tool planning, conversation orchestration, evidence grounding, and optional speaker-risk assessment.

## Boundaries

- **Atlas Hub backend (Java)** owns identity, authenticated sessions, organization scopes, RBAC, entitlements, business data, command handlers, approvals, transactional truth and accounting.
- **Atlas Intelligence (Python)** owns conversation processing, tool proposals, self-hosted inference, speech-to-text, text-to-speech, model evaluation, and privacy-aware voice-risk signals.
- Services communicate through **private versioned gRPC/Protocol Buffers**, with mutually authenticated service identities. Java validates authorization again before executing any proposed action.
- Voice similarity **must never serve as sole authentication** or authorize a payment, refund, payroll disbursement, or high-risk action.
- No unrestricted direct access to Atlas Hub's business database.

## Proposed design

Domain-Driven Design, Clean/Hexagonal Architecture and CQRS for meaningful stateful commands/queries. No mechanical CQRS for individual audio chunks.

Read the documents before writing application code:

- [Architecture and bounded contexts](docs/architecture/system.md)
- [Module-by-module implementation plan](docs/modules/index.md)
- [Voice identity, anti-spoofing and user recovery](docs/security/voice-identity.md)
- [Internal gRPC contracts](docs/contracts/grpc.md)
- [ADR 0001: Java/Python service boundary](docs/decisions/0001-service-boundary.md)

## Delivery principles

Start with read-only text tools, then authorized mutation tools with previews and approvals. Add streaming speech and self-hosted model serving based on measured performance, consent and security testing. Provider independence means ownership of inference/agent orchestration; it does not require training a frontier language model from zero.

**Planning only.** No module, API, ML model, or speaker verification system should be described as implemented until tested and released.


## Global user identity and provider-connected businesses (2026-10-25 decision)

Global `User` registration occurs through standalone [Auth Web](https://github.com/OlamidotunIY/atlashub_auth_web) and Java identity (no Organization required). Separate authenticated organization creation provides OWNER membership; an individual creator/AI user may have zero organizations. Storefront creator workspace uses [separate Portal frontend](https://github.com/OlamidotunIY/atlashub_storefront_portal). The Java backend is moving from compulsory Atlas-owned Anchor banking/KYB to **merchant-connected eligible payment/shipping providers**. Intelligence tools must check actual Java IAM/organization context and integration capabilities; never fabricate bank balances, execute unavailable provider operations, or claim financial transfers succeed without verified provider outcome. Provider onboarding/verification remains required where applicable. The Auth frontend is separate; AI is not an identity provider. See [Java v2 PRD](https://github.com/OlamidotunIY/atlashub_backend/blob/docs/global-platform-ai-storefront-architecture/docs/atlashub-prd-v2.md).


## Required Python engineering architecture

Use **DDD + Clean Architecture + Hexagonal ports/adapters** for conversation, tool authorization and model/speech business policies. Keep the Python domain pure and infrastructure-neutral. gRPC clients, local model runtimes, speech engines, persistence and telemetry are adapters; Java business tools remain authoritative. Apply CQRS to meaningful stateful operations, not every stream frame/token.

- [Core architecture, bounded contexts and Java authority](docs/architecture/clean-hexagonal.md)
- [Python package/folder layout and import rules](docs/architecture/folder-structure.md)
- [Engineering/security/voice rules](docs/guidelines/engineering-rules.md)
- [Testing, evaluation and deployment gates](docs/guidelines/testing-release.md)
- [Module-by-module backlog](docs/modules/index.md)

Standalone Python repository with normal Python tooling; no Turborepo/Nx. The infrastructure, AI engine and production speaker verification are not implemented yet.


## Shared Protobuf contracts (2026-10-26)

[atlashub_contracts](https://github.com/OlamidotunIY/atlashub_contracts) is the canonical source for Java↔Python native gRPC tool service schemas, including the initial **internal** `atlashub.intelligence.v1alpha1` draft. Python imports version-pinned generated Protobuf/grpcio stubs **only from infrastructure adapters** and maps into its own Clean Architecture application types. Java remains the authority for user/org scopes, business tools, provider eligibility, idempotency and approvals. Browser voice streams use WebRTC/WebSocket gateway, not browser native bidirectional gRPC. These are contracts, not working RPC endpoints. [Tool gateway draft](https://github.com/OlamidotunIY/atlashub_contracts/blob/main/proto/atlashub/intelligence/v1alpha1/tool_gateway.proto).
