# ADR-003: Wire cbm's ADR-005 cross-model checkpoint into ARL's merge gate as the loop's no-skimp guarantee

Status: proposed
Date: 2026-05-29

> Cross-repo ADR. Governs how the drive-to-maturity loop guarantees it drives cbm to H5 *without skimping phases*. Spans **codebase-mapper/cbm** (its ADR-005 checkpoint + HORIZONS acceptance gates) and **agentic-review-loop / ARL** (merge gating, `agentic-review-loop#15`). Companion to [ADR-002](ADR-002-drive-converge-handoff-contract.md). Derived from the AOR lab design (`agentic-family-design/01-autonomous-drive-to-maturity-loop.md` §3 S3).
>
> **Naming note:** "ADR-005" here is **cbm's** decision (`codebase-mapper/.planning/decisions/ADR-005-cross-model-checkpoint-mandatory-for-pass-claims.md`, accepted 2026-05-02), *not* an agentic-ecosystem ADR. This ecosystem ADR (003) wires that cbm decision into the cross-repo merge seam.

## Context

Quality degrades **monotonically across iterations unless something structural forbids it** — and *prompting cannot stop it*: SlopCodeBench (arXiv 2603.24755, T3) finds erosion in 80% of trajectories where an agent extends its own prior work under evolving specs (exactly the H2→H5 regime), and shows a prompt-intervention "does not halt degradation." Self-certification is actively gamed: ImpossibleBench (arXiv 2510.20270, T3) measures cheating at 54–89%, dropping to **~0% when the grader is blind to the tests** and **54%→9% when an honest-abort option exists**. Gate-gaming grows with horizon length (SpecBench arXiv 2605.21384, T3: held-out gap +~28pp per 10× code size).

cbm already ships the mechanism: **cbm's ADR-005** enforces `SCOPES_REQUIRING_CROSS_MODEL = {pass-claim, main-merge, broad-goal-restart}` in `cbm/cli.py`, so `cbm-loop-status --scope pass-claim` exits 0 only when artifact-grounded acceptance is met, a checkpoint exists at scope, its `reviewer_model_id` is a **different model family** than the dev agent (mechanically enforced), disposition is `accept`, and no review session is unfinished. **Hallucinating "done" does not move the exit code.** Two honesty constraints must be recorded so the loop does not overclaim this guarantee.

## Decision

1. **The no-skimp guarantee is the composition of cbm's non-bypassable HORIZONS acceptance gates + cbm's ADR-005 cross-model pass-claim checkpoint, made structural and wired into ARL's converge/merge step.** ARL's autonomous-merge (its own ADR-001) for a **horizon/pass-claim transition** is conditioned on `cbm-loop-status --scope pass-claim == 0`, **not on visible-CI-green alone.** The model that *claims* a horizon done is structurally never the one that *confirms* it (actor ≠ verifier, applied to the completion claim).

2. **Scope the gate honestly (do not overclaim).** cbm's ADR-005 fires only at `{pass-claim, main-merge, broad-goal-restart}` — **horizon/pass-claim transitions, NOT every stage-slice PR.** The bulk of a multi-day drive is intra-horizon slice PRs merging on ordinary acceptance + whatever the bot cascade supplies (which may be same-vendor). This is acceptable because (a) intra-stage acceptance is itself a harness fact (`cbm-loop-status` per stage), and (b) the stronger cross-model gate backstops every horizon boundary. **"Horizon-completion is gated" must never be read as "every PR is cross-model gated."**

3. **Build item (conditional):** if push-heavy, same-vendor-only stage merges become a measured risk, **require an orthogonal-provider reviewer at each merge** — extend `SCOPES_REQUIRING_CROSS_MODEL` in cbm, or make ARL's bot cascade enforce vendor-orthogonality per merge. Flagged, not assumed away.

4. **The honest-abort path is mandatory and is a discrete deliverable** (`agentic-review-loop#15`, build step 5.4b): the engine may emit "I cannot complete this stage honestly → park + escalate" instead of manufacturing a pass. Without it the loop manufactures false completions (ImpossibleBench 54%→9%). It is framed as an *extension* of ARL's existing escalation/dormancy contract (its ADR-001 step 8), not a parallel mechanism.

## Alternatives considered

- **Prompt the agent to "not skip phases / verify before done."** Rejected: SlopCodeBench shows prompting does not halt degradation; self-certification is gamed (ImpossibleBench).
- **Gate every merge on cross-model review.** Deferred (build item #3), not default: it is expensive and the per-stage acceptance gate + per-horizon cross-model backstop is the cheaper structural floor. Adopt per-merge orthogonality only if measured drift warrants.
- **An aggregate "maturity score" gate.** Rejected: cbm's anti-vision forbids aggregate scores; a single trajectory-health number is the most Goodhartable object (gaming grows with horizon).

## Consequences

**Positive:** premature/hallucinated horizon-completion cannot move the gate; the guarantee is structural, not advisory; doubles as the over-suppression / self-rubber-stamp mitigation (the grader is a different model family). The over-suppression *falsifier* is recorded independently via `pr-review-journal#6`'s `extras.outcome` (`CONTRADICTED × REJECTED_*` = unwarranted dismissal).

**Negative:** intra-stage PRs are only acceptance-gated (the honest scope limit, #2) — a measured residual the loop must monitor; cbm H4's isomorphic refresh-verifier (the strongest blast-radius guard) is *pending*, so until H4 ships this guard is designed, not fully delivered. ARL's auto-merge (ADR-001) is also not blast-radius-gated today — the high-risk-merge detector is an open requirement in `agentic-review-loop#15`.

**Neutral:** gated behind ARL wiring the gate into its merge step (P-stage work) and cbm H4. Adopting this ADR specifies the wiring; it does not build it. It surfaces the tension with the "human last-word lifts acceptance" finding (arXiv 2603.15911) — resolved by treating the cross-model independent reviewer as the structural stand-in for human-last-word, with high-risk merges restoring an actual human via escalation (the autonomy boundary, [OQ-005](../../OPEN_QUESTIONS.md)).
