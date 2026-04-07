<!-- Last verified: 2026-04-07 by /codebase-mirror -->

# AEE (Agent Envelope Exchange) — Codebase Map

## Overview
Minimal envelope format for agent-to-agent communication with human-in-the-loop. Fixed structure, explicit causality, flat, no hidden state.

| Field | Value |
|-------|-------|
| Version | 1 |
| Status | Experimental |
| IETF Draft | draft-cowles-aee-00 |
| License | MIT |

## Metrics
| Metric | Count |
|--------|-------|
| Schema Files | 1 (decision-evidence.schema.json) |
| Envelope Fields | 14 |
| Envelope Types | 5 |
| Reserved Intents | 7 (aee.*) |

## Invariants
| Check | Status | Details |
|-------|--------|---------|
| schema-valid | ✓ pass | JSON schema valid |
| envelope-types | ✓ pass | 5 types (task, result, event, error, stream) |
| reserved-intents | ✓ pass | 7 aee.* intents |
