# freshrubric

A grounding layer that keeps AI-generated hiring content tied to the real, current job market.

[SpectrumHire](https://interviewrubric.com) generates interview rubrics, job descriptions, and interview-prep plans. Today those generators work from a model's training data plus whatever the user pastes in. Training data goes stale: hiring language changes, companies rewrite their values pages, and interview loops for the same title look different from company to company.

freshrubric fixes that. It ingests live job postings and company culture context, indexes them for retrieval, and grounds generation in what real companies are posting right now, with **citations back to the real postings** so every grounded claim can be checked.

## Status

**Design phase. No application code yet.**

- Architecture and scope are decided: see [ADR-0001](docs/adr/0001-reality-grounding-layer.md) and [ADR-0002](docs/adr/0002-drift-detection-agent-swarm.md).
- Implementation starts at Milestone 0. See the [roadmap](docs/roadmap.md).
- This README will be updated as each milestone lands. If it says something is built, it is in this repo.

## How it will work

1. **Ingest.** A scheduled job pulls postings from the public, no-auth job board APIs of Greenhouse, Lever, and Ashby. No scraping.
2. **Normalize.** Each source maps into one common `JobPosting` schema, so adding a new board only touches the ingestion layer.
3. **Store.** Raw and normalized JSON, plus company culture/values context, land in Azure Blob Storage.
4. **Index.** An Azure AI Search indexer builds a hybrid (keyword + vector) index.
5. **Retrieve and generate.** On request, the most relevant postings and culture context are retrieved and a LangChain chain grounds an Azure OpenAI completion in them.
6. **Cite.** The output links back to the real postings it was grounded in.

A second flow ([ADR-0002](docs/adr/0002-drift-detection-agent-swarm.md)) adds a drift-detection agent swarm that watches the index over time and proposes generator updates for human review.

Full detail, with diagrams: [docs/architecture.md](docs/architecture.md).

## Planned stack

Python · Azure Functions · Azure Blob Storage · Azure AI Search · Azure OpenAI · LangChain (LangGraph only if a real multi-step need appears) · Bicep · GitHub Actions · Application Insights · pytest / pytest-bdd

## Design principles

- **Sanctioned data only.** Public job board APIs and companies' own careers pages, fetched with robots.txt checks, rate limits, and a clear User-Agent.
- **Grounded means verifiable.** Output carries citations to its sources.
- **A real cost ceiling, not just an alert.** A low Azure OpenAI tokens-per-minute quota plus budget automation that shuts spend off.
- **Humans approve changes.** Drift findings start at 100% human review, with autonomy granted only against measured results.
- **Plan first, then build.** Decisions are made in planning and recorded as ADRs; AI-assisted coding implements decided designs. See [AGENTS.md](AGENTS.md).

## Docs

- [Architecture walkthrough](docs/architecture.md)
- [Roadmap](docs/roadmap.md)
- [Architecture Decision Records](docs/adr/README.md)

## License

[MIT](LICENSE) © 2026 Bakari Holmes
