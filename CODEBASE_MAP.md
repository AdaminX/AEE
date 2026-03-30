<!-- Last verified: 2026-03-30 by /codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Repository Summary
Protocol specification for agent-to-agent communication. AEE standardizes a 14-field JSON envelope for portable, observable messaging across frameworks.

## Metrics
| Metric | Count |
|--------|-------|
| Envelope Fields | 14 (10 required, 4 optional) |
| Envelope Types | 5 (task, result, event, error, stream) |
| Reserved Intents | 7 (aee.*) |
| Extension Intents | 1 (aee.ext.decision_evidence) |
| Application Intents | 3 (example set) |
| Schema Files | 1 |
| Documentation Files | 9 |

## Spec Status
- **Version:** 1 (experimental)
- **IETF Draft:** draft-cowles-aee-00
- **License:** MIT

## File Structure
```
aee/
├── aee.md                           # Full spec (13 sections)
├── intents.md                       # Intent definitions + schemas
├── README.md                        # Overview + quick examples
├── AI_README.json                   # Machine-readable spec (valid AEE envelope)
├── quickstart.md                    # 5-minute onboarding
├── relationship-to-mcp-acp.md       # Protocol comparison
├── CHANGELOG.md                     # Version history
├── LICENSE                          # MIT license
├── schemas/
│   └── decision-evidence.schema.json  # Decision evidence fragment
├── examples/
│   └── handshake.md                 # Capability discovery examples
└── .github/
    ├── CODEOWNERS
    ├── pull_request_template.md
    └── ISSUE_TEMPLATE/
        ├── feature.md
        └── bug.md
```

## Authoritative Files
| File | Purpose | Size |
|------|---------|------|
| `aee.md` | Full specification (13 sections) | 18KB |
| `intents.md` | Intent definitions + schemas | 11KB |
| `README.md` | Overview, flow diagrams, quick examples | 11KB |
| `AI_README.json` | Machine-readable spec for AI agents (valid AEE envelope) | 10KB |
| `schemas/decision-evidence.schema.json` | Decision evidence fragment schema | 1.4KB |
| `quickstart.md` | Quick start guide (5 min) | 2.7KB |
| `relationship-to-mcp-acp.md` | Protocol comparison (MCP/ACP/AEE) | 2.8KB |
| `examples/handshake.md` | Capability discovery examples | 3.1KB |
| `CHANGELOG.md` | Version history | 1.3KB |

## Envelope Structure

### Required Fields (10)
| Field | Type | Description |
|-------|------|-------------|
| `v` | string | Envelope schema version (always "1") |
| `id` | string | Unique message ID (ULID/UUID recommended) |
| `ts` | string | ISO 8601 UTC timestamp |
| `type` | enum | `task \| result \| event \| error \| stream` |
| `from` | string | Sender ID (agent.*, human.*, service.*) |
| `to` | string | Recipient ID or channel (bus.*) |
| `intent` | string | Namespaced verb+noun (domain.subdomain.noun.verb) |
| `corr` | string | Correlation ID for workflow threading |
| `priority` | enum | `low \| normal \| high \| urgent` |
| `payload` | object | Intent-specific data |

### Optional Fields (4)
| Field | Type | Description |
|-------|------|-------------|
| `reply_to` | string/null | ID of message being replied to (MUST be non-null for result/error) |
| `trace` | object/null | OpenTelemetry hooks {trace_id, span_id} |
| `requires` | object/null | Constraints (timeout_ms, human_approval, evidence, decision_evidence) |
| `sig` | object/string/null | Cryptographic signature/auth proof |

## Reserved Intents (aee.*)
| Intent | Purpose |
|--------|---------|
| `aee.status.ping` | Liveness check (pong response) |
| `aee.status.health` | Health/readiness status |
| `aee.spec.query` | Query AEE version + capabilities |
| `aee.capability.list` | List supported intents |
| `aee.context.fetch` | Retrieve envelope by reference |
| `aee.context.refute` | Reject referenced context |
| `aee.validate.payload` | Validate payload against intent schema |

## Extension Namespace (aee.ext.*)
| Intent | Purpose |
|--------|---------|
| `aee.ext.decision_evidence` | Structured decision evidence in result payloads |

### Decision Evidence Levels
| Level | Fields Included |
|-------|----------------|
| `none` | No evidence (default) |
| `minimal` | decision, reason_summary |
| `standard` | All except context_refs |
| `full` | All fields including context_refs |

### Decision Evidence Schema Fields
| Field | Type | Description |
|-------|------|-------------|
| `inputs_used` | string[] | Inputs the agent considered |
| `context_refs` | string[] | Locators/IDs of referenced context (full level only) |
| `tools_used` | string[] | Tool names invoked |
| `decision` | string | Concise decision statement |
| `reason_summary` | string | Why (1-3 sentences) |
| `action_taken` | string | What the agent did |
| `confidence` | number (0-1) | Self-assessed confidence |

## Application Intents (Starter Set)
| Intent | Purpose |
|--------|---------|
| `ops.network.port.probe` | Probe TCP port reachability |
| `ops.backup.status.check` | Check recent backup job status |
| `docs.summarize.with_citations` | Summarize document with sources |

## Adoption Tiers
| Tier | Fields | Compliant | Use Case |
|------|--------|-----------|----------|
| MVE-Required | 10 required | ✓ Yes | Full agent-to-agent communication |
| MVE-5 | 5 (v, id, type, from, intent) | ✗ No | Logging/observability only |

## Related Protocols (Quox Family)
| Protocol | Role | Integration |
|----------|------|-------------|
| AOCL | Orchestration control layers | Emits aocl.* intents as AEE envelopes |
| VOLT | Tamper-evident evidence ledger | Uses corr as correlation_id, records envelope events |
| WARD | Content-free hash-chain witnessing | Witnesses envelopes by ID + payload hash |

## Causality Model
- **corr** — Shared across all envelopes in a workflow
- **reply_to** — Points to immediate predecessor (null for initiators)
- Chain: `human.* → agent.router → agent.worker → result` (same corr, reply_to bubbles back)

## Key Design Principles
- **Envelope only** — AEE standardizes structure, not semantics (intent schemas define meaning)
- **No nesting** — Use references (`payload.references`) instead of embedded envelopes
- **Human-first** — Humans are first-class entities via `human.*` namespace
- **Transport-agnostic** — Works over HTTP, WebSocket, NATS, Kafka, etc.
- **Framework-agnostic** — Works with LangGraph, AutoGen, CrewAI, or custom agents

## Invariants
| Check | Status | Details |
|-------|--------|---------|
| schema-valid | ✓ pass | decision-evidence.schema.json valid |
| envelope-types | ✓ pass | 5 types documented (task, result, event, error, stream) |
| reserved-intents | ✓ pass | 7 aee.* intents documented with schemas |
| extension-intents | ✓ pass | 1 aee.ext.* intent documented |
| application-intents | ✓ pass | 3 example intents with full schemas |
| adoption-tiers | ✓ pass | MVE-Required and MVE-5 defined |
| related-protocols | ✓ pass | AOCL, VOLT, WARD integration documented |
