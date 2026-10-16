# Architecture walkthrough

What each part of freshrubric is, why it's shaped this way, and how the pieces fit. [ADR-0001](adr/0001-reality-grounding-layer.md) and [ADR-0002](adr/0002-drift-detection-agent-swarm.md) are the authoritative decision records; this doc is the guided tour.

> **Status:** design only. Nothing below is built yet. See the [roadmap](roadmap.md).

## 1. Where this sits

```mermaid
graph LR
    subgraph "Existing: live in production"
        SH[SpectrumHire<br/>Cloudflare Workers / Queues / R2<br/>direct Claude API calls]
    end

    subgraph "This project: standalone"
        FR[freshrubric<br/>grounding layer on Azure<br/>separate repo and infrastructure]
    end

    FR -. "wired in later,<br/>only once proven" .-> SH

    style FR fill:#2d5f7c,color:#fff
    style SH fill:#555,color:#fff
```

**Why separate:** SpectrumHire is a live product with its own architecture. Keeping freshrubric standalone means it can be built and experimented with on Azure, including getting things wrong, with no risk to the live product. Integration is a future step, not part of this build.

## 2. Data flow: how a real posting becomes a grounded, cited answer

```mermaid
flowchart TD
    GH["Greenhouse Job Board API<br/>(public, no auth)"]
    LV["Lever Postings API<br/>(public, no auth)"]
    AB["Ashby Job Board API<br/>(public, no auth)"]

    GH --> ING
    LV --> ING
    AB --> ING

    ING["Ingestion: Azure Function<br/>(timer-triggered)<br/>normalizes all sources into<br/>one JobPosting schema"]

    ING --> BLOB[("Blob Storage<br/>raw + normalized JSON<br/>+ culture/values context")]
    BLOB --> IDX["AI Search indexer<br/>(built-in, points at the container)"]
    IDX --> SEARCH[("Azure AI Search<br/>hybrid keyword + vector index")]

    subgraph "On request"
        REQ["role / company / title"] --> RET["Retrieve top-k<br/>relevant postings + culture context"]
        SEARCH --> RET
        RET --> CHAIN["LangChain: assemble<br/>grounded prompt"]
        CHAIN --> AOAI["Azure OpenAI<br/>completion"]
        AOAI --> OUT["Grounded output<br/>+ citations to real postings"]
    end

    style ING fill:#2d5f7c,color:#fff
    style CHAIN fill:#2d5f7c,color:#fff
```

**Why this shape:**

- **Blob → AI Search indexer** instead of hand-written indexing code: the standard Azure RAG pattern, with little custom code.
- **Three sources** (Greenhouse, Lever, Ashby), all sanctioned public APIs, not scraping (ADR-0001a).
- **Normalize at the edge:** each source maps to one schema, so a fourth job board only touches ingestion.
- **Culture context, not just requirements:** primarily the "About [Company]" section in postings; secondarily a guardrailed fetch of the company's own careers page (ADR-0001h).
- **Citations in the output:** they're what make "grounded" verifiable instead of a claim.

ADR-0002 adds a second flow alongside this one: a drift-detection agent swarm that watches the same index over time and proposes generator updates for human review.

## 3. Azure resource map

```mermaid
graph TB
    subgraph RG["Resource group"]
        ST[("Storage account<br/>Blob container")]
        AIS["AI Search<br/>(Free tier, hard-capped)"]
        FA["Function App<br/>(Consumption plan)<br/>timer + HTTP triggers"]
        AOAI["Azure OpenAI<br/>+ deployment-level TPM quota"]
        KV["Key Vault<br/>secrets"]
        AI["Application Insights<br/>logs + telemetry"]
    end

    GHA["GitHub Actions"] -->|"Bicep deploy"| RG
    GHA -->|"publish functions"| FA
    FA -.->|reads secrets| KV
    FA --> ST
    FA --> AIS
    FA --> AOAI
    FA -.->|telemetry| AI

    BUDGET["Azure Budget<br/>(~$15-20/mo threshold)"] -->|"80% / 100% alert"| AG["Action Group"]
    AG -->|"sets TPM quota to 0"| AOAI
    AG -->|"email"| NOTIFY["Notification"]

    style BUDGET fill:#5c2d2d,color:#fff
```

**Why Bicep + GitHub Actions:** standing an environment up reproducibly from code, and tearing it down cleanly to control cost, is the real skill, not clicking through the portal once.

**A real cost ceiling (ADR-0001g):** Azure Budgets only alert; they don't stop consumption. Two independent layers do: a low Azure OpenAI tokens-per-minute quota from day one (caps burn rate), and budget automation that sets that quota to 0 near the threshold (caps total spend). Everything else is free or near-free at this scale by design.

## 4. Testing strategy: TDD by default, a thin BDD layer on top

```mermaid
graph TD
    subgraph "Every PR: fast, mocked, plus one cheap live check"
        T1["Normalizers<br/>(each source → common schema)"]
        T2["HTTP client retry / pagination<br/>(mocked)"]
        T3["Blob idempotency<br/>(Azurite emulator)"]
        T4["AI Search calls<br/>(in-memory fake behind a thin interface)"]
        T5["LangChain chain logic<br/>(fake LLM + fake embeddings)"]
        T6["3 real job board fetches<br/>(free, fast, catch upstream changes)"]
    end

    subgraph "Nightly: real Azure resources"
        L1["Blob round-trip"]
        L2["AI Search round-trip"]
        L3["Azure OpenAI completion"]
        L4["Plain-English 'what changed and why' summary"]
    end

    subgraph "Acceptance: pytest-bdd, capped at 3-4 features"
        B1["Given a board with N postings,<br/>when ingestion runs,<br/>then the index has N documents"]
        B2["Given real postings exist,<br/>when a grounded rubric is requested,<br/>then the output cites a real posting"]
    end

    T1 & T2 & T3 & T4 & T5 & T6 -->|"gates every merge"| CI["GitHub Actions: PR check"]
    L1 & L2 & L3 & L4 -->|"costs money / needs secrets"| NIGHT["GitHub Actions: nightly"]
    B1 & B2 -->|"living documentation"| CI
```

Most of this project is deterministic transformation logic and infrastructure I/O, which unit tests handle well. BDD gets a narrow place because a few Given/When/Then scenarios double as documentation anyone can read.

## 5. ADR discipline

```mermaid
flowchart TD
    Q{"New load-bearing dependency?<br/>New Azure resource type?<br/>Reversing a prior ADR?<br/>Real tradeoff pause?<br/>Would forgetting 'why' hurt later?"}
    Q -->|yes to any| WRITE["Write docs/adr/000N-title.md<br/>Status / Context / Decision / Consequences"]
    Q -->|no| SKIP["No ADR: just build it<br/>(naming, version bumps, routine files)"]
    WRITE --> INDEX["Link it from docs/adr/README.md"]
```

## 6. Decision process

Decisions (architecture, style, linting, testing, features, CI/CD, docs) are made in planning and recorded here. AI-assisted coding implements decided designs and stops to ask when something isn't decided. See [AGENTS.md](../AGENTS.md) and ADR-0001i.
