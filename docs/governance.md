# jonathan-ops — Operating Governance

**Version:** 1.4
**Status:** Source of truth for operational governance (file, repo, secret, and docs handling). The OpenBrain canonical mirror is the one derived copy of this file (ADR-001, ADR-002 amended). When the two disagree, this file wins.
**Maintained by:** Claude (Project Lead) on behalf of Jonathan Openshaw (Architect of Record)
**Repo:** `ai-team-ops/jonathan-ops/governance.md`
**Date:** 2026-06-04

**Governing anchor:** אֲנִי קוֹחֵז בָּאֱמֶת — *I anchor in truth.*

**Precedence:** Subordinate to TSH-9 (interaction character) and jonathan-context (identity/voice); governs the operational layer only. Full stack: TSH-9 → jonathan-context → LENS → this file (ops) → DCS (state).

> This file encodes the cross-cutting rules for how **any agent** works with Jonathan's local files, Git repositories, secrets, and project documentation. Rules name the action and the actor abstractly ("the committing agent," "any agent with filesystem access") — never a specific agent — so the same governance installs cleanly on any agent profile and survives capability changes (ADR-003). Charter, decision record, and review process: see `docs/`.

---

## 1. Git commit convention

The committing agent uses the standard minimal four-line block:

```
cd <repo path>
git add <specific file>        # name the file(s); never a blind `.`
git commit -m "<message>"
git push origin <branch>
```

- Name the specific file(s) being committed. A blind `git add .` is not the convention; when several intended files are added together, list them or add a single intended subtree explicitly.
- No defensive scripting, status checks, branch detection, or error handling unless Jonathan asks for it.
- Confirm the default branch before pushing (`main` or `master`); do not assume.
- All canonical commits flow through the governed pipeline regardless of which agent holds write capability.

## 2. Filesystem topology

Canonical local paths under the `Documents_PC` root:

- `00_JobSearch` — job-search working files
- `01_Resumes` — resume masters and archive
- `GIT_Repo_Private` — private repositories
- `Git_Repo_Public` — public repositories

A single GitHub account, `jropenshaw1`, holds mixed-visibility repositories (public and private). Any agent with filesystem access follows these path conventions; an agent without filesystem access does not guess paths and does not fabricate them.

## 3. Public / private visibility boundary

Private-repo content — personal context, contact details, search posture, infrastructure detail, local paths — never reaches a public repo. No secrets are committed to any repo, public or private. Before any sync or commit that touches a public repo, the acting agent verifies nothing private has crossed the boundary.

## 4. Secrets by reference only

Live secrets are referenced by location only, never reproduced. The credential file (`CredentialTracker.md`) lives outside every repo tree, at `Documents_PC\03_AIProjects\OpenBrain\`; its contents never enter this file, the OpenBrain mirror, chat, or any agent context. The rotation runbook is the set of inline per-credential checklists inside that file, referenced by location and never copied out. Secrets live only in that one file and are never committed to any repo.

> **Propagation exception (ADR-005 corollary):** The credential-file path above is recorded in this file only — it is a private-repo (`ai-team-ops`) topology fact. It is intentionally **absent** from the OpenBrain canonical mirror, which carries §4 without the path. Section 10's byte-faithful propagation does **not** apply to this one line; a propagate-and-verify run must treat the missing path in the mirror as intended, not as drift to correct. Rationale: this is deliberate documentation, not security-through-obscurity. The path is recorded because an undocumented location is an operational risk (it will be forgotten), and the path's secrecy is not a control — the credential file sits behind real OneDrive filesystem protections, and an actor past those protections is already compromised regardless of whether the path was written down. The path is kept to the private repo simply because the OB mirror has wider and differently-scoped reach; there is no reason to widen the path's footprint beyond where it is needed. The credential-file path is configuration metadata, not credential material. This exception applies only to the credential-path reference defined in this section (§4) and may not be generalized to other governance content without a separate ADR.

> An agent that needs a credential is pointed to the credential file by location and retrieves it there. It does not ask for the secret to be pasted, and it does not write the secret into any artifact.

## 5. Personal-OB / corporate boundary

Personal OpenBrain never connects to a corporate-licensed agent. A corporate-licensed instance of any agent is treated as outside the trust boundary for personal OB, regardless of how capable it is.

## 6. Filesystem-MCP preference

When a bulk write through the filesystem MCP would be slow or token-expensive, the acting agent offers a manual path (e.g., produce the file for Jonathan to save) rather than forcing the expensive automated route. Efficiency and Jonathan's token budget are part of the decision.

## 7. Resume sync triad

The `.docx` is canonical. The `.md` and `.pdf` regenerate from it. Before syncing, verify the source `.docx` is the current version; never regenerate the `.md`/`.pdf` from a stale source. (This is the one place a `.docx` is canonical by design — distinct from narrative docs like the team roster, where the `.md` is authoritative.)

## 8. Documentation-first discipline

Every new project starts with a PRD (purpose, in-scope, out-of-scope, concrete Definition of Done) and an ADR log, written before build, following the AegisRelay docs standard. The PRD states *what*; the ADR log states *why*, with rejected alternatives. Both precede code. New projects carry a `docs/` folder holding at least these two.

## 9. Multi-agent review & sign-off gate

Charters, PRDs, and ADR logs are **Finalized** only after sign-off from at least three reviewers — the authoring agent as first reviewer plus two additional agents from **distinct foundation-model families** — with Jonathan as final authority. The default reviewer pair is Gee (OpenAI) and Mini (Google), which with Claude (Anthropic) spans three families. Two same-family reviewers are not a valid default.

The orchestrator collects feedback by two paths — file-capable reviewers read the artifact directly; reviewers that cannot reach the file receive it inline — integrates a consensus summary, and presents a finalization recommendation. Jonathan makes the final call.

**Orchestrator fallback:** the default orchestrator/integrator is Claude. If Claude is unavailable, or is the author of the artifact under review (or otherwise materially conflicted), Gee acts as backup orchestrator. Because Gee has no local-filesystem access, when acting it produces the consensus summary and any revised artifact as a downloadable artifact for Jonathan to save.

Full process, schemas, and prompt templates: `docs/03_Review_Handoff_Process_v1.1.md`. Decision record: `docs/02_ADR_Log_v1.1.md` (ADR-007, ADR-008).

## 10. Propagate-and-verify (meta-rule)

A change to this governance is not done until it is (1) written to both **Standing Channels** — this file (source) and the OpenBrain canonical mirror — and (2) each write is verified to have landed. Propagate, then verify each, then done. For OpenBrain, verification is a `search_thoughts` confirmation after the write. An unverified write is an assumed write, not a completed one.

**Channel taxonomy (ADR-002 amended):**
- **Standing Channels** (propagate-and-verify targets): the source file (Filesystem) and the OpenBrain canonical mirror.
- **Transient Delivery Artifact** (NOT a propagate-and-verify target): the on-demand paste block generated from source for non-file, non-OB agents (web-Claude, Mini, Lexi). It is produced fresh from the current source at point of need and is never tracked, mirrored, or verified as a standing copy.
- **Capability coverage:** every known agent class is covered by at least one access path — file-capable agents (Claude Code, Cursor, Copi) via the Filesystem pointer, OB-capable non-file agents (Gee) via the mirror, and transient/manual agents via the generated paste block.

The jonathan-ops skill is retired as a governance distribution channel as of v1.4 and retained only as historical context (it is not a Standing Channel and not a Transient Delivery Artifact).

---

## Change discipline

Any change originates **here**, in governance.md, then propagates outward to the OpenBrain mirror (ADR-004). The mirror is never edited as an independent original. Bump the version line above on any substantive change, and run the propagate-and-verify gate (Section 10) before considering the change complete.

---

*Source of truth for jonathan-ops governance. The derived mirror (OpenBrain canonical entry) carries a footer naming this file as canonical. Part of the `ai-team-ops` private repository.*
