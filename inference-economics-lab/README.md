# Inference Economics Lab

A dependency-free interactive website for learning and modelling:

- token-based API pricing;
- prompt caching and cache-write/read economics;
- long-conversation replay and compaction;
- retrieval, routing, batch/flex, and output-control strategies;
- subscription versus API economics;
- hypothetical inference COGS and provider economics;
- current provider pricing snapshots with official sources.

## Open

For the simplest single-file version, open `inference-economics-lab-standalone.html`. You can also double-click `index.html`, or serve the directory locally:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

No build step or network connection is required for the app itself. External links in the Sources section open official documentation and papers.

## Important scope

Pricing data is a snapshot dated 2026-07-13. The simulator is an explicit economic model, not an invoice calculator. Provider cache semantics and service-tier modifiers differ; all important tariff fields are editable.
