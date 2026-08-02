# SandboxAQ

SandboxAQ (SB Technology, Inc.) builds Large Quantitative Models (LQMs) — AI systems that fuse physics,
chemistry and proprietary scientific data — and ships them as commercial platforms. Three product lines
carry public developer surfaces, and they do not share a data core.

## APIs

| API | Style | Contract | Base |
|---|---|---|---|
| [Flint AI Platform API](https://docs.flintai.dev/flintai/api-reference/index) | REST | OpenAPI 3.0.3, 55 operations | `https://{instance}.flintai.dev/api/v1` |
| [AQtive Guard API](https://aqtiveguard.sandboxaq.com/docs/api/) | GraphQL | 414 types, 6 queries, 73 mutations | `https://aqtiveguard.sandboxaq.com/api/v2` |
| [SandboxAQ MCP Server](https://docs.aisim.sandboxaq.com/mcp) | Model Context Protocol | 5 tools + 4 job-lifecycle helpers | `https://mcp.{tenant}.aisim.sandboxaq.com` |

## Artifacts

- `openapi/` — the Flint AI Platform API spec, harvested verbatim from
  `https://docs.flintai.dev/flintai/api-reference/openapi.json`
- `graphql/` — the AQtive Guard SDL, transcribed from the provider's published schema reference
  (live introspection is API-KEY gated) and validated with `graphql-core`
- `grpc/` — Protobuf definitions from the open-source [Sandwich](https://github.com/sandbox-quantum/sandwich)
  cryptography library
- `a2a/` — A2A Agent Card served at `https://docs.flintai.dev/.well-known/agent-card.json`, graded
- `mcp/` — MCP server manifest with published tool schemas, plus the tool crosswalk recording the hard
  split between the three surfaces
- `skills/` — the provider-published Flint agent skill (verbatim) plus one generated skill grounded in
  real operationIds
- `llms/` — `llms.txt` harvested verbatim from the Flint AI docs host
- `packages/`, `cli/` — the `flintai-cli` / `flintai-sdk-py` / `pysandwich` distributions and the
  `flintai` + `cs-api` command surfaces
- `authentication/`, `conventions/`, `errors/`, `data-model/`, `lifecycle/`, `changelog/`,
  `conformance/`, `well-known/`, `security/`, `agentic-access/`, `overlays/`

## Links

- Website — https://www.sandboxaq.com/
- Products — https://www.sandboxaq.com/products
- Blog — https://www.sandboxaq.com/blog
- GitHub — https://github.com/sandbox-quantum
- Secondary market listing — https://www.hiive.com/securities/sandboxaq-stock
