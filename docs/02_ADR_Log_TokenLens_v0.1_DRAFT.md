# TokenLens — Architecture Decision Records

**Project:** TokenLens
**Owner:** Jonathan Openshaw
**Maintained by:** Claude (Project Lead)
**Date:** 2026-06-05
**Status:** DRAFT — not yet through the review gate (Claude + two distinct-model agents, default Gee + Mini; Jonathan final authority)

> **Public-repo boundary note.** TokenLens lives in a public repo. Before this log is committed, the same sanitization items flagged on the PRD apply: generalize or consciously surface (a) references to the private `jonathan-ops` governance repo, (b) agent nicknames if the roster is not yet public, (c) the "job search phase" status, and (d) personal OB usage specifics. No live secrets, OB instance URLs, or credential paths appear in this document.

---

## ADR Index

| ADR | Title | Status |
|---|---|---|
| ADR-001 | Data Architecture: Relational Token Ledger + Collection-Point Model | **Proposed** |
| ADR-002 | Fidelity Lane Separation | **Proposed** |
| ADR-003 | Measurement Model Schema | Proposed (stub) |
| ADR-004 | Visualization Layer & Brand Palette | Proposed (stub) |
| ADR-005 | Public/Private Boundary for Portfolio Use | Proposed (stub) |

---

## ADR-001: Data Architecture — Relational Token Ledger + Collection-Point Model

**Date:** 2026-06-05 | **Status:** Proposed

### Context
TokenLens must store token-usage data from multiple surfaces with three distinct fidelities (exact, measured-activity, estimate). Two facts shape the architecture:

1. **The token stream is numeric time-series, not semantic memory.** Token counts are aggregated (summed by day, agent, project), never searched by meaning. OpenBrain's `thoughts` table is pgvector-backed for semantic retrieval; embedding numeric records there would waste compute and storage and misuse the tool.
2. **Not all agents can write to OpenBrain or to a local file.** Direct OB/MCP writers (Claude, Gee, Copi, Cursor) differ from API-only members (Lexi/Perplexity, surfaced through a sidecar) and manual-relay members (Mini). An architecture that requires every agent to self-report its tokens strands the agents that cannot write.

A further constraint: Supabase bills on database size, not tokens. Token-ledger rows are tiny (date, agent, two integers, a cost float, a lane label), so storing the exact-token stream in Supabase carries negligible cost.

### Decision
TokenLens uses a **relational `token_ledger` table** for the exact-token lane and a **collection-point model** for population.

- **Exact lane** → a relational `token_ledger` table in the *same Supabase project* that backs OpenBrain, using plain Postgres (no pgvector). One backend, two data models: `thoughts` stays vector for qualitative memory; `token_ledger` is relational for the numeric stream.
- **Measured-activity lane** → OpenBrain `thoughts` (vector), unchanged. Source tags already populate the agent/tool field; topics populate the project field.
- **Estimate band** → computed at visualization time from activity counts; never persisted as if exact.
- **Population follows a collection-point model, not per-agent self-reporting.** A single orchestrator collects usage from the surfaces that expose it (API responses, usage endpoints) and writes to `token_ledger`. Who can self-write is irrelevant when the orchestrator does the collecting.

### Alternatives Considered
- **OB-extend (embed token data in `thoughts`)** — rejected. Wrong shape for numeric time-series; generates embeddings for records that never need semantic search; conflates memory with metering.
- **Separate JSONL transcript file** (the reference-implementation pattern) — rejected as the primary store. JSONL wins only for portability off Supabase, which is not a requirement; the relational table is the correct shape for aggregation and is already hosted in the paid backend. JSONL/CSV remains available as an export format, not the source of truth.
- **Per-agent self-reporting to OB or local file** — rejected. Not all team members can write to either surface; the architecture cannot depend on a capability the agents do not uniformly have.

### Consequences
- v1 wires only the OB measured-activity lane (per PRD §6); the `token_ledger` table is designed here but populated in v2.
- The exact lane has a defined home (`token_ledger`) ready for the first collector.
- **Lexi/Perplexity is the cleanest first exact-lane collector:** the Sonar API already returns a `usage` block with exact `prompt_tokens`, `completion_tokens`, `total_tokens`, **and** a dollar-cost breakdown (`input_tokens_cost`, `output_tokens_cost`, `request_cost`, `total_cost`) in every response. The lexi-relay skill already receives this; capturing it to `token_ledger` is a one-line addition at relay time. Lexi is exact-and-cost-exact, not estimate-band.
- Other API surfaces (OpenAI API, OpenRouter) follow the same collection-point pattern; each one's exact response shape is verified when its collector is wired.
- **Supabase Metrics API** is noted as a future *infrastructure-cost* collector (DB size, the actual driver of the OB bill) — a separate concern from token lanes, but part of the same cost-visibility picture, and a candidate for a later ADR.
- The architecture is resilient to vendor change: when a vendor exposes a usage API, a new collector promotes that agent from estimate to exact with no schema change.

---

## ADR-002: Fidelity Lane Separation

**Date:** 2026-06-05 | **Status:** Proposed

### Context
Usage data arrives at three fidelities and is tempting to sum into one headline number. A blended total is dishonest: it presents an estimate as if it carried the certainty of a measured figure. This is the integrity constraint at the heart of the project — the analog of the secrets-by-reference rule in the governance layer.

### Decision
Measured, exact, and estimated data are kept structurally separate at every layer — storage, query, and view. Every figure carries its lane label. No view blends lanes into a single total. The dashboard leads with what is measured and clearly bands what is estimated. An estimate is acceptable; an estimate presented as exact is not.

### Alternatives Considered
- **Single blended total with a footnote** — rejected. A footnote does not undo a misleading headline; the blend is the violation.
- **Estimate-only (ignore exact data to keep one consistent fidelity)** — rejected. Discards the most reliable data (API usage blocks) to achieve a false uniformity.
- **Exact-only (track only what is measurable, drop chat tools)** — rejected. Would erase the majority of current activity (claude.ai and ChatGPT chat), defeating the visibility purpose.

### Consequences
- Storage separates lanes (`token_ledger` for exact; OB `thoughts` for measured activity; estimates not persisted as exact).
- Every visualization carries a persistent fidelity legend (measured / exact / estimate).
- Promoting an agent from estimate to exact (when a vendor exposes usage) is a lane change recorded explicitly, not a silent quality upgrade.

---

## ADR-003: Measurement Model Schema *(stub — to expand at review gate)*

**Date:** 2026-06-05 | **Status:** Proposed

### Context
The schema is the durable core of the project (PRD §1). Candidate fields, adapted from the reference model: Date, Agent/Tool, Project, Job, Work Type (assistant vs. computer work), Count/Token/Estimate, Fidelity Lane, Outcome, Review Burden, Next Move. Open questions for the gate: which fields are v1-required vs. forward-capture-only; how Work Type maps to OB `type`; whether Outcome and Review Burden are captured at all in v1 given historical records lack them.

*Decision / Alternatives / Consequences: to be drafted during the review gate.*

---

## ADR-004: Visualization Layer & Brand Palette *(stub — to expand at review gate)*

**Date:** 2026-06-05 | **Status:** Proposed

### Context
v1 visualization is a React artifact in Jonathan's brand palette (amber/burnt sienna, accent #c2410c, Inter + JetBrains Mono, no dark mode), showing agent distribution, project distribution, and activity-over-time, each fidelity-labeled. Open questions: charting library; whether to mirror the reference dashboard's GitHub-style heatmap and log-scale axis; static artifact vs. deployed page.

*Decision / Alternatives / Consequences: to be drafted during the review gate.*

---

## ADR-005: Public/Private Boundary for Portfolio Use *(stub — to expand at review gate)*

**Date:** 2026-06-05 | **Status:** Proposed

### Context
TokenLens lives in a public repo from inception, so the public/private boundary is a day-one constraint, not a pre-publication cleanup. Open questions: which personal usage details are shown vs. sanitized; how to reference the private governance layer without disclosing it; whether real agent nicknames appear or are abstracted; how the portfolio version separates from any private working data.

*Decision / Alternatives / Consequences: to be drafted during the review gate.*

---

*TokenLens ADR Log v0.1 (DRAFT). ADR-001 and ADR-002 proposed with full reasoning; ADR-003 through ADR-005 stubbed pending the review gate. Companion to `01_PRD_TokenLens_v0.1_DRAFT.md`.*
*Next step: run the PRD + ADR log through the review gate (Claude + Gee + Mini, Jonathan final); resolve the public-repo boundary items before first commit to the public repo.*
