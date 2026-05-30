# Open questions (cross-repo)

Unresolved boundary decisions that span two or more `agentic-*` repos. ADRs
in [`docs/adr/`](docs/adr/) record cross-repo decisions *made*; this file
records the ones still *pending*.

Entries get an `OQ-NNN` ID for citation. A question belongs here only if it
governs a seam *between* repos; questions internal to one repo live in that
repo's own `OPEN_QUESTIONS.md`.

## OQ-001 — Is `agentic-review-loop` a product, or `agentic-ops`'s module?

**Status:** open. Surfaced 2026-05-29 during the ecosystem mapping pass.

[ADR-001](docs/adr/ADR-001-review-journal-stays-independent.md) settled that
ARL *consumes* `pr-review-journal`. It did not settle ARL's own status. ARL
is a scoped inversion of `agentic-ops`'s no-auto-merge kernel rule and
targets `agentic-ops` as an install consumer — which raises the question of
whether ARL is a standalone product or is really `agentic-ops`'s
"convergence loop" module that happens to live in its own repo.

Arguments for **standalone**: it has a distinct, well-researched problem
(iteration-chain collapse), its own roadmap to v1.0, its own install path,
and a deliberately *different* merge policy than `agentic-ops` (autonomous vs.
human-gated) — co-locating them would force one merge discipline on both.

Arguments for **module of `agentic-ops`**: both operate on the same PR-review
surface; `agentic-ops` already owns "review"; a solo maintainer carries two
roadmaps, two CI setups, two release cadences for what may be one product's
two layers.

**What would change the answer:** whether ARL acquires a consumer *other*
than `agentic-ops` (→ standalone); whether its autonomous-merge policy proves
to need a genuinely separate governance surface from `agentic-ops`'s (→
standalone); or whether, after P4 ships, its runtime turns out to be
inseparable from `agentic-ops`'s review invocation (→ module). Defer until
ARL reaches P4 (first real supervisor-loop deployment) — the empirical usage
pattern decides it.

## OQ-002 — Should `agentic-handbook` and `agentic-trellis` consolidate?

**Status:** open. Surfaced 2026-05-29 during the ecosystem mapping pass.

The two emerged together, from the same `agentic-mail` supervisor sessions.
Both are pre-code (Scoping / Seed). Their concerns are adjacent: the handbook
is situational practice guidance delivered *to* an agent; trellis is
consultant guidance *for running* planner/executor agent workflows. Under the
couple-vs-compose test in
[ADR-001](docs/adr/ADR-001-review-journal-stays-independent.md), they *pass*
all three coupling conditions relative to each other — neither ships, same
cadence, same origin — which argues for one repo, not two.

The counter-argument is identity: the handbook explicitly positioned itself
as "parallel, not a spinoff," and a portable cross-project practice corpus is
a genuinely different artifact from an operations consultant. Merging risks
muddying two clean visions before either has shipped a line of code.

**What would change the answer:** whichever ships first and proves its own
distribution mechanism establishes the boundary; if *neither* moves, the
default is to consolidate into one "agent-practice" repo rather than maintain
two empty scaffolds. Decide before *either* writes implementation code — the
merge is cheap now and expensive once both have structure.

## OQ-003 — Shared scaffolding template across the family?

**Status:** open. Low priority. Surfaced 2026-05-29.

Every family repo independently reproduces the same scaffold (VISION /
ROADMAP / OPEN_QUESTIONS / `docs/adr/README.md` with the full status-value
essay / AGENTS.md). That is duplicated discipline that drifts. Candidate: a
shared template (a `create-agentic-repo` skill, or a template repo, or a
documented checklist here) so the conventions have one source.

**What would change the answer:** the number of future family repos. At the
current count the duplication is tolerable; if the family keeps growing, the
DRY case strengthens. No action until a 7th repo is genuinely warranted.

## OQ-004 — Where does the reviewer-quality flywheel live, and what does sycophancy detection require?

**Status:** open. Surfaced 2026-05-29.

[ADR-001](docs/adr/ADR-001-review-journal-stays-independent.md) and the
expanded `pr-review-journal` entry in [`ECOSYSTEM.md`](ECOSYSTEM.md) establish
that the journal's purpose is not just a per-PR verdict ledger but a
**reviewer-quality flywheel**: the accumulated verdict trail is the dataset for
(1) detecting sycophancy vs. warranted pushback in the responding agent, and
(2) improving reviewer-agent design (suppress noise classes, close gap classes).
Two things are unresolved.

**(1) Engine location.** Does the aggregation + learning layer — the flywheel
*engine* — live **inside** `pr-review-journal`, or as a **separate consumer**
that attaches to its verdict records? The journal owns both discipline skills
and the data, which pulls *inside*; but its own Unix-philosophy framing
("anything that makes the protocol smarter is suspect") and its data model —
which reserves extension hooks for an external "metrics consumer" and "learning
system" rather than building them in — pull toward *separate*. If separate,
that consumer may itself warrant a named concern (and, by the
two-project-signal rule in [`AGENTS.md`](AGENTS.md), eventually a repo).

**(2) Sycophancy needs an outcome signal the schema lacks.** Accept-rate cannot
distinguish "agreed because correct" from "agreed to be agreeable." The flywheel
needs **outcome-linkage** per verdict: was an *accepted* finding later reverted
or did it regress? was a *rejected* finding later vindicated or re-opened? The
current verdict schema carries no such field. What is the minimal mechanism
(post-merge revert/bug tracking; periodic ground-truth audit of a sample), and
does it belong in the schema or the engine? This is the ground-truth-anchoring
discipline (from the AgenticOps failure-mode research) applied to the responder.

**What would change the answer:** a first real batch of accumulated verdicts
(does longitudinal analysis need a separate service, or is a query over the
existing journal enough?); whether `agentic-ops` or another reviewer
implementation needs flywheel output in-band vs. as periodic reports. Decide the
outcome-linkage field *before* building analytics — it is cheap to add to the
schema now and expensive to backfill later.

## OQ-005 — What is the autonomy boundary for the drive-to-maturity loop, and does running it resolve OQ-001?

**Status:** open. Surfaced 2026-05-29 during the AOR loop-design pass.

For the autonomous loop driving `codebase-mapper`/cbm H2→H5 under a **full-auto,
minimize-involvement** posture (see [ADR-002](docs/adr/ADR-002-drive-converge-handoff-contract.md)
and [ADR-003](docs/adr/ADR-003-cross-model-gate-no-skimp-guarantee.md)): **what
does the loop decide alone vs. MUST escalate, and how is that boundary calibrated
without either over-asking (flooding the human) or under-asking (silent
over-suppression)?**

This is cross-repo: the boundary spans cbm (`/goal` raises escalations;
`vision_ambiguity` stop), ARL (autonomous-merge authority per *its* ADR-001 —
scope-inverted vs. `agentic-ops`'s human-gated rule; see
`agentic-review-loop#15`), `agentic-mail`/escalation transport, and the
maintainer. **It extends [OQ-001](#oq-001--is-agentic-review-loop-a-product-or-agentic-opss-module):**
ARL's autonomous-merge posture *is* the live experiment that calibrates this, so
OQ-005 and OQ-001 resolve together.

What the research says, so the options are grounded:

- Selective escalation is a **model-level deficit**: HiL-Bench (arXiv 2604.09408,
  T3, COI: Scale.AI) — full-info pass@3 75–89% collapses to 4–24% once the agent
  must *judge* whether to ask. Self-report is unreliable → the boundary must key on
  **structural, harness-detectable triggers**, with agent self-uncertainty a
  secondary signal only.
- "Minimize involvement" is formally an **Ask-F1 precision/recall** tradeoff
  (question-spam is penalized: 80% recall via 50 questions = 14.5% score) —
  *escalate rarely but correctly*, not "ask less."
- Over-automation worsens oversight (the "responsibility vacuum") — the fix is
  **rare, high-signal, information-rich escalations** (carry the full triage
  trace), not a human gate everywhere (which re-creates the vacuum via
  rubber-stamping).

**Options.** (1) *Static deny-list + cost function (design-only), per-horizon
tightness* — MUST-escalate: vision/spec ambiguity, destructive/irreversible
(harness-detected), high-risk merge, repeated CI failure, reviewer disagreement,
stall-past-budget, high-stakes whitelist; gray-zone cost function `escalate if
(blast_radius × irreversibility) > θ OR confidence < τ OR trigger ∈ deny-list`;
tighter at H2, widening as cbm acceptance gates prove out. *(Leaning — the only
option grounded in harness-detectable signals.)* (2) *Learned deferral policy
(RL)* — rejected for now (no training loop; a learned policy on a pre-1.0 surface
inverts the stable-dependency direction). (3) *Human-gate every merge* — rejected
(re-creates the responsibility vacuum via rubber-stamping; defeats the loop's
purpose).

**The unresolved residual (carry forward, do not pretend solved):**
over-suppression of escalation is **invisible by construction** — there is no
internal signal for "I should have asked and didn't." The cross-model checkpoint
(ADR-003) catches pass-claim errors but **not** "the human had context the loop
never knew it lacked." The only external anchors are the in-flight
**outcome-linkage** field (`pr-review-journal#6`'s `extras.outcome`, where
`CONTRADICTED × REJECTED_*` is the over-suppression alarm — this is exactly the
falsifier [OQ-004](#oq-004--where-does-the-reviewer-quality-flywheel-live-and-what-does-sycophancy-detection-require)
reserves), the orthogonal-reviewer ensemble, and human escalation on disagreement.

**What would change the answer:** ARL P4-onward dogfooding on ARL's own PRs, then
the cbm drive, producing real **Ask-F1** data — resolve OQ-005 + OQ-001 together
*after* the first real supervisor-loop deployment, not before. Resolve by writing
a cross-repo ADR (the calibrated boundary) or deferring to `agentic-review-loop`'s
P4+ phase.

## How this file evolves

- Add `OQ-NNN` when a mapping pass or a PR surfaces a genuine *cross-repo*
  tradeoff not decided by an ADR.
- Resolve by either writing a cross-repo ADR (decision recorded) or folding
  the question into a repo's roadmap (deferred to a specific phase).
- Do not delete resolved entries; mark them `Status: resolved by ADR-NNN` or
  `Status: deferred to <repo>/<phase>`.
