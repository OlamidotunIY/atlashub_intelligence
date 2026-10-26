# Atlas Intelligence — Internal gRPC contract proposal

**Update (2026-10-26):** Canonical initial `v1alpha1` private Java/Python tool RPC schema now exists at [atlashub_contracts: tool_gateway.proto](https://github.com/OlamidotunIY/atlashub_contracts/blob/main/proto/atlashub/intelligence/v1alpha1/tool_gateway.proto). The service is **not implemented**; the operations listed below beyond ListAuthorizedTools/ExecuteAuthorizedTool remain future proposals. Import generated Python protobuf/grpcio packages through infrastructure ports, not the domain. Browser real-time audio still needs a WebRTC/WebSocket gateway. Generated artifacts not published.


**Status:** Draft service design. The first shared .proto draft exists in separate `atlashub_contracts`, but runtime client/server and released generated packages do not.

## Trust boundary
Java API is the identity/authorization authority; Python is a delegated tool planner, not an independent admin. Mutual service authentication and short-lived signed context required. Reject organization/environment IDs supplied only by frontend or model.

## Candidate service operations
- `StartConversation` -> conversation handle, allowed capabilities and server policy version.
- `SubmitTextTurn` -> streamed response events plus evidence/tool statuses.
- `StreamVoiceTurn` -> bidirectional audio frames, transcription revisions, server turns, synthetic audio chunks and interruption signals.
- `GetConversation` / `CloseConversation` -> user-scoped conversation read and closure.
- `ProposeAction` -> typed action proposal and preview, never an execution success.
- `ConfirmAction` -> Java-validated approval token and accepted request ID.
- `CancelTurn` -> best-effort model/audio interruption; persist cancellation state.
- `ExecuteAuthorizedTool` (Python-to-Java) -> known schema, approved principal, trusted command/query result.

## Common envelope fields
`request_id`, `correlation_id`, `conversation_id`, `turn_id`, `tool_name`, `schema_version`, `action_proposal_id`, `idempotency_key`, `result_status`, `evidence_refs`, `redacted_error_code`, `timestamp`.
Identity is validated through server credentials and a delegated, signed token, not raw client fields.

## Streaming and reliability
Explicit sequence numbers, deadlines, max frame sizes, quotas, backpressure, cancellation, retry policy and ordered event kinds. Never retry mutation tools without stable idempotency keys. Unknown outcome must stay `UNKNOWN` until reconciliation. Trace streamed turns without logging raw secrets or audio by default.

## Compatibility
Version protobuf packages and data contracts; use additive fields and reserve removed field numbers. Provide independent Java/Python contract and security tests before activating any operation. Public browser transport may be WebSocket/WebRTC through a controlled gateway rather than native gRPC.
