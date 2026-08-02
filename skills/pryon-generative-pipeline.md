---
name: Run the Pryon generative pipeline safely
description: Guard, route, rewrite and complete a query against Pryon's generative engine, and summarize retrieval outputs into an attributed answer.
api: openapi/pryon-chat-completion-only-generative-openapi.json
operations:
  - Generative_ListChatCompletionsModels
  - Generative_GuardrailQuery
  - Generative_RouteQuery
  - Generative_CompleteQuery
  - Generative_CreateChatCompletions
  - Generative_AnswerSummarization
docs: https://docs.pryon.com/recipes/getting-started-with-generative-apis
generated: '2026-08-02'
method: generated
---

# Run the Pryon generative pipeline safely

Pryon exposes the generative engine as discrete tasks rather than one opaque endpoint. Chain
them in this order and you get the guardrails, routing and grounding that make the answer
defensible.

## Prerequisites

- A bearer token (see `pryon-authenticate.md`).

## Steps

1. **Pick a model.** `Generative_ListChatCompletionsModels` —
   `GET /api/generative/v1alpha1/chat/completions/models` — lists the models available to the
   tenant for chat completions. Do not hardcode a model name; deployments differ, and
   air-gapped tenants can bring their own LLM.

2. **Guard the query first.** `Generative_GuardrailQuery` —
   `POST /api/generative/v1alpha1/tasks/guardrail-query` — returns classifications for query
   toxicity and prompt injection. Run this before anything reaches a model, and stop on a
   positive classification.

3. **Route it.** `Generative_RouteQuery` — `POST /api/generative/v1alpha1/tasks/route-query` —
   classifies the incoming query so you can send it down the right response pathway (a RAG
   lookup, a direct completion, a refusal). This is what Symphony's conditional branching is
   built on.

4. **Rewrite references out of it.** `Generative_CompleteQuery` —
   `POST /api/generative/v1alpha1/tasks/complete-query` — replaces pronouns and references
   from prior turns with their referents, so a multi-turn question retrieves as well as a
   standalone one. Always do this before retrieval in a conversation.

5. **Complete.** `Generative_CreateChatCompletions` —
   `POST /api/generative/v1alpha1/chat/completions` — is OpenAI-compatible and supports
   streaming, function calling and structured outputs (Q1 2026). Use structured outputs
   whenever a downstream system consumes the result.

6. **Or summarize retrieval instead of generating free-form.**
   `Generative_AnswerSummarization` — `POST /api/generative/v1alpha1/tasks/summarize-answer` —
   takes retrieval outputs plus configuration and produces a summarized answer with
   fine-grained attribution. Prefer this over a bare completion when the answer must be
   grounded in customer content.

## Conventions that apply

- **Guardrail, then route, then rewrite, then generate.** Skipping the guardrail step puts an
  unclassified user string in front of a model.
- **Ground before you generate.** Summarizing retrieval outputs (step 6) is what makes an
  answer citable; a raw chat completion is not grounded in the collection.
- **Errors are `google.rpc.Status`** with `{code, message, details[]}` — see
  `errors/pryon-problem-types.yml`.
- **These are `v1alpha1` surfaces.** Expect them to move; pin your integration to a
  documented release (`changelog/pryon-changelog.yml`).
