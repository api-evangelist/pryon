---
name: Build and inspect a Pryon knowledge collection
description: Create a knowledge collection from content sources, confirm the knowledge domain built behind it, and inspect the ingested content and content groups.
api: openapi/pryon-knowledge-collections-may2025-openapi.json
operations:
  - KnowledgeCollectionService_CreateCollectionWithSources
  - KnowledgeCollectionService_ListCollections
  - KnowledgeCollectionService_GetCollection
  - KnowledgeCollectionService_GetCollectionActiveDomain
  - KnowledgeCollectionService_GetCollectionDomain
  - ContentService_ListContents
  - ContentService_GetContent
  - ContentService_ListContentGroups
  - ContentService_GetSummary
  - KnowledgeCollectionService_UpdateCollectionActiveDomain
  - KnowledgeCollectionService_RestoreCollectionActiveDomain
docs: https://docs.pryon.com/docs/creating-a-collection
generated: '2026-08-02'
method: generated
---

# Build and inspect a Pryon knowledge collection

A **collection** is the durable object your users ask questions of. Behind it sits a
versioned **knowledge domain** — the immutable index that content is actually ingested into.
Understanding that split is the whole skill.

## Prerequisites

- A bearer token (see `pryon-authenticate.md`).
- Source content prepared per <https://docs.pryon.com/docs/preparing-source-content>, with
  URLs in the form each connector requires
  (<https://docs.pryon.com/docs/url-formats-by-connector>).

## Steps

1. **Create the collection with its sources.** `KnowledgeCollectionService_CreateCollectionWithSources`
   — `POST /api/knowledge/v1alpha1/collections`. This kicks off ingestion into a new
   knowledge domain.

2. **Find it again.** `KnowledgeCollectionService_ListCollections`
   (`GET /api/knowledge/v1/collections`) paginates with `page_size` (default and maximum 100),
   `page_token` and `order_by`, and filters on `team_id` and `solution_id`. Fetch one with
   `KnowledgeCollectionService_GetCollection`.

3. **Resolve the active domain.** `KnowledgeCollectionService_GetCollectionActiveDomain`
   (`GET /api/knowledge/v1/collections/{collection_id}/domain`) returns the knowledge domain
   currently serving answers. `KnowledgeCollectionService_GetCollectionDomain` fetches a
   specific version. `KnowledgeCollectionService_ListActiveDomains` lists them across the org.

4. **Inspect what actually ingested.** `ContentService_ListContents` filters by
   `knowledge_domain_id` and `status` — this is how you find failures. Then per item:
   - `ContentService_GetContent` — the content record
   - `ContentService_GetSummary` — its generated summary
   - `ContentService_ListProcessedContentExtracts` and
     `ContentService_ListRetrievalContentExtracts` — the chunks retrieval will see
   - `ContentService_GetContentMetadata` — metadata carried from the system of record
   - `ContentService_ListContentSubjects` — classifications applied

5. **Group content.** `ContentService_ListContentGroups`, `ContentService_GetContentGroup`
   and `ContentService_ListContentGroupContents` walk the grouping layer;
   `KnowledgeDomain_ListDomainContentGroups` scopes it to a domain.

6. **Promote or roll back a version.** `KnowledgeCollectionService_UpdateCollectionActiveDomain`
   (`PATCH .../domain:setActive`) points the collection at a domain version;
   `KnowledgeCollectionService_RestoreCollectionActiveDomain` (`PATCH .../domain:restoreActive`)
   reverts. This is the safe way to change what a collection answers — see
   <https://docs.pryon.com/docs/reverting>.

## Conventions that apply

- **Pagination** is `page_size` / `page_token` / `order_by` on every list operation; loop
  until no `next_page_token`.
- **No idempotency key exists.** `CreateCollectionWithSources` is a plain POST — a blind
  retry after a timeout can create a second collection. List and reconcile before retrying.
- **Deletes cascade.** `KnowledgeCollectionService_DeleteCollection` also decommissions every
  knowledge domain version associated with the record, and
  `ContentService_CascadeDeleteContent` removes content from all content groups. These are
  irreversible.
