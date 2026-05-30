# ADR-002: The drive↔converge handoff is a typed PR-convergence state machine owned at the ecosystem layer

Status: proposed
Date: 2026-05-29

> Cross-repo ADR. Governs the seam between **codebase-mapper/cbm** (`/goal` drive engine) and **agentic-review-loop / ARL** (convergence controller), recording into **pr-review-journal** (ledger). Derived from the AOR lab design (`agentic-family-design/01-autonomous-drive-to-maturity-loop.md` §3 S1). Companion to [ADR-003](ADR-003-cross-model-gate-no-skimp-guarantee.md); the loop requirements it imposes on ARL are tracked in `agentic-review-loop#15`.

## Context

The autonomous drive-to-maturity loop (driving cbm H2→H5) composes cbm `/goal` (produces a PR per stage slice) with ARL (monitors bots/CI, ingests findings, converges to merge) and resumes `/goal` on merge. **Neither cbm nor ARL owns this handoff today** — it is the literal core of the loop and a seam no single repo can own (the ecosystem charter: a decision belongs here only if changing it requires edits in more than one repo).

Three facts constrain the design:

1. **Agent PRs fail at *converging*, not at writing code.** ≈68% of rejected agentic PRs carry no explicit reviewer feedback (arXiv 2602.04226, T3); the leading non-merge cause is unresolved test failures (arXiv 2602.00164, T3); merge-time is bimodal — fast or never (arXiv 2604.00917, T3). The converger cannot assume it will be told *why*.
2. **Blindly applying bot findings is harmful.** AI-reviewer suggestions are adopted at 16.6% vs 56.5% for humans; 28.7% of unadopted AI suggestions are simply wrong; adopted ones add ~10× cyclomatic complexity (arXiv 2603.15911, T3). Convergence must be triage-gated, finding-localized, turn-bounded.
3. **ARL and pr-review-journal already have a contract** ([ADR-001](ADR-001-review-journal-stays-independent.md)): the journal's verdict schema is the integration contract; ARL consumes `pr-review-triage`, does not reimplement it, and the dependency direction is fixed **controller → ledger, never the reverse**. This ADR extends that contract *upward* to the drive engine without violating that direction.

## Decision

Define and own, at agentic-ecosystem, a **typed PR-convergence state machine** as the drive↔converge contract:

| Ledger state | Meaning | Transitioned by |
|---|---|---|
| `ci_pending` / `ci_failed` / `ci_green` | CI status — a `gh pr checks` **harness fact**, never a model belief | `pr-watch` / harness |
| `review_pending` | bots fired, no verdict yet | `pr-watch` |
| `findings_open` | triaged findings awaiting address | `pr-review-triage` (ARL P4) |
| `findings_addressed` | re-pushed; awaiting re-review | ARL P5 executor |
| `rejected_silent` / `rejected_with_reason` / `stale` | first-class non-merge verdicts | ARL P4 |
| `converged` → `merged` | gates pass; auto-merge fired | ARL ADR-001 |

Binding rules:
- **`/goal` resumes ONLY on `converged → merged`** (re-anchor goal + re-validate state fingerprint + act-once; see [ADR-003](ADR-003-cross-model-gate-no-skimp-guarantee.md)).
- **ARL reads only the typed acceptance criteria + ledger states — never `/goal`'s intent inferred from the PR body** (the PR body is untrusted content; injection surface).
- **`/goal`'s per-PR slice stays small** (oversized PR is an agent-only rejection mode) — this constraint flows *back* onto the drive engine; cbm's per-stage allowed/disallowed write-set already bounds it.
- **The merge→resume signal is ARL P5's `bin/pr-watch` `pr_closed_or_merged` terminal event** — the engine parks on the same blocking file contract and resumes on that event (`agentic-review-loop#15`, P5 requirement).
- **ARL-posture → journal-verdict mapping is specified here.** ARL's prospective postures (its 9-posture vocabulary) are mapped onto the journal's retrospective 8 values (`ACCEPTED` / `ACCEPTED_MODIFIED` / `DEFERRED` / `REJECTED_FALSE_POSITIVE` / `REJECTED_BAD_FIT` / `REJECTED_REGRESSION` / `OBSOLETE` / `DUPLICATE`). The mapping rides in the journal's free-form `extras{}` under an ARL-owned key — **distinct from `pr-review-journal#6`'s `extras.outcome`** — so the journal gains no dependency on ARL (ADR-001's `controller → ledger` invariant is preserved: ARL writes the mapping *into* the ledger; the ledger never reads ARL). Example: "Narrow Fix" → `ACCEPTED`; "Class-fix" → `ACCEPTED_MODIFIED`; "Reject as Stale" → `OBSOLETE` / `REJECTED_FALSE_POSITIVE`. (Final mapping table to be ratified with pr-review-journal.)

## Alternatives considered

- **Let ARL infer intent from the PR body / `/goal`'s prose handoff.** Rejected: PR bodies are untrusted (injection), and prose handoffs are the documented multi-agent failure (specification + inter-agent misalignment, MASFT arXiv 2503.13657). The contract must be typed states.
- **Put the state machine inside ARL or cbm.** Rejected: it is a bidirectional seam (ARL constrains cbm's slice size; cbm's merge resumes ARL). A seam owned by one end re-creates the asymmetric-coupling problem ADR-001 fixed.
- **A swarm of converger agents.** Rejected: naive multi-agent collaboration lowers success ~30% (curse of coordination, arXiv 2601.13295); keep the boundary to exactly two agents (drive + converge) with a clean typed contract (CAID arXiv 2603.21489).

## Consequences

**Positive:** the converger reasons over states, not reviewer prose (handles silent rejection); the slice-size constraint and the merge-resume signal become explicit and testable; reuses the ADR-001 schema-as-contract pattern; no new repo.

**Negative:** introduces a new shared artifact (the state machine) that both cbm and ARL must conform to — a coordination cost; the ARL-posture→verdict mapping is a second contract to maintain.

**Neutral:** gated behind ARL P4/P5 (the verdict brain + `pr-watch` event source, `agentic-review-loop#15`) and the pr-review-journal outcome-linkage work (`pr-review-journal#6`). This ADR is a specification; nothing is built by adopting it.
