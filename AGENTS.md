# AGENTS.md — TokenLens

## Source of Authority
- This project is governed by the jonathan-ops governance framework, delivered automatically via the `jonathan-governance` Codex skill. Project-specific rules below supplement, never override, the governance framework.
- TSH-9, LENS, documentation-first discipline, multi-agent review gate, public/private boundary, and secrets-by-reference are defined in the governance framework and apply to this repo.
- If this file conflicts with governance, governance wins.

## Project Shape
- TokenLens is a public-repo, personal AI-ops telemetry project for visibility into delegated intelligence spend.
- The measurement model is the system; the dashboard is only a delivery surface.
- Current docs are draft/pre-review-gate. Treat PRD/ADR decisions as Proposed unless Jonathan accepts them.

## Architecture Conventions
- Preserve fidelity lanes at storage, query, and view layers: `exact`, `measured_activity`, `estimate`.
- Never blend exact, measured, and estimated data into one total, headline, chart, or summary.
- Every numeric figure must carry a fidelity-lane label.
- v1 scope is OpenBrain measured-activity only; exact-token collectors and estimate methodology are later unless approved.
- Exact-token lane target is relational `token_ledger`; measured-activity lane remains OpenBrain `thoughts`; estimates are computed, not persisted as fact.
- Canonical telemetry uses per-source adapters, preserves `raw_payload`, and derives cost from versioned pricing instead of storing cost as fact.
- Judgment fields belong in a separate overlay, not in auto-captured telemetry.

## Public Boundary
- This is a public repository: do not commit live secrets, credential values, OB instance URLs, private local paths, or private-only personal context.
- Reference secrets and credentials by approved location only; never paste secret contents into code, docs, commits, or chat.
- Before any commit or publication, check that private repo details, job-search posture, and personal infrastructure details are intentionally disclosed or sanitized.

## Approval Required
- Do not create, modify, delete, rename, stage, commit, push, or publish files without Jonathan's explicit approval.
- Do not change PRD/ADR status from Proposed/Draft to Accepted/Finalized without the review gate and Jonathan's final call.
- Do not add dependencies, build systems, telemetry collectors, external services, or database schema changes without approval.
- Do not run destructive commands or broad git operations.

## Commands
- Build: none defined yet.
- Test: none defined yet.
- Lint/format: none defined yet.
- Until commands exist, verify by reading affected docs/schema paths and reporting that automated checks are unavailable.

## Verification Before Reporting Done
- Confirm intended files only were changed; if no approval was given, confirm no files were changed.
- Re-read changed content for fidelity-lane honesty, public/private leakage, and governance compliance.
- If code or data views are added later, verify no blended totals and every metric has a lane label.
- Report exact commands run, checks performed, unavailable checks, and remaining approval/review gates.