---
paths:
  - "tests/**"
---

# Writing tests

- **Test first.** Write the failing test, watch it fail for the right reason, then write the code (ADR-0001e).
- **The unit suite never touches the network or Azure.** Use mocks, fixtures, Azurite, and in-memory fakes behind thin interfaces. It gates every PR.
- **Live tests are split by cost.** The three real job board fetches run on every PR. Anything touching real Azure resources (Blob, AI Search, Azure OpenAI) runs nightly only, because it costs money and needs secrets.
- **BDD stays thin.** pytest-bdd is capped at 3–4 feature files. A 5th is a signal it belongs in plain pytest.
- **Tests are typed too** (`-> None`, typed fixtures); `assert` is allowed here (ADR-0003c).
- Run `uv run pytest`.
