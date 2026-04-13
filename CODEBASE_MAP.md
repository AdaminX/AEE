<!-- Last scan: 2026-04-13T00:30:00Z by codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Overview

Minimal envelope format for agent-to-agent communication with human-in-the-loop. Fixed structure, explicit causality, flat, no hidden state.

| Field | Value |
|-------|-------|
| Version | 1 |
| Status | Experimental |
| IETF Draft | draft-cowles-aee-00 |
| License | MIT |
| Repo | github.com/quoxai/aee |

## Metrics

| Metric | Count |
|--------|-------|
| Schema Files | 1 |
| Envelope Fields | 14 (10 required, 4 optional) |
| Envelope Types | 5 |
| Reserved Intents | 7 (aee.*) |
| Extension Intents | 1 (aee.ext.*) |
| Application Intents | 3 (example) |
| Adoption Tiers | 2 |

## File Structure

```
aee/
├── README.md                           # Main documentation + protocol overview
├── aee.md                              # Full v1 specification (14 fields, validity rules, JSON schema)
├── intents.md                          # Intent registry (aee.*, aee.ext.*, application intents)
├── quickstart.md                       # 5-minute intro, copy-paste examples
├── relationship-to-mcp-acp.md          # Protocol positioning vs MCP, ACP
├── AI_README.json                      # Self-describing AEE envelope for AI consumption
├── CHANGELOG.md                        # Version history
├── LICENSE                             # MIT license
├── schemas/
│   └── decision-evidence.schema.json   # Reusable schema for decision evidence blocks
├── examples/
│   └── handshake.md                    # Capability discovery examples
└── .github/
    ├── CODEOWNERS                      # Code ownership
    ├── pull_request_template.md        # PR template
    └── ISSUE_TEMPLATE/
        ├── bug.md                      # Bug report template
        └── feature.md                  # Feature request template
```

## Envelope Structure (14 Fields)

### Required Fields (10)

| Field | Type | Description |
|-------|------|-------------|
| `v` | string | Protocol version (`"1"`) |
| `id` | string | Unique envelope ID (ULID/UUID) |
| `ts` | string | ISO 8601 UTC timestamp |
| `type` | enum | `task` \| `result` \| `event` \| `error` \| `stream` |
| `from` | string | Sender ID (`agent.*`, `human.*`, `service.*`) |
| `to` | string | Recipient ID or channel |
| `intent` | string | Namespaced action (`ops.backup.check`, `aee.status.ping`) |
| `corr` | string | Correlation ID (shared across workflow) |
| `priority` | enum | `low` \| `normal` \| `high` \| `urgent` |
| `payload` | object | Intent-specific data |

### Optional Fields (4)

| Field | Type | Description |
|-------|------|-------------|
| `reply_to` | string\|null | ID of envelope being replied to (MUST be non-null for `result`/`error`) |
| `trace` | object\|null | OpenTelemetry context (`trace_id`, `span_id`) |
| `requires` | object\|null | Constraints (`timeout_ms`, `human_approval`, `evidence`, `decision_evidence`) |
| `sig` | object\|string\|null | Cryptographic signature |

## Envelope Types

| Type | Purpose | Response |
|------|---------|----------|
| `task` | Request to do work | Expects `result` or `error` |
| `result` | Successful completion | `reply_to` MUST be set |
| `error` | Failed completion | `reply_to` MUST be set |
| `event` | Informational signal | No response expected |
| `stream` | Progress/partial update | For long-running processes |

## Reserved Intents (aee.*)

| Intent | Purpose |
|--------|---------|
| `aee.status.ping` | Liveness check (returns pong) |
| `aee.status.health` | Health/readiness status |
| `aee.spec.query` | AEE version and capabilities |
| `aee.capability.list` | List supported intents |
| `aee.context.fetch` | Retrieve envelope by reference |
| `aee.context.refute` | Reject a context reference |
| `aee.validate.payload` | Validate payload against schema |

## Extension Intents (aee.ext.*)

| Intent | Purpose |
|--------|---------|
| `aee.ext.decision_evidence` | Structured decision evidence capture (levels: none, minimal, standard, full) |

## Application Intents (Examples)

| Intent | Purpose |
|--------|---------|
| `ops.network.port.probe` | TCP port reachability check |
| `ops.backup.status.check` | Backup job status |
| `docs.summarize.with_citations` | Document summarization |

## Adoption Tiers

| Tier | Fields | Status |
|------|--------|--------|
| **MVE-Required** | 10 required fields | Schema-valid, full compliance |
| **MVE-5** | `v`, `id`, `type`, `from`, `intent` | Log-friendly, NOT schema-valid |

## Decision Evidence

Structured auditability for AI decisions. Requested via `requires.decision_evidence`:

| Level | Includes |
|-------|----------|
| `none` | No evidence (default) |
| `minimal` | `decision`, `reason_summary` |
| `standard` | + `inputs_used`, `tools_used`, `action_taken`, `confidence` |
| `full` | + `context_refs` with locators |

Schema: `schemas/decision-evidence.schema.json`

## Related Protocols

| Protocol | Role | Repo |
|----------|------|------|
| **AEE** | Envelope format + causality | *(this repo)* |
| **AOCL** | Orchestration control layers | github.com/quoxai/aocl |
| **VOLT** | Verifiable evidence ledger | github.com/quoxai/volt |
| **WARD** | Hash-chain witnessing | github.com/quoxai/ward |

## Integration Points

- **MCP**: AEE wraps MCP tool results; MCP handles capability exposure
- **ACP**: AEE envelopes can be carried as ACP payloads
- **Middleware**: Gateway-layer auto-wrapping for observability without code changes

## Key Design Principles

1. **Fixed structure** → No negotiation, just parse
2. **Explicit causality** → `corr` + `reply_to` link every hop
3. **No nesting** → Flat is fast; use references not embeddings
4. **No hidden state** → Everything survives the wire
5. **Human-first** → `human.*` is a first-class entity type

## Entity Identifier Prefixes

| Prefix | Description |
|--------|-------------|
| `agent.*` | Autonomous agents |
| `human.*` | Human participants |
| `service.*` | Traditional services |
| `bus.*` | Broadcast channels |

## Invariants

| Check | Status | Details |
|-------|--------|---------|
| schema-valid | ✓ pass | JSON schema in aee.md validates envelope structure |
| envelope-types | ✓ pass | 5 types: task, result, event, error, stream |
| reserved-intents | ✓ pass | 7 aee.* intents documented |
| extension-intents | ✓ pass | 1 aee.ext.* intent documented |
| decision-evidence-schema | ✓ pass | Reusable fragment in schemas/ |
