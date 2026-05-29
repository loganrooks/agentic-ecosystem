# agentic-ecosystem

> The map and governance layer for the `agentic-*` family of repositories.
> Not a product — the neutral place that records what each repo owns, how
> they compose, and the cross-repo decisions no single repo can own.

## What this is

A family of independently installable, composable AI-development tools has
grown up under the `agentic-*` name: `agentic-ops`, `pr-review-journal`,
`agentic-review-loop`, `agentic-mail`, `agentic-handbook`, and
`agentic-trellis`. Each has rigorous *internal* discipline — VISION,
ROADMAP, ADRs, OPEN_QUESTIONS. None of them sits *above* the others to
record how they fit together. This repo fills that gap.

It exists because the absence of an above-the-repos map has a concrete,
already-observed cost: `agentic-review-loop` and `pr-review-journal` were
both built to dispose of reviewer findings, and neither repo's docs
referenced the other. A family of peers needs one shared map, or the peers
silently overlap.

## What this is NOT

- **Not a product.** Nothing here installs or runs. It is documentation and
  decisions.
- **Not a parent repo.** The family members are peers. This repo *describes*
  them; it does not own or subordinate them. (See [`ECOSYSTEM.md`](ECOSYSTEM.md)
  §"Why a neutral home" for why this map deliberately does not live inside
  `agentic-ops`.)
- **Not a monorepo.** The code stays in its own repos, each independently
  versioned and installable. This is the index, not the container.

## Contents

| File | Purpose |
|---|---|
| [`ECOSYSTEM.md`](ECOSYSTEM.md) | The map: each repo's owned concern, non-goals, maturity, and the composition DAG |
| [`docs/adr/`](docs/adr/) | Cross-repo decisions that have been **made** (ADR-001: the review-journal stays independent) |
| [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) | Cross-repo boundary decisions still **open** (e.g. is `agentic-review-loop` a product or a module?) |
| [`AGENTS.md`](AGENTS.md) | Discipline for editing this repo, including the rule for when a new family repo may be created |

## How to use it

- **Starting work in a family repo?** Read its entry in `ECOSYSTEM.md` first —
  know what it owns and, just as important, what it must *not* grow into.
- **Tempted to create a new repo?** Apply the "two-project signal" rule in
  [`AGENTS.md`](AGENTS.md) before you do.
- **Made a decision that spans two repos?** It belongs here — an ADR if
  decided, an OPEN_QUESTIONS entry if not — not buried in one repo's tree
  where the other repo will never see it.

## Scope boundary

This repo covers the `agentic-*` family that shares the `agentic-ops`
lineage and design doctrine. `agentic-research-orchestrator` is **not** part
of this ecosystem — it predates the family and shares neither dependencies
nor doctrine (see `ECOSYSTEM.md`).
