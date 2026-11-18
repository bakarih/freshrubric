---
paths:
  - "pyproject.toml"
  - "uv.lock"
---

# Changing dependencies or project config

- **A new runtime dependency is a planning decision**, and a load-bearing one also needs an ADR (AGENTS.md). Don't add one inline.
- **Use `uv add` / `uv add --dev`.** Never hand-edit `uv.lock`; CI runs `uv sync --locked` and fails on a stale lock.
- **Dev tools go in the `dev` dependency group** (ADR-0003e).
- **The Python pin (`==3.13.*`) follows the Functions runtime** (ADR-0004). Changing it is a deliberate, planned version bump.
- Lint, type-check and test settings here implement ADR-0003c; changing the rule set is a planning decision.
