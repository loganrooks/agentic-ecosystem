# AGENTS.md — agentic-ecosystem operative discipline

This file is read by AI agents (and humans) on every contribution to
`agentic-ecosystem`. It is short because this repo is small and its job is
narrow.

## Scope

This is the **governance and map layer** for the `agentic-*` family. It
contains documentation and decisions only — no product code, no install
surface. Strategic content is [`ECOSYSTEM.md`](ECOSYSTEM.md); decisions are
in [`docs/adr/`](docs/adr/) and [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md).

This repo describes a family of **peers**. Nothing here may be written as if
this repo, or any one family repo, owns or commands the others.

Before contributing, read:

- This file
- [`ECOSYSTEM.md`](ECOSYSTEM.md) — the map you are maintaining
- [`docs/adr/README.md`](docs/adr/README.md) — ADR format (defers to the
  family convention in `agentic-ops`)

## Where things go

- **A decision that spans ≥2 repos, now made** → a cross-repo ADR in
  `docs/adr/`.
- **A boundary tradeoff that spans ≥2 repos, still open** → an `OQ-NNN` in
  `OPEN_QUESTIONS.md`.
- **A change to what a repo owns / its maturity** → update its entry in
  `ECOSYSTEM.md`.
- **Anything internal to a single repo** → does NOT belong here. It goes in
  that repo's own tree.

## Creating a new family repo

A new `agentic-*` repo is a high-cost, low-reversibility act (repos are cheap
to create, costly to maintain, very costly to merge later). Before creating
one, both must hold:

1. **The two-project signal.** The concern has shown up as a real need in at
   least two distinct contexts — not one session's enthusiasm. (Borrowed from
   `agentic-handbook`'s own unit-admission rule; applied here to repos.)
2. **No existing repo can own it.** Name the concern and check it against
   every `Owns` line in `ECOSYSTEM.md`. If an existing repo could grow to
   cover it without breaking its identity, extend that repo instead.

A new repo is not real, for ecosystem purposes, until it has an
`ECOSYSTEM.md` entry stating the single concern it owns that no existing repo
does.

This governance/index repo is itself exempt from rule (1) — its purpose *is*
to bound family sprawl, which is categorically different from being another
product.

## Discipline

- Keep the map honest: if two repos' `Owns` lines overlap, that is a defect —
  fix it here (or open an `OQ`), don't let it sit.
- ADR bodies are immutable once `accepted`; revise via a new ADR.
- Never push to `main` directly once branch protection is configured; change
  via PR. (This repo follows the family's merge discipline.)
