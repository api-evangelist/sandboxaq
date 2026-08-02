---
name: Audit an AI agent inventory with the Flint AI Platform API
description: Walk a tenant's discovered AI estate — agents, the models, tools and MCP servers they use,
  and the issues raised against them — using the Flint AI Platform API's inventory graph, then pull the
  rule behind any finding.
api: openapi/sandboxaq-flint-ai-platform-openapi.json
operations:
- GetAllAispmAgents
- GetAispmAgentDetails
- GetAISPMAgentEdges
- GetAllAispmMcpServers
- GetAispmMcpServerDetails
- GetAllAispmTools
- GetAispmToolDetails
- GetIssuesIndex
- GetIssueObjects
- GetRule
- GetSummaryAispmDashboard
---

# Audit an AI agent inventory

Use this when someone asks "what agents do we have", "which models and MCP servers are our agents
reaching", or "what's wrong with our AI estate" and the org runs the Flint AI Platform.

## Before you start

- **Base URL** is per-tenant: `https://{instance}.flintai.dev/api/v1`. `{instance}` is the customer's
  subdomain; the spec's `your-instance` default does not resolve.
- **Auth** is a bearer token created in the Platform console under **Settings → API Keys**, sent as
  `Authorization: Bearer <token>`. The token is shown once at creation. The spec declares the
  `BearerAuth` scheme but never applies it at operation level — send it anyway: all 55 operations
  declare a `401 Unauthorized`.
- **Every path is scoped** to `{tenant_id}` and `{workspace_id}`. You cannot read across tenants.
- **Collections are tabular.** Responses use `engine_output.EngineDataPage`: `header` names the columns
  and `rows` carries arrays of values. Zip `header` against each row before reasoning about a record.
- **Pagination is cursor-based.** Pass `cursor` and `page_size`; follow the `cursor` in the response
  until it is empty. One operation uses `pageSize` instead — check the parameter name per operation.

## Steps

1. **Take the temperature.** Call `GetSummaryAispmDashboard` for object counts, top issues, top at-risk
   agents/models/MCP servers, the issues histogram and scan counts. This tells you whether the estate is
   small enough to enumerate or you should go straight to the findings.

2. **List the agents.** Call `GetAllAispmAgents` for the tenant + workspace. Results are ordered by
   `last_seen` descending, so the top of the list is what is actually running. Page with `cursor` and
   `page_size`.

3. **Open an agent.** Call `GetAispmAgentDetails` with the agent id. The detail response embeds the
   neighbour collections directly — `models`, `tools`, `mcp_servers`, `sub_agents`, `locations`,
   `input_guardrails`, `output_guardrails` — plus `burn_rate` and `cost_history`. In most cases this one
   call answers "what does this agent touch" without any graph walking.

4. **Traverse when you need one hop at a time.** Call `GetAISPMAgentEdges` with the agent id and an
   `{edge}` name to fetch just that relationship. The same `{id}/{edge}` pattern exists on models
   (`GetAISPMModelEdges`), tools (`GetAISPMToolEdges`), MCP servers (`GetAISPMMcpServerEdges`), assets
   (`GetAssetEdges`) and locations (`GetLocationEdges`), so you can walk the whole graph.

5. **Audit the MCP surface specifically.** Call `GetAllAispmMcpServers`, then `GetAispmMcpServerDetails`
   on anything unfamiliar — the detail response embeds the agents that connect to it and the locations it
   was found in. Do the same with `GetAllAispmTools` / `GetAispmToolDetails`; the tool detail response
   additionally embeds `issues`, so an unrecognised tool with findings is your first stop.

6. **Pull the findings.** Call `GetIssuesIndex` to list issues grouped by `rule_id` and severity, with
   filtering, sorting and pagination. For any issue, call `GetIssueObjects` to see which assets, models
   or agents it affects, and `GetIssueInstances` / `GetInstanceDetailsV2` for the instance-level detail.

7. **Explain the finding.** Call `GetRule` with the issue's `rule_id` to get `rule_name`,
   `rule_description`, `object_types` and `source` from the unified rules table. Use
   `GetStaticRules` / `GetExternalRules` when you need to know whether a rule is built in or externally
   sourced. Never paraphrase a rule from memory — read it from the API.

## Rules

- **Errors are not RFC 9457.** A 4xx/5xx returns `application/json` with `common.RequestError`:
  `{"code": <int>, "message": <string>}`, both required. A `401` means the token is wrong or missing; a
  `403` means the token is valid but has no access to that tenant or workspace; a `404` means the
  resource is not in the tenant+workspace you named — re-check the scope before concluding it does not
  exist.
- **There is no idempotency contract.** No surface accepts an `Idempotency-Key`. The only writes in the
  whole API are the client-storage upserts (`PutClientStorage*`, idempotent by HTTP method) and deletes,
  plus `ExportView`. Do not retry `ExportView` blindly.
- **This API is read-mostly by design.** 47 of 55 operations are `GET`. If a task seems to need a write
  to inventory, it does not exist here — inventory is populated by the discovery GitHub Action
  (`sandbox-quantum/flintai-codescan-action`) and the runtime Python SDK (`flintai.wrap()`), not by the
  API.
- **No rate limits are documented** for this API. Be conservative when paging large estates.

## Cross-references

- Auth profile: `authentication/sandboxaq-authentication.yml`
- Conventions (pagination, scoping, error envelope): `conventions/sandboxaq-conventions.yml`
- Error catalog: `errors/sandboxaq-problem-types.yml`
- Entity graph: `data-model/sandboxaq-data-model.yml`
- CLI (`flintai scan` / `flintai eval`): `cli/sandboxaq-cli.yml`
