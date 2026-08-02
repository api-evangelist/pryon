# Pryon

Pryon is a Raleigh, North Carolina enterprise AI company founded in 2017 by Igor Jablokov that
provides a retrieval-augmented generation (RAG) platform for regulated and security-conscious
organizations. The Pryon RAG Suite couples an Ingestion Engine, a Retrieval Engine and a
Generative Engine behind a single set of REST APIs, with Symphony as the low-code orchestration
layer and App Builder producing embeddable conversational applications. It deploys in Pryon
SaaS, customer VPC, on-premises and air-gapped environments.

- Website — https://www.pryon.com/
- Developer documentation — https://docs.pryon.com/
- API authentication — https://docs.pryon.com/reference/api-authentication
- Release notes — https://docs.pryon.com/changelog
- llms.txt — https://docs.pryon.com/llms.txt

## APIs

| API | Operations | Contract |
|---|---|---|
| Pryon Admin API | 136 | `openapi/pryon-admin-openapi.json` |
| Pryon Retrieval API | 48 | `openapi/pryon-retrieval-may2025-openapi.json` |
| Pryon Generative API | 43 | `openapi/pryon-generative-*.json` |
| Pryon Feedback API | 22 | `openapi/pryon-feedback-*.json` |
| Pryon User-Defined Metadata API | 11 | `openapi/pryon-user-defined-metadata-may2025-openapi.json` |
| Pryon Universal Connector API | 6 | `openapi/pryon-universal-connector-may2025-openapi.json` |
| Pryon Knowledge Collections API | 6 | `openapi/pryon-knowledge-collections-may2025-openapi.json` |
| Pryon Exchange API | 4 | `openapi/pryon-exchange-*.json` |

22 OpenAPI documents, 276 operations, 184 unique operationIds — all harvested verbatim from
`https://docs.pryon.com/openapi/`.

## Artifacts

`openapi/` `overlays/` `authentication/` `conformance/` `conventions/` `errors/` `lifecycle/`
`changelog/` `data-model/` `components/` `packages/` `security/` `well-known/` `llms/` `mcp/`
`skills/` `asyncapi/` `agentic-access/`

Notable findings from the 2026-08-02 enrichment pass:

- **No MCP server, no A2A agent card, no GraphQL.** `mcp/pryon-mcp.yml` carries a *candidate*
  tool set derived from the OpenAPI, not a published server.
- **No public SDK packages.** Pryon markets JavaScript/TypeScript/Python client libraries but
  ships them under enterprise engagement; nothing on npm, PyPI, or its GitHub org.
- **No `/.well-known/` surface**, no security.txt, no status page, no deprecation policy.
- **No idempotency contract** and no rate-limit documentation.
- **Real streaming, no events.** Two WebSocket and three SSE operations; no webhooks and no
  AsyncAPI — recorded honestly in `asyncapi/pryon-events.yml`.
- **Errors are `google.rpc.Status`**, not RFC 9457 — every operation declares only `200` and
  `default`.
