# Atlas Intelligence — DDD, Clean Architecture and Hexagonal/Ports-and-Adapters

**Status:** Normative design for a separate Python AI/voice service. No inference/agent application has been implemented.

## Guiding principle
The Python service understands business language and proposes operations; Java **remains authoritative for identity, organization/IAM, provider integrations, catalog/stock/orders/HR/accounting and approved command execution**. A model's text is not a trusted user, tool approval or database mutation.

## Bounded contexts
- **Conversations:** Conversation, Turn, VoiceSession, ConversationRetentionPolicy, ContextWindow and memory scope.
- **ToolCatalog:** authorized Java-defined ToolSchema, ToolCapability, TenantScope, ToolVersion and tool discovery.
- **ActionOrchestration:** ActionProposal, Evidence, RiskTier, ActionApproval, ExecutionOutcome, PendingAction; no direct business DB writes.
- **ModelRuntime:** ModelRelease, PromptVersion, InferencePolicy, ProviderHealth/LocalModelAdapter, evaluation; self-hostable model inference.
- **Speech:** AudioChunk, TranscriptRevision, SpeakerTurn, VoiceResponse, interruptions/ASR/TTS pipelines.
- **SpeakerRisk:** enrollment consent/reference, comparison and spoof-risk signal; voice alone does **not** authenticate.
- **Audit/Evaluations:** tool trace, redacted event, evaluation case, safety check.

## Dependency direction
Domain contains pure Python value objects/entities/policies and ports as Protocols/ABCs: no FastAPI/gRPC, OpenAI/vLLM/ASR/TTS package, Redis/Postgres, browser SDK or Java DB imports. Application use cases orchestrate domain decisions and call outbound interfaces; infrastructure implements gRPC clients to Java, inference engine, ASR/TTS, persistence, audio transport, cryptography and telemetry; presentation translates typed gRPC/stream requests. Composition root builds providers, validates env and selects adapters.

Examples of ports: `JavaToolRegistryPort`, `JavaAuthorizedToolExecutionPort`, `AuthContextVerifierPort`, `ConversationRepositoryPort`, `ModelInferencePort`, `SpeechRecognizerPort`, `SpeechSynthesizerPort`, `SpeakerRiskPort`, `ApprovalPort`, `Clock`, `AuditSink`. Test with fake adapters, do not let model package return direct ORM objects.

## Identity, tenant and provider eligibility
Individual User registration is separate from Organization creation through Java/IdP and Auth Web. A User may legitimately have **no active organization**, permitting normal non-business AI conversation where allowed; all business tools require explicit currently authenticated Java organization membership/permissions. Never infer org from conversation or let model choose arbitrary `organizationId`.

Atlas Hub is retiring mandatory Atlas-operated Anchor banking and five-step KYB; merchant payment/shipping integrations are optional and provider/capability-specific. AI tool availability must come from Java authoritative capability checks; no claim that external balances belong to Atlas, no payment initiated without an eligible connection and transaction approval, never pretend no payout provider means success.

## Tool execution trust
User voice/text, retrieved documents, websites and tool outputs are **untrusted content**, not security principals. Java validates every mutation against active session, RBAC, bounded tool schema, limits, idempotency and maker-checker/step-up for high-risk tasks. Python cannot bypass via service identity or shared DB credentials. Use delegated short-lived authenticated identity, private mTLS gRPC, typed errors and audit.

## Voice and biometric policy
ASR determines words; speaker comparison supplies **only risk signal**, anti-spoofing is imperfect; high confidence cannot replace central session/passkey/MFA. Cold/voice change must offer non-biometric fallback. Enrollment explicit consent, encrypted embeddings and safe revocation. See [voice security](../security/voice-identity.md).

## CQRS pragmatically
Command/use-case abstractions for `StartConversation`, `SubmitTurn`, `ProposeAction`, `ConfirmAction`, `CancelTurn`, `DeleteConversation`; query paths for conversation/evidence/tool registry; do **not** create a command bus for every audio frame, token or ASR partial transcript. Stream latency and cancellation matter.

## Domain vs ML runtime
Domain policies decide approvals, bounded memory, tool visibility, transition invariants; ML infrastructure estimates probabilities/generates content. Model inference cannot be treated as domain policy authority. Ground business output in Java tool evidence.

See [folder structure](folder-structure.md), [engineering rules](../guidelines/engineering-rules.md), [tests](../guidelines/testing-release.md).
