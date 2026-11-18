---
paths:
  - ".github/**"
---

# Changing CI and GitHub config

- **CI design is a planning decision** (AGENTS.md). Implement what ADR-0003f and its growth table say; raise anything else.
- **Pin third-party actions to a full commit SHA** with the version in a trailing comment. Dependabot keeps them current.
- **Keep the token read-only** (`permissions: contents: read`) unless a job needs more, and scope any extra permission to that job.
- **One Python version, no matrix.** CI tests the version Functions runs.
- **No stored Azure secrets.** The nightly and deploy workflows (M8) authenticate with OIDC federated credentials. Anything that costs money runs nightly, not on PRs.
