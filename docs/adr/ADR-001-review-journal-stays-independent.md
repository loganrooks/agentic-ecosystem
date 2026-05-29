# ADR-001: `pr-review-journal` stays independent; consumers integrate via the verdict-schema contract

Status: accepted
Date: 2026-05-29

## Context

`agentic-review-loop` (ARL) and `pr-review-journal` were built in parallel
and, as of this decision, neither repo's framing docs reference the other —
yet they operate on the same surface. ARL's roadmap (P3 pre-triage pack, P4
supervisor-pr-triage skill) is building "dispose of each reviewer finding
with a documented verdict." `pr-review-journal` already ships exactly that:
the `pr-review-triage` skill plus a parseable verdict schema and a 35-test
suite, as `v0.1.0`.

This forces a boundary decision: should the finding-disposition capability be
**folded into** ARL (or `agentic-ops`), or should `pr-review-journal` remain
an **independent** component that consumers integrate?

Three facts constrain the answer:

1. **The journal already has multiple, independent consumers.** It was
   extracted from `tap-n-filter` (outside this family) and is used in
   `erebus`. Human responders in unrelated repos already depend on it. ARL
   would be one *additional* consumer, not the owner.
2. **The dependency is asymmetric.** A convergence loop needs a verdict
   discipline to function; a verdict ledger is fully useful with no loop at
   all (a human triaging CodeRabbit findings wants the audit trail). The
   general, shipped component (journal) must not depend on the specific,
   pre-1.0 one (ARL).
3. **The journal is observability, not control.** Per its own framing it
   *records* the reviewer↔responder interaction. It is the ledger; ARL is one
   controller that writes to it. You do not fold a logging library into one
   of the services that logs to it.

## Decision

`pr-review-journal` **stays an independent repo and product.** It owns the
**verdict schema** and the issue/dispose discipline (the `pr-reviewer` and
`pr-review-triage` skills) for the reviewer↔responder interaction, for any
reviewer and any repo.

`agentic-review-loop` **consumes** `pr-review-journal`:

- ARL adopts the journal's verdict schema as its loop's disposition output
  format. The **schema is the integration contract.**
- ARL treats `pr-review-triage` as the **canonical** triage discipline and
  **does not reimplement it.** ARL adds only what is genuinely loop-specific:
  when to escalate (dormancy contract), when to merge (gating preconditions),
  and cross-vendor adjudication.
- Integration mechanism is "install both; ARL references the journal's
  tool/skill." The coupling is the data contract, not a code-level hard
  dependency — the loosest coupling that still composes, and one that does
  not require plugin-depends-on-plugin support.

The dependency direction is fixed: **controller → ledger, never the
reverse.** `pr-review-journal` must never take a dependency on ARL or on
`agentic-ops`.

This resolves the prior `agentic-review-loop` open question on whether P4
should consume vs. reimplement triage: **consume.**

## Alternatives considered

- **Fold `pr-review-journal` into `agentic-review-loop`.** Rejected: it would
  strip a property the journal already has and other repos already rely on
  (independent use in `tap-n-filter`, `erebus`), and would invert the
  stable-dependency direction (general/shipped depending on specific/pre-1.0).
- **Fold it into `agentic-ops`.** Rejected: `agentic-ops` is a finding
  *producer*; the journal sits on the *responder* side recording verdicts on
  *all* reviewers (including `agentic-ops`'s own output). Coupling it to one
  producer breaks its vendor-agnostic premise.
- **Merge ARL and `pr-review-journal` into one repo.** Rejected against the
  couple-vs-compose test below: the journal has standalone value, ships on
  its own cadence, and has multiple consumers — it fails all three coupling
  conditions.
- **Let ARL reimplement triage independently.** Rejected: two divergent
  copies of the same verdict discipline across two repos the same maintainer
  owns — guaranteed drift, no upside.

## Consequences

**Positive:**
- ARL's P4 scope shrinks to genuinely loop-specific concerns; the verdict
  discipline is reused, not rebuilt.
- `pr-review-journal` keeps its clean, vendor-agnostic, portable identity and
  its existing non-family consumers.
- The seam is a versioned data contract — inspectable, testable, and
  decoupled from plugin-dependency mechanics.

**Negative:**
- The verdict schema becomes a shared contract that can drift; it needs an
  owner (the journal) and a versioning discipline. Schema changes ARL wants
  must be proposed upstream rather than made locally.
- Consumers must install/reference two artifacts instead of one.

**Neutral:**
- Establishes the family's general rule: **couple/merge only when all three
  hold — (1) the component has no value without its host, (2) shared release
  cadence, (3) exactly one consumer.** Used here to keep the journal
  independent; the same test recommends *grouping* `agentic-handbook` +
  `agentic-trellis` (see [OQ-002](../../OPEN_QUESTIONS.md)).
