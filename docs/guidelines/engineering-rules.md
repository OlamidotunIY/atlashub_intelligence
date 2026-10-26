# Atlas Intelligence — Python engineering and safety rules

**Normative target policy.**

## Python engineering
Use supported Python 3.12+ baseline as chosen by team, type hints/strict static checks, Ruff/formatter, pytest, Pydantic **at the boundary** rather than injecting serialization models into pure domain objects, reproducible dependency lockfile and optional dependency groups for inference/ASR/TTS. No unbounded implicit agent loops. Streams implement backpressure, timeout, cancellation and reconnection.

## Trust & execution
- Java/IdP session/role and Organization context are authoritative; Python never generates its own merchant superuser privilege. A logged-in User can lack Organization; allow only globally permitted personal/general AI capabilities in that state.
- Every business tool call has Java-defined schema, signed short-lived delegated identity, current org and allowed scopes, idempotency key/approval status and server-side Java reauthorization.
- High-risk payments, payroll, refunds, supplier bank changes and bulk operations require explicit previews and risk-based step-up/maker-checker; voice does not waive approvals.
- Tool success shown to user **only after Java authoritative confirmation**, unknown/pending statuses remain transparent. Strict rate/amount/cost budgets; no automatic retries of potentially non-idempotent mutations.
- Tool instructions and retrieved business data are untrusted. Defend against prompt injection, cross-tenant document and memory leakage, malicious tool arguments and SSRF.
- Self-hosted open-weight inference is target; document model license, data residency, hardware use, safety benchmark and reproducibility. No assumption that training from scratch is necessary.

## Voice privacy and accessibility
Enroll separate consent-governed voice risk profiles if used. Protect biometrics with encryption/revocation/retention policy, avoid raw audio retention by default, test cloned/replayed audio, accents, colds and changed voice. Voice comparisons and ASR **not standalone authentication**. Offer accessible text/passkey/MFA continuation.

## Business/provider rules
Merchant-owned payment/logistics provider connections managed by Java, not Python. No Atlas Anchor universal bank/KYB gate; provider-specific verification still applies. Never reveal merchant secret credential, claim unsupported processor eligibility or fabricate external balances. Java recalculates all amounts.

## Architecture discipline
Domain and application do not import framework/ML libraries. Ports defined inward with typed failures. Observability uses redacted trace/correlation and evidence refs; do not log passwords, bank tokens, biometrics or full customer private records. PR must include threat analysis for each new tool/speaker feature and tests.

## Memory design
Conversation and user memory bounded by current identity/tenant, consent, retention and data minimization. Tool results cannot silently become permanent global memory. Respect deletion/role revocation and ensure previously cached authorized data cannot escape after permission change.
