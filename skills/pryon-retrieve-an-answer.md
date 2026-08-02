---
name: Retrieve a grounded answer from a Pryon collection
description: Run extractive retrieval against a collection, read the ranked outputs and their source content, render the citation image, and stream a generative answer over SSE.
api: openapi/pryon-retrieval-may2025-openapi.json
operations:
  - Retrieval_CreateRetrieval
  - Retrieval_GetRetrieval
  - Retrieval_CreateWsRetrieval
  - Retrieval_GetRetrievalContentImage
  - ContentService_ListRetrievalContentExtracts
  - ContentService_GetContentMetadata
  - GenerativeRetrieval_CreateGenerativeRetrievalSSE
  - GenerativeRetrieval_GetGenerativeRetrieval
docs: https://docs.pryon.com/reference/retrieval-request-and-response
generated: '2026-08-02'
method: generated
---

# Retrieve a grounded answer from a Pryon collection

Two modes share one retrieval core: **extractive** returns ranked passages from your own
documents; **generative** summarizes those passages into prose with attribution. Both are
grounded in the same retrieval, so the citation story is identical.

## Prerequisites

- A bearer token (see `pryon-authenticate.md`).
- A published collection and its `collection_id` (see `pryon-build-a-collection.md`).

## Extractive retrieval

1. **Ask.** `Retrieval_CreateRetrieval` — `POST /api/conversation/v1alpha1/retrievals`.
   Send the query, the `collection_id`, and any metadata filters. Pass a `conversation_id`
   to continue an existing conversation so query canonicalization and reference resolution
   have prior turns to work with.

2. **Read the outputs.** `Retrieval_GetRetrieval`
   (`GET /api/conversation/v1alpha1/retrievals/{retrieval_id}`) returns the query and its
   ranked outputs. Rank order is the confidence order — highest match first.

3. **Show the source.** `Retrieval_GetRetrievalContentImage`
   (`GET /api/conversation/v1alpha1/retrievals/{retrieval_id}/image`) returns an image of the
   source page with bounding boxes over the passage that answered the question. This is the
   citation UI Pryon expects you to build.

4. **Go deeper on a source document.** `ContentService_ListRetrievalContentExtracts` lists
   every retrieval extract for a `content_id`; `ContentService_GetContentMetadata` returns
   the system-of-record metadata for that content inside the domain.

5. **Stream instead of polling.** `Retrieval_CreateWsRetrieval`
   (`POST /api/conversation/v1alpha1/retrievals/wss`) opens the same conversation over a
   WebSocket.

## Generative retrieval

1. **Ask and stream.** `GenerativeRetrieval_CreateGenerativeRetrievalSSE`
   (`POST /api/conversation/v1alpha1/generative-retrievals`) runs retrieval and streams the
   summarized answer back as server-sent events.

2. **Fetch the finished result.** `GenerativeRetrieval_GetGenerativeRetrieval` returns the
   completed generative retrieval by id — use it to re-read an answer you already streamed.

## Conventions that apply

- **Filter, do not over-fetch.** Metadata filters (`data-model/pryon-data-model.yml`) narrow
  retrieval far more cheaply than post-filtering. Vast search can rank up to 400 results.
- **Verified answers win.** If a verified answer matches the query, it is prioritized over
  retrieval output — see `pryon-curate-answers.md`.
- **Errors are `google.rpc.Status`.** Only `200` and `default` are declared; read `code` and
  `message` from the envelope, not the HTTP status alone.
- **No idempotency key.** Retries of `Retrieval_CreateRetrieval` create new retrievals.
