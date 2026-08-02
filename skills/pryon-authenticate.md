---
name: Authenticate to the Pryon APIs
description: Obtain and reuse an OAuth2 client-credentials JWT bearer token for every Pryon API call, on either the classic or the unified platform.
api: https://api.pryon.net
operations: []
docs: https://docs.pryon.com/reference/api-authentication
generated: '2026-08-02'
method: generated
---

# Authenticate to the Pryon APIs

Every Pryon operation is behind an OAuth 2.0 client-credentials JWT. Do this once per
process, not once per call.

## Before you start

- `client_id` and `client_secret` are issued by the customer's Pryon Solutions Manager, or by
  the organization administrator on an on-premises deployment. They are not self-serve.
- Determine which platform the tenant is on. It changes only the token host.

## Steps

1. **Request a token.** POST JSON to the token endpoint for the tenant's platform:
   - classic platform: `https://auth.pryon.net/oauth/token`
   - unified platform: `https://auth-us-central-1.pryon.net/oauth/token`

   ```json
   {
     "client_id": "<client_id>",
     "client_secret": "<client_secret>",
     "audience": "https://pryon/api",
     "grant_type": "client_credentials"
   }
   ```

2. **Read the response.** It returns `access_token`, `token_type: Bearer` and `expires_in`
   (default `86400`, i.e. 24 hours).

3. **Cache and reuse the token for its full lifetime.** Pryon explicitly documents reusing
   server tokens rather than minting one per request. Refresh only when `expires_in` is
   nearly elapsed or a call fails authentication.

4. **Send it on every API call** as `Authorization: Bearer <access_token>` against
   `https://api.pryon.net` (or the tenant's own base URL for VPC / on-prem / air-gapped
   deployments).

5. **Add the tenant header where required.** 34 administrative operations take
   `x-pryon-org-id`; supply the organization identifier.

## Gotchas

- The published OpenAPI documents disagree with each other on the header name: 140
  operations declare a header parameter called `Authentication` and 81 declare
  `Authorization`, and one securityScheme carries the typo `Authoriation: Bearer`. The
  documented and working form is `Authorization: Bearer <token>`.
- End-user (UI) flows use OpenID Connect implicit or authorization-code against the tenant's
  federated IdP instead of client credentials — do not use a machine token to act as a user.
- There is no scope surface. A client-credentials token is scoped by the service account
  Pryon issues it to, not by requested scopes.

## Error handling

Failures return the `google.rpc.Status` envelope — `{code, message, details[]}` — not
RFC 9457 problem+json. `code` is the numeric google.rpc.Code, not the HTTP status. See
`errors/pryon-problem-types.yml`.
