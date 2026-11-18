# ADR-0004: Host the Function App on the Flex Consumption plan

- **Status:** Accepted
- **Date:** 2026-10-19
- **Deciders:** Bakari Holmes
- **Amends:** [ADR-0001c](0001-reality-grounding-layer.md#adr-0001c-compute-azure-functions-python) and the Azure Functions row of [ADR-0001g](0001-reality-grounding-layer.md#adr-0001g-cost-ceiling-and-kill-switch). Everything else in ADR-0001 stands.

## Context

ADR-0001c chose Azure Functions (Python), and ADR-0001g's cost table put it on the Consumption plan. Python function apps run only on Linux, and by M0 Microsoft had frozen the Linux Consumption plan:

- Python 3.12 is the last Python version Linux Consumption will get.
- The plan receives no new features and retires on 2028-09-30.
- Microsoft now calls **Flex Consumption** its recommended serverless hosting plan.
- Apps can't be migrated in place between plans; moving means creating a new function app and redeploying.

Choosing the plan at M0, before any Functions infrastructure exists (M4/M7), costs nothing. Choosing it after M7 means rebuilding the app.

## Decision

Host freshrubric's Function App on the **Flex Consumption** plan, running **Python 3.13** (the version [ADR-0003a](0003-python-toolchain-and-conventions.md#adr-0003a-python-version-and-layout) pins).

- **On-demand billing only.** Always-ready instances stay at their default of 0, so the app scales to zero and uses the on-demand monthly free grant. Always-ready billing has no free grant.
- **Maximum instance count** is set deliberately low in Bicep. Flex allows a maximum as low as 1, which adds a third brake on runaway cost alongside the TPM quota and budget automation in ADR-0001g. The exact value is set at M7.
- Timer and HTTP triggers, as ADR-0001c specified, are unchanged.

## Consequences

- freshrubric runs on the plan Microsoft is investing in, with Python 3.13 supported until October 2029, instead of a plan that retires in 2028.
- ADR-0001g's cost picture holds: the plan scales to zero and on-demand use at personal scale stays within the free grant. Confirm current grant amounts on the Azure Functions pricing page before M7.
- Deployment is a zip package uploaded to a Blob container (by default in the same storage account the app already needs). That suits the `src/` layout and uv: CI builds the package, Flex runs it. The packaging details are decided at M7/M8.
- Flex Consumption isn't in every Azure region. The region chosen for Azure OpenAI quota and AI Search must also support Flex.
- Flex has one app per plan and no deployment slots. Neither matters at this scale.
- The `WEBSITE_TIME_ZONE` and `TZ` settings aren't supported, so timer schedules are written in UTC.
