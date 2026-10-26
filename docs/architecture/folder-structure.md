# Atlas Intelligence — Python clean and hexagonal source structure

**Planned source blueprint**, not present yet. Standalone Python service, no monorepo.

```text
atlashub_intelligence/
├── docs/
│   ├── architecture/
│   ├── decisions/
│   ├── modules/
│   ├── contracts/
│   ├── security/
│   └── guidelines/
├── src/
│   └── atlashub_intelligence/
│       ├── domain/
│       │   ├── conversations/          # Conversation, Turn, retention, scope
│       │   ├── actions/                # Proposal, approval, risk, outcomes
│       │   ├── tools/                  # typed tool definitions/capabilities
│       │   ├── models/                 # local model version/policy
│       │   ├── speech/                 # voice session/turn semantics
│       │   ├── speaker_risk/           # consent and risk signals, NOT auth
│       │   └── evaluations/            # evidence, metrics and cases
│       ├── application/
│       │   ├── ports/                  # Java tools, inference, ASR/TTS, storage
│       │   ├── commands/               # converse, propose, approve, cancel
│       │   ├── queries/                # session, audit/evidence, available tools
│       │   ├── orchestrators/          # bounded agent workflows
│       │   └── dto/
│       ├── infrastructure/
│       │   ├── java_grpc/             # authenticated, scoped RPC client
│       │   ├── inference/             # self-hosted inference adapters
│       │   ├── asr/                   # streaming speech-to-text
│       │   ├── tts/                   # voice synthesis
│       │   ├── speaker_risk/          # embedding, spoof score and consent
│       │   ├── persistence/           # conversation/memory refs
│       │   ├── credentials/           # KMS/mTLS/service tokens
│       │   ├── streaming/             # VAD, turn-taking, cancellation I/O
│       │   └── telemetry/
│       ├── presentation/
│       │   ├── grpc/                  # private typed services
│       │   ├── streaming/             # audio gateway handler
│       │   └── health/
│       └── bootstrap/                # config, DI, lifecycle
├── proto/
│   └── atlashub/intelligence/v1/
├── tests/
│   ├── unit/
│   ├── contract/
│   ├── integration/
│   ├── adversarial/
│   ├── audio/
│   └── performance/
├── evaluations/
│   ├── datasets/                    # anonymized/licensed, not real PII
│   └── reports/
├── pyproject.toml
└── README.md
```

## Import/dependency rules
- `domain` imports only standard library/pure domain helpers, typed records and policies; **never imports infra**. `application` imports domain and Protocol ports; no FastAPI/SQLAlchemy/vLLM/grpcio or cloud SDK.
- `infrastructure` implements ports for remote Java, inference, storage, ASR/TTS and speaker risk; model adaptations strictly behind interfaces. `presentation` turns protobuf/audio into application DTOs, never calls Java business DB or applies arbitrary state.
- DI/bootstrap wires environment, security and adapters. Source path imports remain local, public protobuf package versioned across Java/Python; no Java/TypeScript source deep imports.

## Deployment/compute
Gateway/stream handlers, conversation state/control and GPU inference worker can be separately deployed **within this repo** when needed; it is not a requirement for independent git repos for every process. CPU voice streaming and GPU model hosting sized independently. No Docker requirement for local user development assumed; document remote test environment separately.
