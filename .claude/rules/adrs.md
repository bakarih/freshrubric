---
paths:
  - "docs/adr/**"
---

# Writing or changing an ADR

- **Format:** `docs/adr/000N-title.md` with Status / Context / Decision / Consequences, plus Date and Deciders. Lettered sub-decisions (`ADR-000Nx`) for batches, as in ADR-0001.
- **Write one for** a new load-bearing dependency, a new Azure resource type, a reversal of a prior ADR, or a real tradeoff pause. Skip it for naming, version bumps and routine files.
- **Link every new ADR** from [docs/adr/README.md](../../docs/adr/README.md).
- **Don't rewrite an accepted ADR.** A reversal is a new ADR; the old one gets a short "Amended by" note, as ADR-0004 did to ADR-0001.
- ADRs record Bakari's decisions. Draft one only for a decision he has made in planning.
