# The agentic-* ecosystem

A map of the `agentic-*` family: what each repo owns, what it deliberately
does *not* own (and who owns it instead), how the repos compose, and where
each sits on the maturity curve.

The map's one job is to prevent silent overlap. Each repo's own VISION /
ROADMAP / ADRs remain canonical for that repo's internals; this file is
canonical only for the **boundaries between** repos.

How to read an entry: **Owns** is the single concern the repo is allowed to
grow. **Does NOT own** names the adjacent concerns it must defer to a
sibling — these lines are the actual fences. If two repos ever claim the
same "Owns," that's a bug in the map (or in a repo), and it gets resolved
here.

---

## The family at a glance

| Repo | Owned concern | Cluster | Maturity |
|---|---|---|---|
| **agentic-ops** | AI-failure-mode review engine + onboarding + drift detection; family doctrine origin | A (review) | Shipping |
| **pr-review-journal** | Verdict ledger **+ reviewer-quality flywheel** (sycophancy/pushback detection + reviewer-design improvement) | A (review) | Shipping (ledger) |
| **agentic-review-loop** | Collapsing AI fix-fix-fix iteration chains to 1–2 rounds | A (review) | Bootstrap |
| **agentic-mail** | Async file-mediated agent↔agent / agent↔human messaging protocol | B (coordination) | Bootstrap |
| **agentic-handbook** | Portable, agent-consumable situational *practice* guidance | B (coordination) | Scoping |
| **agentic-trellis** | "Systems consultant" for running planner/executor agent workflows | B (coordination) | Seed |

Maturity tiers: **Shipping** (released, has consumers) · **Bootstrap**
(real code, pre-1.0, no external consumers) · **Scoping** (framing docs,
no code) · **Seed** (design capture only).

---

## Cluster A — the PR-review pipeline

Surface: a pull request's review threads. Three repos, one pipeline.
**Data flows** producers → ledger → controller. **Dependencies** run the
other way and downhill: the controller depends on the ledger; the ledger
depends on neither. Nothing general depends on anything specific.

```
  agentic-ops review.yml ┐
  CodeRabbit / Codex App ─┼─▶  pr-review-journal  ─▶  agentic-review-loop
       (produce findings) ┘     (records each finding   (reads ledger, decides
                                 + the responder's        escalate / merge,
                                 verdict; independent)     writes verdicts back)
                                        ▲
                                        └── also written to by HUMAN responders
                                            in unrelated repos (tap-n-filter,
                                            erebus). ARL is just one more consumer.
```

> **Longitudinal view:** beyond the per-PR flow above, the accumulated
> verdict trail *is* the **reviewer-quality flywheel** (see the
> `pr-review-journal` entry below) — it improves the responder (sycophancy
> vs. warranted pushback) and the reviewers (design lessons) over time.

### `agentic-ops` — the hub + doctrine source

- **Owns:** the centralized devops platform for AI-led development —
  multi-specialist PR review tuned for AI failure modes, project onboarding,
  and drift detection. Also the *doctrine origin*: the ADR / VISION /
  OPEN_QUESTIONS discipline every other repo here inherited.
- **Does NOT own:** the coordination protocol (→ `agentic-mail`), the verdict
  ledger (→ `pr-review-journal`), the convergence loop (→ `agentic-review-loop`),
  or this ecosystem map (→ this repo). Per its own ADR-006, its deployment
  scope is bounded — it is a platform, not a registry.
- **Maturity:** Shipping. `v1` floating tag; live external consumer
  `codebase-mapper`; 10 ADRs.
- **Composition role:** finding **producer** (`review.yml`) and the source of
  shared discipline.
- **Key relationships:** consumes `agentic-mail` `v0.1.0` as an advisory
  channel; its review output is recorded by `pr-review-journal`; its
  no-auto-merge kernel rule is *scoped-inverted* by `agentic-review-loop`'s
  ADR-001.

### `pr-review-journal` — the reviewer-quality ledger + flywheel (independent shared infra)

- **Owns:** the measurement substrate and discipline of the reviewer↔responder
  interaction. Concretely: the parseable **verdict schema** ("recommendation →
  accepted/rejected/deferred + why"), the two discipline skills (`pr-reviewer`
  issue-side, `pr-review-triage` dispose-side), and — the larger purpose — the
  **reviewer-quality flywheel** that runs on the accumulated verdict trail:
    - *Responder-quality loop:* detect sycophancy (rubber-stamping reviewer
      findings to close threads) vs. warranted pushback in the responding agent.
    - *Reviewer-design loop:* use verdict history to improve reviewer-agent
      design — suppress consistently-rejected noise classes, close gap classes
      that let real defects through — so reviews raise PR/codebase quality.
  Vendor-agnostic by design ("reviewer behaviour as config, not code").
- **Does NOT own:** producing findings (→ any reviewer, incl. `agentic-ops`),
  deciding what to *do* with a PR — loop timing, escalation, merge
  (→ `agentic-review-loop`), or any reviewer *implementation*. The fence is
  **measure-vs-implement**: this repo owns the quality *measurement + design
  discipline*; reviewer implementations (`agentic-ops/review.yml`, CR / Codex
  configs, the `pr-reviewer` reference design) *consume* the lessons.
- **Maturity:** Shipping as a **ledger** (`v0.1.0`, installable plugin, 35-test
  suite). The **flywheel is latent, not built**: the data model reserves
  extension hooks ("metrics consumer", "learning system") but no aggregation or
  sycophancy-detection layer exists yet, and sycophancy detection further needs
  an outcome-linkage signal the schema does not carry (see
  [OQ-004](OPEN_QUESTIONS.md)). Born **outside** this family (extracted from
  `tap-n-filter`); also used in `erebus`.
- **Composition role:** the independent ledger every responder writes to, and
  the measurement substrate that improves every reviewer over time.
- **Key relationships:** works with `agentic-ops/review.yml` as one finding
  source; [ADR-001](docs/adr/ADR-001-review-journal-stays-independent.md)
  records that it **stays independent** and that `agentic-review-loop`
  **consumes** it via the verdict-schema contract rather than reimplementing
  triage.

### `agentic-review-loop` — the convergence controller

- **Owns:** reducing AI "fix-fix-fix" iteration chains (4–12 rounds → 1–2) via
  a pre-triage layer, a governance-invariants linter, and a
  supervisor↔implementer loop.
- **Does NOT own:** the verdict discipline (→ consumes `pr-review-journal`),
  or producing findings (→ consumes reviewers). It adds only what is
  genuinely loop-specific: when to escalate (dormancy contract), when to
  merge (gating preconditions), cross-vendor adjudication.
- **Maturity:** Bootstrap (P0 done, P1 next; nothing installable yet).
- **Composition role:** the automated **responder/controller** — reads the
  ledger, decides escalate/merge, writes verdicts back through the journal's
  schema.
- **Key relationships:** consumes `pr-review-journal` (ADR-001); its own
  ADR-001 is a *scoped* inversion of `agentic-ops`'s no-auto-merge rule;
  targets `agentic-ops` / `montage_cli` / `agentic-mail` as install
  consumers. **Open:** is it a standalone product or `agentic-ops`'s loop
  module? → [OQ-001](OPEN_QUESTIONS.md).

---

## Cluster B — the coordination / practice substrate

Surface: how agents and humans coordinate, and how practice knowledge
reaches an agent. `agentic-mail` is the hub; the other two spawned out of
its sessions and orbit it.

```
  agentic-handbook ─┐  (consult-for-guidance;
                    ├──  spawned-from)        agentic-mail   ── consumed by
  agentic-trellis ──┘                         (transport hub)   agentic-ops
                                                    │            (dogfood install)
                                                    └── future: agentic-review-loop
                                                        escalation transport (P4)
```

### `agentic-mail` — the coordination transport

- **Owns:** an async, file-mediated messaging protocol between AI agents
  (and humans) sharing a workspace — the substrate for surfacing uncertainty
  agent-to-agent and agent-to-human at session boundaries.
- **Does NOT own:** review ("what does AI think of human code" → `agentic-ops`),
  or blocking human escalation (the channel is *advisory*; blocking
  escalation stays with the consuming repo's discipline).
- **Maturity:** Bootstrap (`v0.1.0`, experimental; design docs are the
  load-bearing artifact).
- **Composition role:** transport substrate; hub of cluster B.
- **Key relationships:** consumed by `agentic-ops` (dogfood install); its
  ADR-001 explicitly chose a separate repo *over* living inside `agentic-ops`
  to avoid conflating ownership; content/pattern precedent for both
  `agentic-handbook` and `agentic-trellis`. Candidate transport for
  `agentic-review-loop`'s escalation path at P4.

### `agentic-handbook` — portable practice guidance

- **Owns:** a portable, agent-consumable corpus of *situational* engineering
  practice guidance — designed to follow a developer across projects.
- **Does NOT own:** project-specific rules (CLAUDE.md / .cursorrules),
  library reference docs (Context7-style), or agent orchestration.
- **Maturity:** Scoping (framing docs only; no units authored, no code).
- **Composition role:** knowledge layer; orthogonal to both pipelines (an
  agent *consults* it, nothing depends on it at runtime).
- **Key relationships:** parallel to `agentic-mail` (its first content
  source), explicitly "not a spinoff." Emerged alongside `agentic-trellis`;
  whether the two should be one repo is → [OQ-002](OPEN_QUESTIONS.md).

### `agentic-trellis` — the agent-workflow consultant

- **Owns:** a "systems consultant" (industrial-engineering pattern) for
  operators running Claude-as-planner / Codex-as-executor workflows —
  guardrails, failure registries, supervisor-role specs, onboarding
  discipline.
- **Does NOT own:** being a product yet — it is explicitly design-only.
- **Maturity:** Seed (design capture; zero implementation, no `.claude-plugin/`).
- **Composition role:** meta-advisory layer over how the *other* repos are
  operated.
- **Key relationships:** derived from `agentic-mail` sessions (N=1.5:
  `agentic-mail` fully, `agentic-ops` partially); default comms adapter is
  the `agentic-mail` protocol. Emerged alongside `agentic-handbook` →
  [OQ-002](OPEN_QUESTIONS.md).

---

## Why a neutral home (not `agentic-ops`)

The obvious instinct is to put this map inside `agentic-ops` — it's the most
mature repo and the doctrine origin. We deliberately don't, for two reasons
grounded in the family's own precedent:

1. **The family has twice rejected `agentic-ops`-centrality.** `agentic-mail`'s
   ADR-001 refused to live inside `agentic-ops` to avoid "conflating
   ownership"; `agentic-handbook` wrote an explicit "parallel, not a spinoff"
   defense. Housing the family charter inside the biggest sibling re-asserts
   exactly the parent/child hierarchy those repos pushed back on.
2. **`agentic-ops` has bounded scope by its own ADR-006.** It is a devops
   platform, not an ecosystem registry. The map describes a family of *peers*
   from outside; it should not be one peer's responsibility.

Discoverability (the real risk of a neutral repo nobody opens) is handled by
**pointers, not location**: each family repo's README should carry a one-line
"where this sits in the family → `agentic-ecosystem/ECOSYSTEM.md`." Then the
map is reachable from wherever work happens, while its canonical copy stays
neutral.

---

## Not part of this ecosystem

- **`agentic-research-orchestrator`** — a repo-agnostic research run-bundle
  control plane. It predates this family, references none of these repos, is
  referenced by none, and shares no doctrine. It is a separate (currently
  dormant) project and is intentionally excluded from this map. Recorded here
  only so future readers don't mistake its absence for an oversight.

---

## How this file evolves

- Update an entry's **Maturity** as a repo ships or stalls.
- When a boundary decision is **made**, record it as an ADR in
  [`docs/adr/`](docs/adr/) and link it from the affected entries.
- When a boundary tradeoff is **open**, record it in
  [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md), not in one repo's tree.
- A new `agentic-*` repo does not exist, for ecosystem purposes, until it has
  an entry here naming a concern no existing repo owns (see
  [`AGENTS.md`](AGENTS.md) §"Creating a new family repo").
