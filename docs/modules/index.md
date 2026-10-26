# Atlas Intelligence — Module-by-module implementation plan

**Status:** Backlog, not implemented. Each module must have its own design, ports, DTO/contracts, observability and tests before being marked complete.

| Module | Domain responsibility | Initial acceptance criteria |
|---|---|---|
| `conversations` | Tenant-bound conversations, turns, retention and redaction | Isolation, resume, cancellation, deletion and expiry tests |
| `tool-registry` | Versioned tool catalog, JSON/protobuf schemas, eligibility | Schema validation, no arbitrary tool execution, permission filtering |
| `tool-execution` | Java tool gateway orchestration, deadlines, idempotent calls | RBAC revalidated by Java per tool call; no direct business DB writes |
| `action-approvals` | Proposals, previews, risk tier, step-up and maker-checker | No sensitive action without approval and authoritative result |
| `model-runtime` | Self-hosted inference adapters, model and prompt releases | Repeatable evaluation, version pinning, fallback and resource budgets |
| `evidence` | Structured tool evidence and grounded business responses | Every financial claim attributable to authorized records |
| `speech` | Streaming ASR, voice synthesis, interruption and turn-taking | Test latency, accessibility, multi-speaker handling and text fallback |
| `speaker-risk` | Optional enrollment, similarity, spoof signal and recovery | Explicit consent, safe deletion, false reject/accept tests, not auth |
| `security-audit` | Tenant-safe traces, privacy, risk events and policy enforcement | No secrets logged; alerts and retention enforcement |
| `evaluation` | Offline/online quality evaluation, regression and drift | Benchmarks for hallucination, tool accuracy, action safety and latency |

## Suggested development sequences

### Slice 1 — text read-only
1. Define protobuf and Java delegated-service identity protocol.
2. Implement conversation persistence (org+environment+user scoped) and text turns.
3. Add read tools for catalog, inventory, outlet and authorized analytics.
4. Add evidence grounding, tool audit and refusal/clarification for ungrounded questions.
5. Test cross-tenant denial, prompt injection resistance and timeouts.

### Slice 2 — state-changing AI commands
1. Define typed commands and action proposals; preview consequences.
2. Integrate `CreateProductHandler`, `AdjustStockHandler`, `RequestStockTransferHandler`, `CreateOutletHandler` through Java-approved tool interfaces.
3. Require confirmation/risk-based step-up; preserve maker-checker.
4. Create idempotency keys and result status machine `PROPOSED -> AWAITING_APPROVAL -> SUBMITTED -> COMPLETED/FAILED`.
5. Verify user-visible success reflects persisted Java command outcome.

### Slice 3 — voice and independent models
1. Benchmark multiple self-hostable language models on Atlas Hub command accuracy.
2. Add speech recognition, speech synthesis and synchronized text transcripts.
3. Support real-time audio interruption and cancellation.
4. Add optional biometric consent/enrollment and spoof *risk* assessment after privacy/security review.
5. Verify all voice users can continue through MFA/passkey or accessible text input if voice changes.

## Tests required
Contract tests for protobuf backward compatibility; isolated domain tests; integration tests with Java fake tool server; adversarial instructions in business data; authorization/approval, replay and duplicate actions; audio noise/accent/replay/clone evaluation; PII retention/deletion; load/latency tests.

## Deferred
Foundational language model training from scratch, autonomous approval of high-risk actions, voice as sole authenticator, unrestricted model database access, cross-organization training without explicit governance.


## Architecture requirements

Each bounded context must conform to [DDD/Clean/Hexagonal design and Java tool authority](../architecture/clean-hexagonal.md), [Python package boundaries](../architecture/folder-structure.md), [security/model/voice engineering policy](../guidelines/engineering-rules.md), and [test/release/evaluation gates](../guidelines/testing-release.md). No Python service can bypass the Java identity, organization, approval or merchant-provider capability boundaries.
