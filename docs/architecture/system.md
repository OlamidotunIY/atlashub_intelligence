# Atlas Intelligence — System Architecture

**Status:** Proposed; documentation only. **Stack candidate:** Python 3.12+, gRPC/Protocol Buffers, self-hosted open-weight inference, async speech pipeline. The exact web framework, model runner and databases remain selection decisions.

## Mission and ownership
Deliver a conversational business command center in text and live voice. The assistant can explain business information and propose authorized operations, but never bypass Java domain invariants or independently write Atlas Hub commerce, inventory, HR, payments or accounting state.

### Architecture flow
1. Authenticated Atlas Hub client opens a conversation via Java API / trusted streaming gateway.
2. A Java-issued short-lived delegated service authorization binds user, active organization, environment, expiry and allowed scopes.
3. The Python orchestrator interprets a text/voice turn and asks the Java tool gateway for approved schemas/reads.
4. Model-generated action plans are treated as untrusted proposals; Java revalidates authorization, business constraints, idempotency, and approval rules on each execution.
5. Python responds using actual tool results. Numerical business claims include structured evidence references.
6. Tool outcomes, approved actions, model version and response trace are logged without retaining secrets.

### Bounded contexts
- **Conversation:** sessions, turns, transcript state, interruptions, memory policy, redaction.
- **Tool Registry:** discover authorized schemas, normalize arguments, query authoritative Java tools.
- **Action Orchestration:** proposal, risk classification, dry-run, approval, execution and result.
- **Model Operations:** adapters, inference, routing, prompt/model releases, evaluation and fallback.
- **Speech:** real-time capture gateway, ASR, TTS, VAD, barge-in and turn-taking.
- **Speaker Risk:** consent-governed voice templates, comparison and spoof indicators; NOT authentication.
- **Evidence/Audit:** source references, call trace, uncertainty, redaction and retention.

### Clean Architecture and DDD
Domain entities, value objects and policy interfaces are Python standard-library-only when practical. Application use cases depend on ports, not database/transport/ML frameworks. Infrastructure provides gRPC clients, persistence, key management, model runtimes, ASR/TTS implementations and telemetry. Presentation exposes versioned internal gRPC and stream handlers.

CQRS is useful for `StartConversation`, `SubmitTurn`, `ProposeAction`, `ConfirmAction`, `GetConversation`, `GetToolTrace` and evaluation records. Don't create heavyweight command/query classes for individual audio frames.

### Scalability
Streaming audio and generative inference may need different compute pools. Keep request cancellation, tenant quotas, bounded tool calls, rate limits and graceful degradation independent. No assumptions that gRPC or GPU hardware automatically ensures low latency. Measure speech-first-token, inference token latency and end-to-end tool outcomes.

### Java integration: cross-project source alignment
Backend source: https://github.com/OlamidotunIY/atlashub_backend
Backend architecture proposal: https://github.com/OlamidotunIY/atlashub_backend/blob/docs/global-platform-ai-storefront-architecture/docs/architecture/ai-command-center-voice-security-proposal.md

Java command handlers already exist for catalog creation, inventory adjustments, outlet creation and stock transfer; map tools to authenticated handlers, not repository tables. Reference Java DDD and hexagonal boundaries, but make this repository independently deployable.

### Non-negotiable invariants
- Organization context cannot be chosen by model output or arbitrary user prompt.
- Client transcripts, provider responses and retrieved business text are untrusted.
- No secret values in the model prompt context or external logging.
- Sensitive writes need step-up and/or existing maker-checker controls regardless of speaker confidence.
- A tool failure must never be reported as a completed operation.
- Every run is cancellable, observable and budget limited.
