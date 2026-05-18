# AEE — Codebase Map

<!-- Last scan: 2026-05-18 -->
> Regenerated: 2026-05-18 — full mirror scan

**Agent Envelope Exchange (AEE)** — A 14-field JSON envelope format for agent-to-agent communication with explicit causality.

## Protocol Summary

| Property | Value |
|----------|-------|
| **Version** | 1 |
| **License** | MIT |
| **Status** | Experimental — Open for Feedback |
| **IETF Draft** | [`draft-cowles-aee-00`](https://datatracker.ietf.org/doc/draft-cowles-aee/) |

**Core purpose:** Standardize the *envelope*, not the agent. Fixed structure, explicit causality, portable across frameworks (LangGraph, AutoGen, CrewAI, etc.).

**What AEE is NOT:** Not orchestration, not a runtime, not a framework, not a transport — just structure.

## File Structure

```
aee/
├── README.md                         # Protocol overview, quick examples, why AEE exists
├── aee.md                            # Full specification (14 fields, validity rules, JSON Schema)
├── intents.md                        # Intent registry (aee.*, aee.ext.*, ops.*, docs.*)
├── quickstart.md                     # First envelope in 5 minutes (copy-paste ready)
├── relationship-to-mcp-acp.md        # How AEE relates to MCP and ACP, middleware adoption
├── AI_README.json                    # Machine-readable spec (valid AEE envelope itself)
├── CHANGELOG.md                      # Version history
├── LICENSE                           # MIT license
├── CODEBASE_MAP.md                   # This file
├── .gitignore
├── schemas/
│   └── decision-evidence.schema.json # JSON Schema for decision evidence (Section 13)
├── examples/
│   └── handshake.md                  # Capability discovery (aee.capability.list, aee.spec.query)
└── .github/
    ├── CODEOWNERS                    # Default reviewers (@quoxai)
    ├── pull_request_template.md
    └── ISSUE_TEMPLATE/
        ├── bug.md
        └── feature.md
```

## Envelope Fields (14 total)

### Required (10)

| Field | Type | Description |
|-------|------|-------------|
| `v` | string | Protocol version (`"1"`) |
| `id` | string | Unique envelope ID (ULID/UUID) |
| `ts` | string | ISO 8601 timestamp |
| `type` | enum | `task`, `result`, `event`, `error`, `stream` |
| `from` | string | Sender ID (`agent.*`, `human.*`, `service.*`) |
| `to` | string | Recipient ID |
| `intent` | string | Namespaced verb+noun (`ops.backup.status.check`) |
| `corr` | string | Correlation ID (shared across workflow) |
| `priority` | enum | `low`, `normal`, `high`, `urgent` |
| `payload` | object | Intent-specific data |

### Optional (4)

| Field | Type | Description |
|-------|------|-------------|
| `reply_to` | string/null | ID of envelope being replied to (MUST be non-null for `result`/`error`) |
| `trace` | object/null | OpenTelemetry context (`trace_id`, `span_id`) |
| `requires` | object/null | Constraints: `timeout_ms`, `evidence`, `human_approval`, `decision_evidence` |
| `sig` | string/object/null | Cryptographic signature |

## Envelope Types

| Type | Purpose | reply_to |
|------|---------|----------|
| `task` | Request to do work | null |
| `result` | Successful completion | MUST be non-null |
| `event` | Informational signal | null |
| `error` | Failed completion | MUST be non-null |
| `stream` | Partial/progress update | null |

## Reserved Intents (`aee.*`)

Protocol negotiation only — not orchestration.

| Intent | Purpose |
|--------|---------|
| `aee.status.ping` | Liveness check (responds with pong) |
| `aee.status.health` | Health/readiness status |
| `aee.spec.query` | AEE version and capabilities |
| `aee.capability.list` | List supported intents |
| `aee.context.fetch` | Retrieve envelope by reference |
| `aee.context.refute` | Reject context with reason |
| `aee.validate.payload` | Validate payload against intent schema |

## Extension Intents (`aee.ext.*`)

| Intent | Purpose |
|--------|---------|
| `aee.ext.decision_evidence` | Structured decision evidence in result payloads |

## Application Intents (Examples)

| Intent | Purpose |
|--------|---------|
| `ops.network.port.probe` | TCP port reachability check |
| `ops.backup.status.check` | Backup job status |
| `docs.summarize.with_citations` | Document summarization with sources |

## Adoption Tiers

| Tier | Fields | Status |
|------|--------|--------|
| **MVE-Required** | 10 required fields | Schema-valid, full compliance |
| **MVE-5** | `v`, `id`, `type`, `from`, `intent` | Log-friendly only, NOT compliant |

## Decision Evidence Extension

When `requires.decision_evidence` is set, result payloads include structured reasoning:

```json
{
  "decision_evidence": {
    "inputs_used": ["..."],
    "context_refs": ["..."],
    "tools_used": ["..."],
    "decision": "...",
    "reason_summary": "...",
    "action_taken": "...",
    "confidence": 0.92
  }
}
```

Levels: `none`, `minimal`, `standard`, `full`

Schema: `schemas/decision-evidence.schema.json`

## Related Protocols (Quox Family)

| Protocol | Role | Repo |
|----------|------|------|
| **AEE** | Envelope format + causality | *(this repo)* |
| **AOCL** | Orchestration control layers | [AOCL](https://github.com/quoxai/aocl) |
| **VOLT** | Verifiable evidence ledger | [VOLT](https://github.com/quoxai/volt) |
| **WARD** | Content-free hash-chain witnessing | [WARD](https://github.com/quoxai/ward) |

## Key Concepts

1. **`corr` + `reply_to`** — Every envelope in a workflow shares `corr`; responses point back via `reply_to`
2. **Wrap-by-reference** — Use `payload.references[]` with IDs/locators, never embed full envelopes
3. **Human symmetry** — `human.*` identifiers are first-class in `from`/`to`
4. **Middleware wedge** — Gateway-layer injection for zero-code adoption
5. **Type vs Intent** — `type` is envelope category (fixed set); `intent` is payload meaning (your namespace)

## Causality Flow Example

```
human.adam                       # A human starts a task
    │
    │ id: aaa-111
    │ corr: xyz-789
    │ intent: backup.check
    ▼
agent.router                     # Router delegates
    │
    │ id: bbb-222
    │ corr: xyz-789              # same corr
    │ reply_to: aaa-111          # points back
    ▼
agent.worker                     # Worker does the job
    │
    │ id: ccc-333
    │ corr: xyz-789              # same corr
    │ reply_to: bbb-222          # points back
    ▼
result                           # Response bubbles up
    │
    │ id: ddd-444
    │ corr: xyz-789              # same corr
    │ reply_to: ccc-333          # traceable chain
    ▼
human.adam sees the result
```

## Companion Protocols Integration

- **AOCL** emits `aocl.*` intents as AEE envelopes — no AEE changes needed
- **VOLT** records AEE envelope events (`aee.envelope.received`, `aee.envelope.sent`) in tamper-evident traces
- **WARD** witnesses AEE envelopes by ID and payload hash — content-free receipts, failures never affect AEE transport

## JSON Schema

The envelope schema (`aee-v1.schema.json`) is embedded in `aee.md` Section 6. Key validation rules:

- 10 required fields must exist
- `type` must be one of: `task`, `result`, `event`, `error`, `stream`
- `reply_to` MUST be non-null for `result` and `error` types
- Unknown fields MUST be ignored (forward compatibility)

## Entry Points

| Task | File |
|------|------|
| First envelope | `quickstart.md` |
| Full spec | `aee.md` |
| Payload schemas | `intents.md` |
| Machine-readable | `AI_README.json` |
| Capability discovery | `examples/handshake.md` |
| Protocol positioning | `relationship-to-mcp-acp.md` |

## No Implementation Code

This repository is **specification only**:
- Protocol documentation (Markdown)
- JSON Schema fragments
- Example envelopes

No validators, SDKs, or runtime code.
