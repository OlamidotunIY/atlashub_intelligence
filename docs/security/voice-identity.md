# Voice identity and speaker-risk design

**Status:** Proposed, not implemented. Voice is an **optional convenience and risk signal**, NOT proof of authentication.

## Distinct pipelines
- ASR (speech-to-text) recognizes **what** was said.
- Speaker verification compares **who appears to speak** with an enrolled profile.
- Liveness / anti-spoof detection attempts to identify replayed or synthetic audio.
- Diarization can identify changing speakers during a conversation.
Each is imperfect. A familiar voice can be replayed or cloned.

## Identity authority
Only the Atlas Hub authenticated session and required passkey/MFA/step-up authorization grant rights to use business tools. Speaker similarity cannot grant, recover or replace account access. NIST SP 800-63B-4 states voice biometric comparison SHALL NOT be used for authentication in systems claiming that standard's conformance. Respect applicable local biometric/privacy laws and consent rules. Reference: https://pages.nist.gov/800-63-4/sp800-63b.html

## Optional enrollment
1. Require current authenticated session and strong step-up.
2. Explain biometric processing purpose, retention, possible false matches/rejections; collect explicit revocable consent.
3. Capture several varied prompted samples with audio quality and environmental tests, without unnecessarily saving raw voice recordings.
4. Generate a versioned voice embedding and encrypted minimal metadata, isolated by user identity, organization scope and environment where applicable.
5. Store ciphertext under managed keys, log enrollment metadata without raw audio, allow opt-out, deletion and enrollment replacement.
6. Re-enrollment requires strong step-up; do not silently update the profile from unverified audio.

## Runtime decisioning
- **Normal risk:** permit conversation, with Java RBAC/action policy checks.
- **Uncertain:** pause voice-only action and offer alternative authenticated UI/passkey/MFA; never automatically ban a user.
- **Suspected spoofing:** block the specific voice action and require secure reauthentication; rate-limit and audit.
Never use a fixed invented universal similarity threshold. Select operating points after measuring population- and environment-specific false accept/false reject behavior.

## Changed voice and accessibility
Evaluate colds, throat illness, age, stress, whispering, accents, languages, microphones, network compression, background noise and disabilities. Give a non-biometric fallback even when enrollment or matching fails. Recovery must not be less secure than standard authentication.

## Sensitive commands
Payouts, supplier bank-detail changes, payroll disbursement, destructive bulk changes, refunds, role permissions, mass inventory transfers and financial journal posting require explicit policy-defined confirmation and potentially maker-checker; voice evidence never waives these.

## Threat model and privacy checklist
Threats: replay, cloned audio, injected browser audio, account takeover, session hijack, profile template theft, cross-tenant profile leakage, spoofed gRPC client, prompt injection, data retention misuse.
Controls: signed short-lived delegated identity, TLS/mTLS, risk-evaluated channels, key rotation, encryption, strict data access, bounded retention, consent revocation, incident monitoring, audit and deletion tests.

## Evaluation gates
Measure speaker detection error tradeoffs, spoof false positives and negatives, genuine-user rejection across voice changes, usability under noise, audio injection attack resistance and impact on assistive access. No claims of identity certainty without representative validation.
