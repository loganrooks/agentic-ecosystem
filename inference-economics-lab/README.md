# Inference Economics Lab

A dependency-free interactive website for learning and modelling:

- token-based API pricing;
- prompt caching and cache-write/read economics;
- long-conversation replay and compaction;
- retrieval, routing, batch/flex, and output-control strategies;
- subscription versus API economics;
- hypothetical inference COGS and provider economics;
- current provider pricing snapshots with official sources.

## Live site

- Development CDN: https://raw.githack.com/loganrooks/agentic-ecosystem/inference-economics-lab-site/inference-economics-lab/index.html
- Immutable snapshot: https://rawcdn.githack.com/loganrooks/agentic-ecosystem/6257069a63b9304e5b8ff7ad74dfe99ca0155855/inference-economics-lab/index.html

The publication branch uses a small same-origin loader that reconstructs the tested standalone HTML from twelve payload parts. This works around connector file-size limits without changing the application code.

## Local source package

The original package contains both a multi-file build and a directly openable standalone HTML file. No build step or network connection is required for the app itself. External links in the Sources section open official documentation and papers.

## Important scope

Pricing data is a snapshot dated 2026-07-13. The simulator is an explicit economic model, not an invoice calculator. Provider cache semantics and service-tier modifiers differ; all important tariff fields are editable.
