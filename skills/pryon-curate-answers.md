---
name: Curate Pryon answers with feedback and verified answers
description: Capture ratings on retrievals and exchanges, run them through review and approval, and promote a corrected response into a verified answer that outranks retrieval.
api: openapi/pryon-retrieval-may2025-openapi.json
operations:
  - RetrievalFeedback_CreateRetrievalRating
  - RetrievalFeedback_GetRetrievalRating
  - RetrievalFeedback_UpdateRetrievalRating
  - RetrievalFeedback_ListRetrievalRatingReviews
  - RetrievalFeedback_UpdateRetrievalRatingReview
  - RetrievalFeedback_CreateVerifiedAnswer
  - RetrievalFeedback_ListVerifiedAnswers
  - RetrievalFeedback_UpdateVerifiedAnswer
  - Feedback_CreateExchangeRating
  - Feedback_ListExchangeRatingReviews
  - Feedback_CreateApproval
  - Feedback_ListApprovals
  - GenerativeFeedback_CreateGenerativeExchangeRating
  - GenerativeRetrievalFeedback_CreateGenerativeRetrievalRating
docs: https://docs.pryon.com/docs/verified-answers
generated: '2026-08-02'
method: generated
---

# Curate Pryon answers with feedback and verified answers

Pryon's quality loop is a three-stage pipeline in the API: a **rating** is left on a result,
a **review** records a curator's perspective on that rating, and an **approval** records the
decision. When the outcome is "the right answer is X", that becomes a **verified answer**,
which is prioritized ahead of retrieval for matching queries.

There is one rating family per result type. Pick the one that matches what was answered:

| Result type | Create rating |
|---|---|
| Extractive retrieval | `RetrievalFeedback_CreateRetrievalRating` |
| Extractive exchange | `Feedback_CreateExchangeRating` |
| Generative exchange | `GenerativeFeedback_CreateGenerativeExchangeRating` |
| Generative retrieval | `GenerativeRetrievalFeedback_CreateGenerativeRetrievalRating` |

## Steps

1. **Capture the rating** with the operation from the table, posting the identifier of the
   retrieval or exchange being rated.

2. **Read it back and correct it.** `RetrievalFeedback_GetRetrievalRating` /
   `RetrievalFeedback_UpdateRetrievalRating` (and the equivalents in each family) fetch and
   amend a rating.

3. **Review it.** `RetrievalFeedback_ListRetrievalRatingReviews` lists the review
   perspectives on ratings; `RetrievalFeedback_GetRetrievalRatingReview` and
   `RetrievalFeedback_UpdateRetrievalRatingReview` read and write the curator's view. The
   same shape exists as `Feedback_*`, `GenerativeFeedback_*` and
   `GenerativeRetrievalFeedback_*`.

4. **Approve.** `Feedback_CreateApproval` records an approval entry;
   `Feedback_ListApprovals`, `Feedback_GetApproval` and `Feedback_UpdateApproval` manage it.
   This is the gate before a correction changes what users see.

5. **Promote the correction.** `RetrievalFeedback_CreateVerifiedAnswer` —
   `POST /api/conversation/v1alpha1/verified-answers` — makes the approved answer the one
   returned for matching queries. `RetrievalFeedback_ListVerifiedAnswers`,
   `RetrievalFeedback_GetVerifiedAnswer` and `RetrievalFeedback_UpdateVerifiedAnswer` maintain
   the set; `RetrievalFeedback_DeleteVerifiedAnswer` retires one.

6. **Watch the aggregate.** `Analytics_ListExchangeRatingCategories`,
   `Analytics_ListExchangeConfidenceCategories` and `Analytics_ListExchangeConfidenceScores`
   show whether curation is actually moving answer quality.

## Conventions that apply

- **Verified answers override retrieval.** Treat creating one as a production content change,
  not as feedback — gate it behind the approval step.
- **Update paths are unusual.** Several PATCH operations put the id under a `data.` prefix
  (e.g. `/retrievals/ratings/{data.rating_id}`); build the URL from the spec, not by
  convention.
- **No idempotency key.** A retried rating POST creates a second rating; list before retry.
- **Errors are `google.rpc.Status`** — see `errors/pryon-problem-types.yml`.
