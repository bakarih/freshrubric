# AGENTS.md

Rules for AI coding assistants (Claude Code, Copilot, Codex, and others) working in this repo. Recorded as [ADR-0001i](docs/adr/0001-reality-grounding-layer.md#adr-0001i-decision-process).

## Two modes, kept separate

**Decisions are made by the maintainer, in planning (Claude Code plan mode).** This covers:

1. Architecture: components, how they connect, which Azure resources exist
2. Coding style
3. Linting rules
4. Testing strategy
5. Features and milestone scope
6. CI/CD design
7. Documentation strategy

**Implementation is AI-assisted.** Once a decision is made, AI help writes the code and tests that carry it out, a few lines at a time. It executes decided designs and conventions. It does not choose them.

## When something isn't decided yet

If a new dependency, naming convention, test boundary, or other choice comes up that isn't covered by an ADR or an explicit instruction: **stop and raise it for a planning decision.** Do not decide it inline while coding.

Coding style, linting, and tooling are decided in [ADR-0003](docs/adr/0003-python-toolchain-and-conventions.md). Follow it. Anything it doesn't cover goes back to planning; do not pick defaults on your own.

## Ground rules

- **Secrets never get committed.** This is a public repo. Use `.env` (gitignored), `local.settings.json` (gitignored), GitHub Actions secrets, and Key Vault.
- **Don't re-open decided architecture** without a planning conversation. The decisions are in `docs/adr/`.
- **Keep the README honest.** Only describe something as built once it is in this repo.
- **Write an ADR** (`docs/adr/000N-title.md`, with Status / Context / Decision / Consequences) for any new load-bearing dependency, new Azure resource type, reversal of a prior ADR, or real tradeoff pause. Skip it for naming, version bumps, and routine files. Link each new ADR from `docs/adr/README.md`.
