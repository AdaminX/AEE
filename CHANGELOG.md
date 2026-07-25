# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Extracted the inline envelope JSON Schema from spec section 6 into a machine-readable file at `schemas/aee-v1.schema.json` (verbatim). This becomes the canonical machine-readable schema that implementations validate against; it previously existed only inline in `aee.md`.
- Repository housekeeping and GitHub templates

### Errata
- `reply_to` is RECOMMENDED-always. The schema only conditionally requires `reply_to` (non-null for `result`/`error`, per section 3 rule 3 and the `allOf` block in section 6), and the field is SHOULD-null for other types. Producers are encouraged to always emit `reply_to` (using `null` where no causal predecessor exists) so envelopes validate cleanly across both spec-strict and lib-strict consumers. This clarifies, but does not change, the normative rules.
- MVE (Minimal Viable Envelope) adoption tiers: MVE-Required (schema-valid) and MVE-5 (log-friendly)
- Reserved `aee.*` protocol intent namespace with 7 seed intents (ping, health, spec.query, capability.list, context.fetch, context.refute, validate.payload)
- Human↔Agent symmetry: clarified `from`/`to` fields support `human.*` identifiers as first-class entities
- Wrap-by-reference guidance: recommended `payload.references` shape to avoid nested envelope bloat
- Middleware adoption strategy documentation for gateway-layer AEE injection

## [0.1.0] - 2025-12-14

### Added
- Initial AEE specification and documentation
- Core envelope structure definition (14 fields)
- Intent-based messaging framework
- Starter intent registry with example schemas
- JSON Schema for envelope validation
- Documentation on relationship to MCP and ACP
- Example task/result/error message envelopes

[0.1.0]: https://github.com/quoxai/aee/releases/tag/v0.1.0
