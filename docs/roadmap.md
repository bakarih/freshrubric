# Roadmap

Milestones are sequenced so work is never blocked waiting on Azure: M0–M3 need no Azure account. Azure OpenAI no longer needs an access request for standard use, but it does need model quota in the chosen region, and a new subscription may have to request it. M0–M3 cover that lead time.

**Current status: M0 in progress.** Coding style, linting, tooling, and CI are decided ([ADR-0003](adr/0003-python-toolchain-and-conventions.md)), as is the Functions hosting plan ([ADR-0004](adr/0004-flex-consumption-hosting.md)). No application code yet.

## Base pipeline (ADR-0001)

| Milestone | What | Needs | Status |
|---|---|---|---|
| M0 | Repo, license, CI skeleton; decide coding style and linting in planning; confirm Azure OpenAI quota | Nothing | In progress: conventions and hosting decided, scaffolding and CI in place; Azure OpenAI quota check pending |
| M1 | `JobPosting` schema + normalizers (TDD, fixtures) | Nothing | Not started |
| M2 | HTTP client layer: real Greenhouse, Lever, Ashby calls | Nothing | Not started |
| M3 | Local pipeline + Azurite (Blob emulator) | Nothing | Not started |
| M4 | Azure foundation via Bicep (Storage, AI Search, App Insights) | Azure subscription | Not started |
| M5 | Ingestion wired to real Blob + AI Search indexer; careers-page fetch with guardrails | Azure subscription | Not started |
| M6 | LangChain RAG generation with citations, job description generator first | Azure OpenAI quota | Not started |
| M7 | Azure Functions wrapper on Flex Consumption (timer + HTTP triggers) + cost-ceiling automation | Azure OpenAI quota | Not started |
| M8 | CI/CD deploy pipeline + nightly live tests | Azure OpenAI quota | Not started |
| M9 | Extend to rubric and interview-prep generators; BDD capstone; README polish | Azure OpenAI quota | Not started |

By the end of M3 there's a demoable local CLI that fetches real postings and writes normalized JSON, before any cloud dependency exists.

## Drift-detection agent swarm (ADR-0002)

| Milestone | What | Needs | Status |
|---|---|---|---|
| M10 | Drift Finding schema + snapshot diffing; one agent per current source | M5 | Not started |
| M11 | Review queue + decision log (every approve/reject/edit recorded by category) | M10 | Not started |
| M12 | Approved findings reach the generators (mechanism decided in planning) | M11 + planning decision | Not started |
| M13 *(future)* | New agents for additional sources, each after its own access/ToS/rate-limit review | M10–M12 proven | Not started |
| M14 *(future)* | Trust-report tooling + judgment-learning mechanism | M11 + real track record | Not started |
