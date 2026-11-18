---
paths:
  - "src/**/*.py"
---

# Writing code in `src/`

- **Follow ADR-0003b.** Full type annotations (`mypy --strict` must pass), Google-style docstrings on public modules, classes and functions, absolute imports only, timezone-aware datetimes only. See [ADR-0003](../../docs/adr/0003-python-toolchain-and-conventions.md).
- **Only sanctioned data sources.** Job postings come from the public Greenhouse, Lever and Ashby job board APIs (ADR-0001a). No scraping of LinkedIn, Indeed or Glassdoor.
- **The careers-page fetch has guardrails that are requirements, not options** (ADR-0001h): check `robots.txt` before every fetch, rate-limit per domain, send an identifying `User-Agent` that links to this repo, fetch only companies in the seed list, and fall back to the posting's "About" section when a page can't be fetched.
- **No new dependency or subpackage name inline.** Raise it for planning (AGENTS.md).
- Before you finish: `uv run ruff check`, `uv run ruff format`, `uv run mypy`.
