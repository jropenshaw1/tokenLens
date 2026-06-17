# ADR-003 — Measurement Model Schema

**Date:** 2026-06-05 | **Status:** Proposed (expanded from stub; awaits review gate)
**Decision owner:** Jonathan (final authority per ADR-007)

> Replaces the ADR-003 stub in `02_ADR_Log_TokenLens_v0.1_DRAFT.md`. Grounded in real vendor payloads (Anthropic Messages API, OpenAI Chat Completions, Perplexity Sonar) and the reference implementation's schema (`nateherkai/token-dashboard`), reviewed 2026-06-05.

## Context

TokenLens captures token telemetry from multiple sources whose payloads disagree on field names, structure, and which metrics they expose. Three observations from inspecting real payloads:

- **Vendor vocabularies conflict, even within a single payload.** Anthropic says `input_tokens`/`output_tokens`; OpenAI Chat Completions says `prompt_tokens`/`completion_tokens`; Perplexity says `prompt_tokens` under `usage` but `input_tokens_cost` under `cost`. OpenAI's own Responses API uses `input_tokens` while its Chat Completions uses `prompt_tokens`. There is no industry-standard field set.
- **Cache is the metric that changes the durability of a token count.** A cache-read token and a cache-creation token are the same unit but price out very differently, and the structure differs by vendor (Anthropic splits creation by TTL; OpenAI reports only reads under `prompt_tokens_details.cached_tokens`; Perplexity omits cache entirely). Cost is reconstructable later *only if* the cache breakdown is captured at the time of the call.
- **Telemetry and judgment are different layers.** The reference implementation captures raw telemetry (tokens, model, project, tools, timing) automatically and captures *no* judgment fields (outcome, value, purpose, work-type). Those belong to a human review overlay, not the auto-collector.

This is a canonical-data-model integration problem — structurally identical to enterprise service bus mediation. Each source is an endpoint with a vendor-shaped input port; the schema is the canonical message; per-source adapters transform square payloads into the round canonical form. Vendor change is isolated to the adapter.

## Decision

TokenLens uses a **canonical token schema** populated by **per-source adapters**, with the **raw payload preserved verbatim**, and **judgment captured as a separate overlay** joined by key. Cost is never stored as fact; it is derived at query time from a versioned rate table.

### Principle: tokens are durable fact, cost is a derived view
Capture the immutable fact (tokens by type, including the cache breakdown). Derive the mutable interpretation (cost) at query time from a rate table. This is the journal-entry / balance-sheet discipline: facts are captured once and never restated; reports are computed and re-computable. Accurate, complete token capture makes cost reconcilable at any future point and at any pricing tier.

### Layer 1 — Canonical telemetry (auto-captured by the collection point)

| Canonical field | Type | Source / derivation | v1 | Notes |
|---|---|---|---|---|
| `event_id` | uuid | generated | ✓ | Primary key |
| `timestamp` | timestamptz | payload / capture time | ✓ | When the call occurred |
| `source_system` | text | adapter | ✓ | anthropic_api \| perplexity_api \| openai_api \| openrouter \| claude_chat \| openbrain_mcp |
| `model` | text | payload | ✓ | e.g. claude-opus-4-8, sonar-pro |
| `skill_name` | text | caller context | ✓ | jfa \| lexi-relay \| quick-fit \| … (vendor-neutral analog of Nate's project_slug) |
| `agent_step` | text | caller context | – | For multi-agent skills; which agent within the skill |
| `session_id` | text | caller context | ✓ | Groups related calls in one work session |
| `input_tokens` | int | adapter | ✓ | Canonical. Maps from `input_tokens` (Anthropic) or `prompt_tokens` (OpenAI/Perplexity) |
| `output_tokens` | int | adapter | ✓ | Canonical. Maps from `output_tokens` or `completion_tokens` |
| `cache_read_tokens` | int | adapter | ✓ | Anthropic `cache_read_input_tokens`; OpenAI `prompt_tokens_details.cached_tokens`; null where unexposed |
| `cache_create_tokens` | int | adapter | ✓ | Anthropic `cache_creation_input_tokens`; TTL split kept in `raw_payload` |
| `reasoning_tokens` | int | adapter | – | OpenAI `completion_tokens_details.reasoning_tokens`; nullable; forward-capture for the field category that keeps growing |
| `total_tokens` | int | **derived** | ✓ | input + output + cache_create (NOT cache_read — see billable vs. volume) |
| `vendor_cost` | numeric | payload if present | – | Perplexity provides it; captured as a cross-check, never the source of truth |
| `fidelity_lane` | text | adapter | ✓ | exact \| measured_activity \| estimate (ADR-002) |
| `raw_payload` | jsonb | verbatim | ✓ | The original usage block, unmodified — the safety net for fields the canonical schema doesn't yet model |

**Billable vs. volume.** `total_tokens` is billable volume and deliberately excludes `cache_read_tokens`, because cache reads price far below fresh input. Total *volume* (including reads) and total *billable* are different numbers; conflating them misstates cost. Both are recoverable from the stored fields; the schema keeps them distinct.

### Layer 2 — Judgment overlay (human-applied, joined by `event_id` or `session_id`)

Captured separately, after the fact, never by the collector: `work_type` (assistant \| computer \| review \| automation \| research \| drafting \| code \| dashboard \| operations), `purpose_category` (job_search \| portfolio_build \| learning \| admin \| personal), `outcome_tag`, `review_burden`, `next_move`, `notes`. These can be added, remapped, or re-scored at any time without touching the telemetry layer — "dress them up any way we want" — precisely because the token facts underneath are stable.

### Per-source adapter (the ESB endpoint)
Each `source_system` has one adapter that maps its payload to the canonical fields above and writes the original to `raw_payload`. Adding a source = adding an adapter. A vendor changing its payload = changing one adapter. The canonical schema and all downstream consumers are insulated from both.

## Alternatives Considered
- **Store each vendor's native field names** — rejected. Pushes normalization into every query and every chart; the square/round mismatch never gets resolved, just deferred to read time, repeatedly.
- **One cache field** — rejected. Read vs. creation price differently and creation has TTL tiers; collapsing them destroys the ability to reconstruct cost. Keep `cache_read` and `cache_create` canonical, TTL detail in `raw_payload`.
- **Store cost as a captured fact** — rejected. Cost is point-in-time and rate-dependent; storing it freezes a number that should be re-derivable. Capture tokens; derive cost from a versioned rate table at query time. (Capture `vendor_cost` only as a reconciliation cross-check.)
- **Single flat table mixing telemetry and judgment** — rejected. Couples machine-collected fact to human-applied interpretation; forces the collector to invent judgment it cannot observe, and blocks re-scoring judgment without disturbing facts.
- **Drop `raw_payload` to save space** — rejected. Supabase bills on size and these payloads are small; the raw column is the migration path when a vendor adds a field (backfill a new canonical column from data already captured) and the audit trail proving a capture was faithful.

## Consequences
- The schema survives vendor change: new fields land in `raw_payload` immediately and graduate to canonical columns on the project's schedule, not the vendor's.
- Cost is reconcilable at any future date and any pricing tier from the stored token facts plus a versioned rate table.
- Judgment fields are deferrable to v2+ and remappable forever, because they never contaminate the telemetry layer.
- v1 wires the OB measured-activity lane against this schema; the exact-lane adapters (Lexi first) populate the same canonical fields in v2.
- A versioned rate table (`pricing` reference data, effective-dated) is a required companion artifact — noted here, specified at build.
