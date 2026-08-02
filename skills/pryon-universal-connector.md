---
name: Implement a Pryon Universal Connector
description: Serve the five-operation contract Pryon calls to ingest content from a repository that has no prebuilt connector, preserving document-level read permissions.
api: openapi/pryon-universal-connector-may2025-openapi.json
operations:
  - ConnectorServiceExternal_PullContents
  - ConnectorServiceExternal_DownloadContentData
  - ConnectorServiceExternal_GetMetadata
  - ConnectorServiceExternal_GetReadPermission
  - ConnectorServiceExternal_ListGroupPermissions
  - ConnectorServiceExternal_ListGroupPermissionMembers
docs: https://docs.pryon.com/docs/universal-connector
generated: '2026-08-02'
method: generated
---

# Implement a Pryon Universal Connector

This skill inverts the usual direction: **you implement the API and Pryon calls you.** Use it
when the content lives somewhere Pryon has no prebuilt connector for (it ships SharePoint,
Salesforce, Amazon S3, Azure Blob Storage, Google Cloud Storage, NFS and a File System
connector out of the box).

Implement all six operations, keyed by the `connector_id` Pryon assigns.

## The contract

1. **`ConnectorServiceExternal_PullContents`** —
   `GET /api/connector/v1/connectors/{connector_id}/contents:pull`
   List every content item to be indexed. This is the crawl. It **must paginate** —
   honour `page_size` / `page_token` and return a `next_page_token` — because Pryon walks the
   whole repository through it.

2. **`ConnectorServiceExternal_DownloadContentData`** —
   `GET /api/connector/v1/connectors/{connector_id}/content-data:download`
   Return the bytes of one content item. Pryon downloads in **multiple parts**, so support
   partial/chunked reads rather than assuming one call per document.

3. **`ConnectorServiceExternal_GetMetadata`** —
   `GET /api/connector/v1alpha1/connectors/{connector_id}/content/metadata`
   Return the content's own metadata. Whatever you emit here is what becomes filterable
   system-of-record metadata inside Pryon, and what entity extraction runs against — so map
   your repository's real fields, not a placeholder.

4. **`ConnectorServiceExternal_GetReadPermission`** —
   `GET /api/connector/v1/connectors/{connector_id}/content/read-permission`
   Return the permission group IDs that may read this item.

5. **`ConnectorServiceExternal_ListGroupPermissions`** —
   `GET /api/connector/v1/connectors/{connector_id}/group-permissions`
   List every permission group ID.

6. **`ConnectorServiceExternal_ListGroupPermissionMembers`** —
   `GET /api/connector/v1/connectors/{connector_id}/group-permission/members`
   List the users (by email) in a permission group.

## Why the permission operations matter most

Operations 4–6 are the reason Pryon can promise document-level ACLs survive ingestion. If
your implementation returns incomplete groups or stale membership, users will retrieve
passages from documents they are not entitled to read. Treat permission freshness as the
hard requirement of the connector, above throughput.

## Conventions that apply

- **Pagination** is `page_size` / `page_token` / `order_by`, Google AIP-158 style.
- **Errors** should be returned as the `google.rpc.Status` envelope
  (`{code, message, details[]}`) to match the rest of the platform.
- **Read-only.** Pryon connects to content repositories read-only by design
  (<https://www.pryon.com/about/security>); do not implement write paths.
- **Throttling.** Pryon's own connectors implement per-service request throttling and honour
  `Retry-After`; returning `Retry-After` on overload is the expected back-pressure signal.
