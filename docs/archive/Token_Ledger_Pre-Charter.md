# Token Ledger — Pre-Charter

**Status:** Pre-Charter (Research Complete, Ready for Implementation Day)
**Created:** 2026-05-23
**Author:** Jonathan Openshaw + Claude
**Origin:** Tangent from JFA Agent 5 removal session — identified during discussion of token savings from removing non-functional Resume Optimizer agent

---

## 1. Problem Statement

Jonathan operates a multi-AI environment (Claude chat, Claude API, Perplexity/Lexi API, OpenBrain MCP) with no unified visibility into where tokens flow, what they cost, or what value they produce. This mirrors the exact challenge enterprises face at scale: Choice Hotels has basic adoption metrics (token burn as a proxy for engagement) but no cost-to-value governance layer. The gap between "we can see how much people use AI" and "we can see whether that usage produces value" is where the Token Ledger sits.

Jonathan's personal AI spend is self-financed, making cost governance a direct financial concern — not a theoretical exercise.

## 2. Stated Goals

1. **Visibility first.** See and understand token spend data before attempting to optimize or attach value metrics. No value-tagging taxonomy until the data tells us what categories matter.
2. **Instrument what's instrumentable.** API calls (JFA runs, Lexi queries) return `usage` objects with `input_tokens` and `output_tokens`. Capture these automatically.
3. **Self-classify what isn't.** Claude.ai chat doesn't expose per-conversation token telemetry. Develop a lightweight self-classification method for chat sessions by purpose — this is what enterprises will need to do too.
4. **Build portfolio-grade governance.** The Token Ledger isn't just a personal tool. It's a demonstrable artifact showing hands-on AI cost governance at a level most enterprises haven't reached. It should extrapolate to enterprise scale.
5. **Companion to Signal Ledger.** Same architectural discipline — Supabase/pgvector persistence, structured schema, semantic queryability. Beginning of an "AI Ledger Suite" pattern.

## 3. Token Streams Identified

| Stream | Source | Instrumentable? | Notes |
|--------|--------|-----------------|-------|
| Claude.ai chat | Subscription (Pro) | No — no per-conversation telemetry exposed to user | Highest personal token volume. Must self-classify by purpose. |
| Anthropic API | JFA skill runs, Claude Code, artifact API calls | Yes — `usage` object in every response (`input_tokens`, `output_tokens`) | Fully instrumentable. Priority capture target. |
| Perplexity/Lexi API | Sonar research queries via lexi-relay skill | Yes — API returns token usage | Lower volume but consistent. |
| OpenBrain MCP | Supabase Edge Functions | Not token-burning (no LLM inference) | API call costs, not token costs. Track separately. |

### Enterprise Parallel

| Personal Stream | Enterprise Equivalent |
|----------------|----------------------|
| Claude.ai chat | Managed chat interfaces (Copilot, ChatGPT Enterprise, Claude for Work) |
| Anthropic API | API-integrated workflows (automated agents, internal tools) |
| Lexi API | Third-party AI services (search, summarization, translation vendors) |
| OpenBrain MCP | Infrastructure calls (vector DBs, embeddings, orchestration) |

## 4. Research Findings — Nate's Archive

### 4.1 Primary Prompt Kit: "You're Burning Money and Blaming the Model"

**Kit ID:** `695e178b-3c89-48e9-8f61-f4f53149e816`
**Published:** 2026-04-02
**Companion Article:** "You're Loading 66,000 Tokens of Plugins Before You Even Type" (post ID: `1d6fe144-8430-467e-825d-9c822e50f663`, URL: https://natesnewsletter.substack.com/p/your-claude-sessions-cost-10x-what)

Five prompts forming a complete token discipline toolkit:

**Prompt 1 — The Stupid Button (Token Burn Diagnostic)**
- Audits actual AI habits against six waste patterns
- Produces a 1-10 burn score with prioritized fixes
- Six waste patterns: (A) Raw Document Ingestion, (B) Conversation Sprawl, (C) Model Misuse, (D) Plugin/Skill Boot Tax, (E) No Prompt Caching, (F) Expensive Web Research
- **Token Ledger relevance:** These six patterns should inform the Token Ledger's classification taxonomy. Run this diagnostic as a baseline before implementation.

**Prompt 2 — The Context Rescue**
- Extracts minimum viable context from bloated conversations for fresh start
- Five-section format: Task State, Decisions Locked, Key Outputs, Open Questions, Next Step
- **Token Ledger relevance:** Already adopted into DCS workflow. Validates that context compression is a core token savings behavior to track.

**Prompt 3 — The Model Router**
- Creates personalized model routing plan (Tier 1: Reasoning, Tier 2: Execution, Tier 3: Cleanup)
- Maps specific recurring tasks to cheapest adequate model tier
- **Token Ledger relevance:** Model routing is a governance behavior the Token Ledger should capture — which model was used for which purpose category.

**Prompt 4 — The KISS Audit (Agent Architecture Waste Finder)**
- Audits agent pipelines against five commandments
- Commandment 5 is literally "Measure What You Burn"
- Five commandments: (1) Index Your References, (2) Prepare Context for Consumption, (3) Cache Your Stable Context, (4) Scope Each Agent's Context to Minimum, (5) Measure What You Burn
- **Token Ledger relevance:** The Token Ledger IS the implementation of Commandment 5. The other four commandments define optimization categories to track.

**Prompt 5 — The Token Translator**
- Reconstructs token economics of an actual session
- Builds side-by-side profiles: actual vs. clean habits
- Key mechanics: Claude resubmits full context every turn (cost escalates multiplicatively), raw PDF ingestion is 5-20x more expensive than markdown, web search adds 10K-50K tokens per search
- **Token Ledger relevance:** The methodology here should inform how the Token Ledger estimates costs for non-instrumented streams (chat sessions).

### 4.2 Secondary Prompt Kit: "OpenAI's New $20K AI Employee"

**Kit ID:** `1fcc503e-7370-45e6-a970-25ac0ac1749a`
**Published:** 2026-02-19

**Prompt 2 — Token Management Maturity Audit** is the enterprise governance framework:

Five maturity dimensions:
1. **Model Routing Intelligence** — deliberate task-to-model matching (1=everyone uses whatever, 5=automated routing with cost-quality optimization)
2. **Context Engineering** — how well information is structured for AI consumption (1=copy-paste into ChatGPT, 5=systematic context pipelines)
3. **Measurement & ROI Tracking** — ability to quantify value per dollar of token spend (1=no measurement, 5=real-time dashboards)
4. **Organizational Structure** — clear ownership of AI operations (1=no ownership, 5=dedicated intelligence ops function)
5. **Supplier Strategy** — strategic AI vendor management (1=month-to-month default, 5=negotiated multi-provider agreements)

Five maturity levels:
- Level 1: Experimental (tool individuals use, not org capability)
- Level 2: Departmental (some teams systematic, no org-wide strategy)
- Level 3: Strategic (centralized with measurement, not yet optimized)
- Level 4: Optimized (deliberate routing, ROI tracking, vendor strategy)
- Level 5: AI-Native (intelligence throughput is primary operating model)

**Token Ledger relevance:** This maturity model is what Jonathan would use to assess an employer's AI governance posture — or pitch his own governance capability. The Token Ledger demonstrates Dimension 3 (Measurement & ROI Tracking) hands-on.

### 4.3 Related Articles Found (Not Yet Read in Full)

- "Dear Manager, Your AI Budget is Costing Me My Career" (post ID: `20162033-d5c4-4127-9694-f859505e2596`) — AI budget justification from employee perspective
- "Executive Briefing: How to Buy The Right AI Solution and Save Millions" (post ID: `3cec3d06-4a6c-4c98-861b-979289390328`) — enterprise AI procurement
- "Why the Pipes Suddenly Matter" (post ID: `6307734e-35f4-49cb-b0e1-e84f82733c94`) — tokenization and distribution strategy
- "You Will Choose Wisely: Definitive Guide to Picking an AI Plan" (post ID: `c31f884e-ecc9-4361-b626-caf59791cf2e`) — plan selection economics

## 5. Key Insight: What Exists vs. What Doesn't

Nate has built the **diagnostic and framework** layer — what to measure, why it matters, how to audit habits and architectures. This is the conceptual foundation.

Nobody has built the **persistent telemetry and data capture** layer — the always-on monitoring system that captures token usage over time and makes it queryable. The Stupid Button is a one-time audit. The Token Ledger would be the continuous measurement infrastructure that Commandment 5 demands but nobody has implemented as a reusable tool.

**The Token Ledger fills the gap between Nate's frameworks and operational reality.**

## 6. Technical Assumptions (Pre-Charter)

- **Persistence:** Supabase (consistent with Signal Ledger, OpenBrain ecosystem)
- **Schema approach:** LENS-pattern documentation (Charter, Data Dictionary, Functional Spec, ADR Log, Definition of Done) — same discipline applied to Signal Ledger
- **Capture method for API calls:** Post-response hook that extracts `usage.input_tokens` and `usage.output_tokens` from every Anthropic/Perplexity API response, writes structured record
- **Capture method for chat sessions:** Manual or semi-automated purpose classification after session completion — log session date, estimated duration, purpose category, outcome
- **Token-to-cost conversion:** Maintain a rate table mapping model + token type (input/output/cache-hit) to current pricing. Apply at query time, not capture time, so historical data re-prices accurately when rates change.
- **Query interface:** Semantic search via pgvector (consistent with Signal Ledger) plus structured queries for spend-by-category, spend-over-time, cost-per-outcome

## 7. What This Is NOT (Scope Boundaries)

- **Not a value-tagging system yet.** Visibility first. Value attribution comes after we understand the data.
- **Not a real-time dashboard.** Start with structured log entries and batch analysis. Dashboard is a future phase.
- **Not an enterprise product.** It's a personal governance tool that demonstrates enterprise-applicable patterns.
- **Not a replacement for Anthropic Console.** Console provides aggregate API spend. Token Ledger provides per-skill, per-agent, per-purpose granularity plus the chat session layer Console doesn't cover.

## 8. Proposed Next Steps (For Implementation Day)

### Step 1: Run the Stupid Button Diagnostic
Run Nate's Prompt 1 from the "Burning Money" kit as a baseline audit of Jonathan's current AI habits. This produces the first data point and validates what categories the Token Ledger needs to track.

### Step 2: Draft the Token Ledger Charter
Using LENS-pattern documentation discipline. Define scope, non-scope, success criteria, and the schema.

### Step 3: Design the Data Dictionary
Define the token_log record schema. Candidate fields:
- `timestamp` — when the call/session occurred
- `source_system` — claude_chat | anthropic_api | perplexity_api | openbrain_mcp
- `model` — specific model used (claude-opus-4-6, claude-sonnet-4-6, sonar-pro, etc.)
- `skill_name` — which skill or process generated the call (jfa, lexi-relay, quick-fit, etc.)
- `agent_step` — for multi-agent skills, which agent within the skill
- `input_tokens` — from usage object (API) or estimated (chat)
- `output_tokens` — from usage object (API) or estimated (chat)
- `cache_hit_tokens` — if prompt caching is used
- `estimated_cost` — calculated from rate table at query time
- `purpose_category` — job_search | portfolio_build | learning | admin | personal
- `outcome_tag` — application_submitted | role_passed | content_published | skill_built | none
- `session_id` — groups related calls within a single work session
- `notes` — freeform context

### Step 4: Implement API-Side Capture
Instrument the JFA skill and lexi-relay skill to write token_log entries after every API call. Start with these two because they're the highest-volume API consumers.

### Step 5: Design Chat Session Classification
Build a lightweight end-of-session logging pattern for Claude.ai chat — purpose, approximate duration, outcome. This is the hardest stream to instrument and the one with the most enterprise relevance.

### Step 6: First Analysis
After one week of capture, run the Token Translator (Nate's Prompt 5) against actual data to validate the schema and identify gaps.

## 9. Reference Materials Index

| Resource | Location | Purpose |
|----------|----------|---------|
| "Burning Money" Prompt Kit (full text) | Executive Circle kit ID `695e178b-3c89-48e9-8f61-f4f53149e816` | Primary framework — six waste patterns, five KISS commandments, token translation methodology |
| "Burning Money" Article | Post ID `1d6fe144-8430-467e-825d-9c822e50f663`, URL: natesnewsletter.substack.com/p/your-claude-sessions-cost-10x-what | Companion article with production pipeline benchmark (<$0.25/user) |
| "$20K AI Employee" Prompt Kit (full text) | Executive Circle kit ID `1fcc503e-7370-45e6-a970-25ac0ac1749a` | Token Management Maturity Audit framework (five dimensions, five levels) |
| Signal Ledger architecture | Supabase project (existing) | Architectural precedent for persistence, embedding, and semantic search |
| LENS documentation pattern | Six locked documents (Charter through Definition of Done) | Documentation discipline template |
| Anthropic API usage object | Every `/v1/messages` response includes `usage.input_tokens` and `usage.output_tokens` | Ground-truth telemetry source |
| Anthropic token counting endpoint | `/v1/messages/count_tokens` | Pre-call cost forecasting |

## 10. Connection to Career Narrative

The Token Ledger positions Jonathan at the intersection of FinOps and AI Governance — his two strongest professional domains. Specifically:

- **Demonstrates Dimension 3 (Measurement & ROI Tracking)** from Nate's maturity model — the dimension most enterprises score lowest on
- **Extends the FinOps discipline** Jonathan built at Choice Hotels ($34M+ portfolio, zero audit findings) into the AI cost domain
- **Provides a concrete artifact** for interviews: "I built a token telemetry system, ran it on my own multi-AI environment, and here's what the data showed about cost-to-value ratios across different AI usage patterns"
- **Bridges the gap** between Choice's current state (adoption metrics) and the governance layer they haven't built yet — Jonathan can speak to both sides with hands-on evidence

---

*This pre-charter captures all research and decisions from the 2026-05-23 session. Ready for a dedicated implementation day. No data lost.*
