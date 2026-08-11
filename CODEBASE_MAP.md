<!-- Regenerated: 2026-08-11 by codebase-mirror scan (verified) -->

# AEE — Codebase Map

**Agent Envelope Exchange (AEE)** — A 14-field JSON envelope format for agent-to-agent communication with explicit causality. Standardizes the *envelope*, not the agent. Part of the Quox protocol family (AEE, AOCL, VOLT, WARD). Specification only, no runtime/validator code.

## Metrics
| Metric | Value |
|--------|-------|
| Protocol Version | **1** |
| Schema files | 2 |
| Example files | 1 |
| Spec docs (Markdown) | 8 |
| IETF Draft | `draft-cowles-aee-00` |
| License | MIT |
| Status | Experimental — Open for Feedback |

## Directory Structure

```
aee/
├── README.md                  # Overview, examples, quick reference (334 lines)
├── aee.md                     # Full specification (489 lines)
├── intents.md                 # Intent registry + payload schemas (501 lines)
├── quickstart.md              # 5-minute getting started guide (115 lines)
├── relationship-to-mcp-acp.md # Protocol layer comparison (82 lines)
├── AI_README.json             # Self-describing AEE envelope for AI agents (220 lines)
├── CHANGELOG.md               # Version history (36 lines)
├── LICENSE                    # MIT license
├── CODEBASE_MAP.md            # This file
├── examples/
│   └── handshake.md           # Capability discovery examples (132 lines)
├── schemas/
│   ├── aee-v1.schema.json            # Canonical envelope JSON Schema (37 lines)
│   └── decision-evidence.schema.json # Decision evidence fragment schema (43 lines)
└── .github/
    ├── CODEOWNERS             # @quoxai
    ├── pull_request_template.md
    └── ISSUE_TEMPLATE/
        ├── feature.md
        └── bug.md
```

## Authoritative Files
| File | Purpose |
|------|---------|
| `aee.md` | Full normative spec (14 fields, validity rules, JSON Schema inline in §6, decision evidence in §13) |
| `schemas/aee-v1.schema.json` | Canonical machine-readable envelope schema (extracted from §6) |
| `intents.md` | Intent registry (`aee.*`, `aee.ext.*`, `ops.*`, `docs.*`) |
| `AI_README.json` | Machine-readable spec (itself a valid AEE envelope) |
| `schemas/decision-evidence.schema.json` | JSON Schema for decision evidence (§13) |
| `examples/handshake.md` | Capability discovery example (`aee.capability.list`, `aee.spec.query`) |
| `quickstart.md` | Emit your first envelope in 5 minutes |
| `relationship-to-mcp-acp.md` | Protocol layer comparison (MCP, ACP, AEE) + middleware adoption strategy |

## Envelope Fields (14)

| # | Field | Req | Type | Description |
|---|-------|-----|------|-------------|
| 1 | `v` | ✓ | string | Protocol version (`"1"`) |
| 2 | `id` | ✓ | string | Unique envelope ID (ULID/UUID) |
| 3 | `ts` | ✓ | string | ISO 8601 timestamp |
| 4 | `type` | ✓ | enum | `task \| result \| event \| error \| stream` |
| 5 | `from` | ✓ | string | Sender ID (`agent.*`, `human.*`, `service.*`) |
| 6 | `to` | ✓ | string | Recipient ID |
| 7 | `intent` | ✓ | string | Namespaced action (`ops.backup.status.check`) |
| 8 | `corr` | ✓ | string | Correlation ID (shared across workflow) |
| 9 | `priority` | ✓ | enum | `low \| normal \| high \| urgent` |
| 10 | `payload` | ✓ | object | Intent-specific data |
| 11 | `reply_to` | — | string\|null | ID of envelope being replied to |
| 12 | `trace` | — | object\|null | OpenTelemetry context |
| 13 | `requires` | — | object\|null | Execution constraints |
| 14 | `sig` | — | object\|string\|null | Cryptographic signature |

**10 required fields, 4 optional.** `result`/`error` types MUST have non-null `reply_to` (enforced via `allOf` conditional in schema).

## Intent Namespaces
| Namespace | Purpose |
|-----------|---------|
| `aee.*` | Reserved protocol negotiation (7 seed intents) |
| `aee.ext.*` | Extensions (e.g. `aee.ext.decision_evidence`) |
| `ops.*`, `docs.*` | Application intents (3 worked examples) |

### Reserved Intents (`aee.*`)
| Intent | Purpose |
|--------|---------|
| `aee.status.ping` | Liveness check |
| `aee.status.health` | Health/readiness status |
| `aee.spec.query` | AEE version + capabilities |
| `aee.capability.list` | List supported intents |
| `aee.context.fetch` | Retrieve envelope by reference |
| `aee.context.refute` | Reject referenced context |
| `aee.validate.payload` | Validate payload against intent schema |

### Application Intents (Examples)
| Intent | Purpose |
|--------|---------|
| `ops.network.port.probe` | TCP port reachability check |
| `ops.backup.status.check` | Backup job status |
| `docs.summarize.with_citations` | Document summarization with sources |

## Adoption Tiers
| Tier | Fields | Use Case |
|------|--------|----------|
| **MVE-Required** | All 10 required | Full AEE compliance, agent-to-agent |
| **MVE-5** | `v`, `id`, `type`, `from`, `intent` | Logging/telemetry only (NOT compliant) |

## Schemas

### Envelope Schema (`schemas/aee-v1.schema.json`)
Canonical JSON Schema (draft 2020-12) for envelope validation. Validates the 10 required fields plus 4 optional. Includes conditional logic: `result`/`error` types require non-null `reply_to`.

### Decision Evidence Schema (`schemas/decision-evidence.schema.json`)
Reusable schema fragment for structured decision evidence in result payloads (§13):
- **Levels (per `requires.decision_evidence`):** `none` | `minimal` | `standard` | `full`
- **Fields:** `inputs_used`, `context_refs`, `tools_used`, `decision`, `reason_summary`, `action_taken`, `confidence`

## Causality Model

```
human.adam                       # Human initiates
    │ id: aaa-111
    │ corr: xyz-789              # Correlation ID (shared)
    ▼
agent.router                     # Router delegates
    │ id: bbb-222
    │ corr: xyz-789              # Same corr
    │ reply_to: aaa-111          # Points to predecessor
    ▼
agent.worker                     # Worker executes
    │ id: ccc-333
    │ corr: xyz-789              # Same corr
    │ reply_to: bbb-222          # Points to predecessor
    ▼
result                           # Response bubbles up
    │ id: ddd-444
    │ corr: xyz-789              # Same corr
    │ reply_to: ccc-333          # Traceable chain
```

Every hop shares `corr`. Every response carries `reply_to`. The chain is traceable from human to final result.

## Related Protocols (Quox Family)
| Protocol | Role | Repo |
|----------|------|------|
| **AEE** | Envelope format + causality | *(this repo)* |
| **AOCL** | Orchestration control layers | [aocl](https://github.com/quoxai/aocl) |
| **VOLT** | Verifiable evidence ledger | [volt](https://github.com/quoxai/volt) |
| **WARD** | Content-free hash-chain witnessing | [ward](https://github.com/quoxai/ward) |

**Integration:** AOCL emits `aocl.*` as AEE envelopes; VOLT records AEE envelope events (`aee.envelope.received`, `aee.envelope.sent`); WARD witnesses AEE envelopes by ID + payload hash (content-free receipts).

## Reference SDK

**`@quox/aee-sdk`** (TypeScript/Node) and **`quox-aee`** (Python 3.10+) at **github.com/quoxai/aee-sdk**:
- Zero-dependency envelope create/validate
- Decision-evidence capture (spec §13.2)
- Local sinks and `aee-validate` CLI
- Both packages validate against `schemas/aee-v1.schema.json`

SDK repo is private today, opening at launch with npm/PyPI publishes.

## Invariants
| Check | Status |
|-------|--------|
| Version `v` = `"1"` across spec, schemas, AI_README.json | ✓ pass |
| Schema validity (`aee-v1.schema.json`, `decision-evidence.schema.json`) | ✓ pass |
| Envelope field count (10 + 4 = 14) | ✓ pass |
| Examples present (`examples/handshake.md`) | ✓ pass |
| IETF draft reference consistent | ✓ pass |

## No Code — Spec Only

This repo contains **no executable code**: no package.json, no entry points, no API routes, no test suite. It is a protocol specification with:
- Markdown documentation
- JSON schemas
- Example envelopes

Implementations live in consuming repos (quox-dashboard, QuoxBastion, quoxflow, aee-sdk, etc.).
