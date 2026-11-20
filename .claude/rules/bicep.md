---
paths:
  - "**/*.bicep"
  - "**/*.bicepparam"
---

# Writing infrastructure (Bicep)

- **Keep the cost ceiling** (ADR-0001g): AI Search on the Free (F0) tier, a deliberately low tokens-per-minute quota on the Azure OpenAI deployment from day one, and the Budget → Action Group → Automation chain that sets that quota to 0.
- **The Function App runs on Flex Consumption, on demand** (ADR-0004): always-ready instances stay at 0, and the maximum instance count stays deliberately low.
- **No secrets in templates or parameter files.** Secrets live in Key Vault (AGENTS.md, ADR-0001f).
- **A new Azure resource type needs an ADR** before it appears here.
