# Cross-repo Architecture Decision Records

This directory holds ADRs for decisions that span **two or more** repos in
the `agentic-*` family — boundary calls that no single repo can own because
they govern the seam between repos.

Decisions *internal* to a repo live in that repo's own `docs/adr/`. A
decision belongs **here** only if changing it would require edits in more
than one repo, or if it assigns ownership of a concern across repos.

## Format

Same convention as the rest of the family (see
[`agentic-ops/docs/adr/README.md`](https://github.com/loganrooks/agentic-ops/blob/main/docs/adr/README.md)
for the full status-value and amendment rules):

```markdown
# ADR-NNN: <decision title>

Status: <proposed | accepted | accepted (provisional) | deprecated; [relationship to ADR-MMM]>
Date: YYYY-MM-DD

## Context        — what forced this decision; what constraints were active
## Decision       — what was chosen
## Alternatives considered — what else was evaluated, and why rejected
## Consequences   — Positive / Negative / Neutral
```

- Filenames: `ADR-NNN-kebab-case-title.md`, `NNN` zero-padded to three digits.
- Numbers are sequential and never reused.
- Bodies are immutable once `accepted`; revise via a new (superseding or
  amending) ADR. The `Status:` line is the one exception.

## Index

| ADR | Title | Status |
|---|---|---|
| [001](ADR-001-review-journal-stays-independent.md) | `pr-review-journal` stays independent; consumers integrate via the verdict-schema contract | accepted |
