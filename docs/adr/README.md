# Architecture Decision Records

Each ADR records one significant decision (or a batch of related sub-decisions) with its context and consequences. Format: Status / Context / Decision / Consequences.

| ADR | Title | Status |
|---|---|---|
| [0001](0001-reality-grounding-layer.md) | Reality-grounding layer for SpectrumHire's generators | Accepted (2026-07-29; ADR-0001i added 2026-07-31); hosting plan amended by ADR-0004 |
| [0002](0002-drift-detection-agent-swarm.md) | Drift-detection agent swarm and human-reviewed feedback loop | Partially accepted (2026-08-26); ingestion mechanism deferred |
| [0003](0003-python-toolchain-and-conventions.md) | Python toolchain, code conventions, and CI shape | Accepted (2026-10-19) |
| [0004](0004-flex-consumption-hosting.md) | Host the Function App on the Flex Consumption plan | Accepted (2026-10-19) |

## When to write a new ADR

Write one when there's:

- a new load-bearing dependency
- a new Azure resource *type*
- a reversal of a prior ADR
- a real tradeoff pause
- anything where forgetting *why* would hurt later

Skip it for naming, version bumps, and routine files.
