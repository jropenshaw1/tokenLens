# 01 — TokenLens Product Requirements Document

**Version:** 0.1 (DRAFT — pre-review-gate)
**Status:** Draft — not yet through ADR-007 review gate (reviewers TBD: Claude + two distinct-model agents; Jonathan final authority)
**Author:** Jonathan Openshaw
**Maintained by:** Claude (Project Lead)
**Date:** 2026-06-05
**Phase:** Job Search

> Working title **TokenLens** is provisional. Alternatives considered: BurnLedger, TokenScope. Final name is Jonathan's call.

---

## 1. Purpose

TokenLens is a token-visibility instrument for Jonathan Openshaw's multi-agent AI team. It ties delegated work to the compute it consumes, so that token usage stops being an invisible byproduct and becomes a feedback loop for how intelligence is allocated across agents.

The thesis, borrowed from Nate Jones's token-burn framing and extended for a governed multi-agent operation: **a token count is not a scoreboard, it is a trace.** It shows where work was handed to an agent, how much delegated intelligence was spent, and whether the result earned the spend. Tied to outcomes, it answers the question that matters — what should be delegated next, to which agent.

**The dashboard is a delivery surface. The measurement model is the system.**

**Governing anchor:** אֲנִי קוֹחֵז בָּאֱמֶת — *I anchor in truth.*

---

## 2. Problem Statement

Jonathan delegates work across six agents (Claude, Gee, Lexi, Mini, Copi, Cursor) coordinated through OpenBrain, but has no unified view of what each delegation costs or returns. Three observed gaps motivate this project:

- **Cost invisibility.** OpenBrain already captures ~1,400 activity records tagged by agent, topic, and type — but records *what work moved*, not *what it cost*. There is no bridge from activity to token spend.
- **Fidelity fragmentation.** Each tool exposes usage differently. The Anthropic API returns exact tokens; Cursor exposes partial usage; claude.ai and ChatGPT expose nothing. Without a discipline that keeps these lanes separate, any single "total" is dishonest.
- **No allocation intelligence.** Jonathan cannot currently answer the questions his own brainstorm surfaced: which agent is most efficient for which task type, where tokens are wasted, how usage patterns shift with new models, or how usage data could inform subscription and vendor negotiations.

TokenLens closes these gaps by making delegated intelligence an explicit, fidelity-honest, multi-source measurement artifact with a visualization layer on top.

### Value dimensions (from Jonathan's 2026-06-05 brainstorm)

| Dimension | Question TokenLens should eventually answer |
|---|---|
| Utilization | Which tools do I use, and for what? |
| Value / waste | Which tokens add value, which are wasted, where can I optimize? |
| Agent efficiency | Which agents are more efficient on which task types? |
| Transparency | Which tools expose utilization and cost metrics at all? |
| Negotiation leverage | How can usage data inform purchase / subscription negotiations? |
| Pattern evolution | How do usage patterns evolve, driven by what — new models, new agents? |
| Portfolio | Is this a public portfolio piece demonstrating FinOps + governance discipline? |

The negotiation-leverage and agent-efficiency dimensions are where TokenLens diverges from Nate's personal-delegation framing into a vendor-governance and capability-allocation instrument — Jonathan's distinct lane.

---

## 3. Governance Standards

- **TSH-9** — Hebrew-rooted AI alignment framework v9.0 (Ferocity Standard). Truth-anchored, direct, peer-level engagement.
- **LENS** — Metacognitive interaction protocol; reframe / irreversibility / cascade detection before durable commits. https://github.com/jropenshaw1/LENS
- **Documentation-first** — this PRD is the project's first instance; an ADR log accompanies it. Template: the AegisRelay docs standard.
- **Multi-agent review gate** — this charter is Finalized only after sign-off from three reviewers (Claude + two distinct-model agents, default Gee + Mini) with Jonathan as final authority, per ADR-007 of jonathan-ops.
- **Fidelity honesty (project-specific).** Measured, exact, and estimated data are never folded into a single total. Every figure carries its lane label. This is the TokenLens analog of the secrets-by-reference rule: a non-negotiable integrity constraint.

---

## 4. Data Sources & Fidelity Lanes

Three fidelity lanes, kept structurally separate per Section 3. The honest move is to lead with what is measured and clearly band what is estimated.

| Agent / Source | Lane | What is available today |
|---|---|---|
| Anthropic API (artifacts, Claude Code if adopted) | **Exact tokens** | Token counts returned in API response usage block |
| OpenBrain | **Measured activity** | ~1,400 records tagged by agent, topic, type, date — counts, not tokens |
| Cursor | **Measured activity / partial** | Usage metrics may be exposed in settings; needs investigation |
| Copi (Copilot) | **Measured activity / partial** | Minimal token visibility on current tier |
| claude.ai (Pro chat) | **Estimate band** | No token exposure in app, billing, or export |
| Gee (ChatGPT) | **Estimate band** | No token exposure in app or export |

OpenBrain's source tags (`[source:claude]`, `[source:chatgpt]`, `[source:cursor]`, `[source:copi]`) already populate the "tool or model" field of the measurement model. This is the project's head start.

---

## 5. Core Design Decisions

| ADR | Title | Status |
|---|---|---|
| ADR-001 | Data Architecture: OB-Extend vs Separate JSONL vs Hybrid | **Proposed** |
| ADR-002 | Fidelity Lane Separation | Proposed |
| ADR-003 | Measurement Model Schema | Proposed |
| ADR-004 | Visualization Layer & Brand Palette | Proposed |
| ADR-005 | Public/Private Boundary for Portfolio Use | Proposed |

ADR-001 is drafted below as the gating decision. The remainder are placeholders to be expanded during the review gate.

---

## 6. v1 Scope

### In Scope

- **Measurement model schema** — the canonical field set (Date, Agent/Tool, Project, Job, Work Type, Count/Token/Estimate, Fidelity Lane, Outcome, Review Burden, Next Move), documented and versioned.
- **Measured-activity lane from OpenBrain** — pull and structure the ~1,400 existing records into the schema. This is free today and requires no new capture.
- **First visualization** — a React artifact in Jonathan's brand palette (amber/burnt sienna, Inter + JetBrains Mono, no dark mode) showing agent distribution, topic/project distribution, and activity-over-time, with explicit fidelity-lane labels.
- **Fidelity-lane discipline** — measured/exact/estimate kept separate in all views; no blended totals.

### Out of Scope for v1

- **Exact-token capture pipeline** from the Anthropic API — design in v2 once the measured-activity loop is proven.
- **Estimate-band methodology** for claude.ai and ChatGPT — the interview-the-user approach Nate describes; v2.
- **Cursor / Copi usage extraction** — investigate feasibility, but not a v1 deliverable.
- **Automated capture / sync tooling** — v1 is manual pull and manual refresh.
- **Outcome and review-burden backfill** — these fields are in the schema but historical records lack them; forward-capture only, no retroactive scoring.
- **Public portfolio version** — sanitization and publication is a separate, later effort (see ADR-005 placeholder).
- **Negotiation-leverage and agent-efficiency analytics** — these are the *destination* (Section 2 value table) but require the exact-token lane first; explicitly deferred.

---

## 7. Definition of Done (v1)

1. **PRD and ADR log** committed to a TokenLens project `docs/` folder following the AegisRelay/jonathan-ops standard.
2. **Measurement model schema** documented and versioned, with every field defined and its fidelity-lane handling specified.
3. **OB measured-activity pull** — the existing records structured into the schema, with a repeatable query (not a one-time export).
4. **First visualization** renders real OB data in the brand palette, with agent distribution, project distribution, and activity-over-time, each carrying a fidelity-lane label.
5. **Fidelity-honesty check** — no view blends measured, exact, and estimated data into a single total; every figure is labeled.
6. **Review gate exercised** — this PRD and ADR log run through the ADR-007 process (Claude + two distinct-model agents, Jonathan final) and reach Finalized status.
7. **Boundary check** — the visualization and any committed artifact contain no live secrets and no private-only context that would block eventual portfolio use; OB instance URL and credential paths referenced by location only, never reproduced.

---

## 8. Project Context

TokenLens is part of Jonathan Openshaw's personal AI infrastructure stack. It is the cost-and-allocation companion to OpenBrain (memory) and jonathan-ops (governance), and a candidate portfolio artifact demonstrating FinOps discipline applied to AI operations. It is not a commercial product.

**Related:**
- `02_ADR_Log_TokenLens_v0.1.md` — companion ADR log (to be created)
- OpenBrain — measured-activity source and the project's head start
- jonathan-ops — governance layer this project inherits from
- [LENS](https://github.com/jropenshaw1/LENS) — metacognitive governance protocol (naming lineage)
- Nate Jones, *I Built a Token Burn Dashboard* (2026-06-05) — conceptual foundation
- `nateherkai/token-dashboard` — reference implementation (Claude Code JSONL; design reference only)

---

# ADR-001 — Data Architecture: OB-Extend vs Separate JSONL vs Hybrid

**Status:** Proposed
**Date:** 2026-06-05
**Decision owner:** Jonathan (final authority per ADR-007)

## Context

Nate's reference implementation reads JSONL transcripts that Claude Code writes natively to `~/.claude/projects/`. Jonathan does not use Claude Code as a primary surface, so he has no native JSONL transcript stream. The measurement data must come from somewhere, and the three lanes (Section 4) have different native formats: API returns JSON usage blocks, OB stores Supabase rows, chat tools expose nothing.

The architecture must decide where the canonical measurement record lives.

## Options

**Option A — Extend OpenBrain.** Add token/outcome fields to OB capture entries; OB becomes the single source of truth for both activity and cost.
- *Pros:* one source of truth; reuses existing source-tagging; no new store; activity and cost co-located.
- *Cons:* couples cost tracking to memory writes; chat-tool entries still have no token data to store; risks bloating OB's purpose.

**Option B — Separate JSONL transcript log.** Write a per-session JSONL file mimicking Nate's format, fed from each lane.
- *Pros:* matches the reference implementation; clean separation from memory; portable.
- *Cons:* manual to populate for chat tools; a second store to maintain; duplicates the agent/project fields OB already has.

**Option C — Hybrid.** OB remains the canonical activity record (measured-activity lane); a side-log (JSONL or CSV) captures the exact-token lane from the API; estimate bands computed at visualization time and never persisted as if exact.
- *Pros:* each lane stored in the format that fits it; OB stays a memory system; exact tokens get a clean home; honors fidelity separation at the storage layer, not just the view.
- *Cons:* two stores to reconcile; visualization must join across them.

## Recommendation (for review)

**Option C — Hybrid**, deferred in part to v1 scope. For v1, only the OB measured-activity lane is wired (Section 6), which means v1 touches only the OB side of the hybrid. The exact-token side-log is designed in ADR-001 but built in v2. This lets v1 ship a real feedback loop from data that already exists, while committing to an architecture that won't require rework when the exact-token lane is added.

This recommendation is **Proposed**, not Accepted. It awaits the review gate.

## Consequences if accepted

- v1 builds against OB only; no new store created yet.
- v2 adds the exact-token side-log per the format specified here.
- The visualization layer is designed from day one to join across lanes and label fidelity, even when only one lane has data.
