# ADR-0003: Python toolchain, code conventions, and CI shape

- **Status:** Accepted
- **Date:** 2026-10-19
- **Deciders:** Bakari Holmes
- **Resolves:** the coding-style and linting decisions that [ADR-0001i](0001-reality-grounding-layer.md#adr-0001i-decision-process) left open until Milestone 0.

## Context

ADR-0001 fixed the architecture but deliberately left coding style and linting undecided until M0, so they would be chosen in planning rather than picked as defaults by AI-assisted coding. M0 also needs a CI skeleton that gates every PR from the first line of code, and a dependency manager that every later milestone builds on.

The Python version is tied to the Azure Functions hosting plan: Linux Consumption stops at Python 3.12. That choice is recorded separately in [ADR-0004](0004-flex-consumption-hosting.md).

## Decision

### ADR-0003a: Python version and layout

- **Python 3.13**, the version the Flex Consumption plan runs (ADR-0004). Pinned in `.python-version` and `requires-python`.
- **`src/` layout**, one package: `src/freshrubric/`. Tests run against the installed package, never against stray files in the working directory. Subpackage names are decided with the code that needs them, starting at M1.

### ADR-0003b: Code conventions

- **Typing:** fully annotated, `mypy --strict`, in both `src/` and `tests/`. Tests mostly need `-> None` and typed fixtures.
- **Docstrings:** Google style. Required on public modules, classes, and functions in `src/`; not required in tests.
- **Imports:** absolute only (`from freshrubric.x import y`); relative imports are banned. Sorted by ruff's isort rules.
- **Naming and layout:** PEP 8 naming, double quotes, 88-character lines (ruff's default).
- **Datetimes:** timezone-aware only. Posting timestamps and drift snapshots (ADR-0002) depend on it.

### ADR-0003c: Linting, formatting, and type checking

- **ruff** for both linting and formatting, replacing black, isort, and flake8.
- **Rule set:** a curated list rather than ruff's defaults or `ALL`:

  | Rules | Why |
  |---|---|
  | `E`, `W`, `F` | pycodestyle and pyflakes basics |
  | `I` | import sorting |
  | `B`, `SIM`, `C4`, `RUF` | likely bugs and simplifications |
  | `UP` | modern syntax for the pinned Python version |
  | `N` | PEP 8 naming |
  | `D` (Google convention) | docstrings |
  | `S` | bandit security checks: the code fetches from the network in a public repo |
  | `PT` | pytest style |
  | `TID` | bans relative imports |
  | `DTZ` | timezone-aware datetimes |

  Tests ignore `S101` (assert) and `D`.
- **mypy `--strict`** as the type checker. It's pure Python, so it installs through uv without a Node runtime, and it has a Pydantic plugin if M1 chooses Pydantic. Pyright was the serious alternative (faster, matches VS Code's Pylance); Astral's ty was still pre-1.0 beta.

### ADR-0003d: Pre-commit hooks

**pre-commit**, with hooks that run as `uv run` local hooks so their versions come from `uv.lock` and can't drift from CI:

- ruff check (with `--fix`) and ruff format
- hygiene hooks from `pre-commit-hooks`: private-key detection, large-file check, TOML/YAML syntax, end-of-file and trailing-whitespace fixers
- **gitleaks** secret scanning, the one hook pulled from its own repo (pinned to a release)

mypy and pytest stay out of pre-commit because they're too slow for every commit; CI runs them. GitHub secret scanning with push protection is the backstop for a skipped hook.

### ADR-0003e: Dependency management

**uv**, with the `uv_build` backend, a committed `uv.lock`, and a PEP 735 `dev` dependency group for tooling. Poetry (slower, own lock format) and pip-tools (no Python or venv management) were the alternatives. How the Functions deployment package gets its dependencies from the lock is decided at M7/M8.

### ADR-0003f: CI shape

At M0, one workflow (`.github/workflows/ci.yml`) runs on every PR and every push to `main`:

- one job, `check`, on `ubuntu-latest` (Functions runs on Linux), Python 3.13 only
- `uv sync --locked` → `ruff check` → `ruff format --check` → `mypy` → `pytest`
- read-only `GITHUB_TOKEN`, superseded runs cancelled, a job timeout
- third-party actions pinned to full commit SHAs and kept current by Dependabot (GitHub Actions and uv, weekly)

**No Python version matrix:** freshrubric is a deployed app with one runtime, not a library. **No coverage gate yet:** the coverage policy is a testing-strategy decision for M1.

`main` is protected by a ruleset: changes land through a PR, `check` must pass, force-pushes and deletion are blocked, and no approving review is required (a solo maintainer can't approve his own PR).

CI grows with the milestones:

| Milestone | Adds |
|---|---|
| M1 | Coverage policy, decided in planning |
| M2 | Live fetch job: the three real job board fetches on every PR (ADR-0001e), with retry and flakiness policy decided then |
| M3 | Azurite service container for Blob tests |
| M4 | Bicep build and lint on PRs that touch infrastructure |
| M8 | Nightly workflow against real Azure resources using OIDC federated credentials (no stored Azure secrets), the plain-English "what changed" summary, and the deploy workflow |
| M9 | pytest-bdd features in the PR job |

## Consequences

- Style and lint questions have an answer to point to, so AI-assisted coding no longer has to stop on them.
- One tool (ruff) covers lint and format, and uv covers Python installs, environments, and locking. The dev toolchain is three tools plus pre-commit.
- Strict typing from the first line costs a little friction in tests but keeps mocks and fakes honest as the Azure SDK and LangChain surfaces grow.
- Hook versions and CI versions come from the same lockfile, except gitleaks, whose pinned release in `.pre-commit-config.yaml` is bumped separately.
- A single Python version in CI means a runtime upgrade is a deliberate version bump, tested once.
- The CI growth table is a plan, not a commitment of shape: each row still goes through planning when its milestone arrives.
