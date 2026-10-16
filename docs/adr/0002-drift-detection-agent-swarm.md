# ADR-0002: Drift-detection agent swarm and human-reviewed feedback loop

- **Status:** Partially accepted. Top-level decisions resolved 2026-08-26; the ingestion mechanism (ADR-0002d) is deferred to planning.
- **Date:** 2026-08-26
- **Deciders:** Bakari Holmes
- **Extends:** [ADR-0001](0001-reality-grounding-layer.md). Does not replace it; the on-request grounding flow is unchanged.

## Context

ADR-0001 grounds SpectrumHire's generators *on request*: when a user asks for a rubric, job description, or interview-prep plan, relevant real postings are retrieved at that moment. That solves drift for a single request.

Nothing watches the corpus *over time* and surfaces when reality has shifted enough that the generators themselves (their prompts, templates, or built-in assumptions) should change. Company sites, values, interview processes, and role requirements all evolve.

## Decision

Build a **drift-detection agent swarm** downstream of the ingestion and indexing pipeline. It watches the corpus for meaningful change over time and produces synthesized findings, not raw data, for human-reviewed incorporation into the rubric, job description, and interview-prep generators.

### ADR-0002a: What the swarm watches, and how

One agent per data source, mirroring the per-source ingestion pattern in ADR-0001a. Each agent compares the current indexed snapshot of its source with the prior snapshot and looks for material change: shifting role requirements, changed interview-process or values language, salary-band movement, and similar.

Agents read from the existing Azure AI Search index. They do not re-hit the job board APIs. Ingestion stays the pipeline's job; analysis is the swarm's job.

### ADR-0002b: Output: the Drift Finding

Each agent emits a structured **Drift Finding**, not free text:

- source and timestamp
- what changed, with specific before/after evidence
- confidence and severity
- which generator(s) it affects (rubric, job description, interview prep; can be more than one)
- a suggested action in plain language

SpectrumHire consumes these findings, not the raw postings or fetched pages.

### ADR-0002c: Human review gate: graduated trust, starting at 100%

At launch, **every finding requires human approval**, with no category exempt. Nothing is applied to the live generators automatically.

This is the first rung of a **graduated trust ladder**, not a permanent state:

1. **Every finding queues for review.** Only findings that are approved, or approved with edits, move forward.
2. **Every decision is logged** against the finding's category (source, confidence/severity tier, affected generators). This track record is what any future autonomy decision must be based on.
3. **Autonomy is never self-granted.** The swarm can surface a trust report (approval rates, rejection reasons, measurable downstream outcomes) by category. Promoting a category out of full review is an explicit maintainer decision made in planning (ADR-0001i).
4. **What "measured results" means is deliberately open.** A threshold such as "N consecutive approvals with zero reversals" is the obvious starting shape, but a change can look right and prove wrong later. The metric gets defined in planning once a real track record exists.

A tiered "small changes auto-apply from day one" option was considered and declined.

### ADR-0002d: How approved findings reach the generators: deferred

**Not yet decided.** An approved finding could edit a generator's prompt or template directly, update grounding content the generator retrieves at request time, or work some other way. This is a genuine open architecture question, to be decided in planning when the milestone is reached.

### ADR-0002e: Scope now vs. later

**Now:** current data sources only: Greenhouse, Lever, and Ashby postings plus the culture context from ADR-0001h.

**On the roadmap, not started:** broader company career/about pages, anonymous employee posts (Blind-style), interview-process reviews (Glassdoor-style), and community discussions (e.g., Reddit). Each would become its own agent following the ADR-0002a pattern, and none starts without a planning review of that source's access method, terms of service, and rate limits.

### ADR-0002f: Learning from reviewer judgment: better proposals, not more authority

Every approve, reject, or edit decision is feedback the swarm can use to propose better findings: fewer low-value findings that historically get rejected, more of the patterns that get approved.

This is learning aimed at **proposal quality**, a separate axis from **approval authority** (ADR-0002c). Better proposals may help a category build the clean track record needed to graduate, but "the swarm learned something" is never itself justification for more autonomy. Only an explicit maintainer decision grants that.

The learning mechanism (rule-based filtering from rejection patterns, few-shot examples in agent prompts, or something more elaborate) is undecided until real decision history exists.

## Consequences

- freshrubric goes from grounding single requests to monitoring and proposing updates over time: a materially more agentic system.
- New load-bearing pieces: the Drift Finding schema and store, a review queue (a generated report is a fine v1), a decision log, and eventually the ingestion mechanism (ADR-0002d).
- Starting at 100% human review means nothing changes in the live generators without approval, at the cost of real ongoing review work. This is not "fully autonomous drift correction."
- Graduated trust is a deliberate "start conservative, expand autonomy only against measured evidence" design.
- Depends on the base pipeline (Milestones 0–5) existing first.

## Roadmap additions

| Milestone | What | Needs |
|---|---|---|
| M10 | Drift Finding schema + snapshot diffing; one agent per current source | M5 |
| M11 | Review queue (a generated report is fine for v1) + decision log recording every approve/reject/edit by category | M10 |
| M12 | Approved findings reach the generators (mechanism decided in planning, ADR-0002d) | M11 + planning decision |
| M13 *(future)* | New agents for broader company pages, Blind-style posts, Glassdoor-style reviews, Reddit; each needs its own access/ToS/rate-limit review | M10–M12 proven |
| M14 *(future)* | Trust-report tooling from the M11 decision log, plus the judgment-learning mechanism (ADR-0002f) | M11 + real track record + planning decision |
