---
name: modelrush-chat-completion
description: Pick a live ModelRush text model and run an OpenAI-compatible chat
  completion, with correct region selection, streaming, retry, and billing
  discipline.
api: ModelRush API
operations:
  - listModels
  - listRegions
  - createChatCompletion
generated: '2026-09-10'
method: generated
source: openapi/modelrush-public.openapi.yaml + https://modelrush.ai/docs
---

# Run a chat completion on ModelRush

1. **Discover before you select.** Call `listModels` (`GET /v1/models`, keyless)
   and choose a model whose `modality` is `text` or `code` and whose
   `available_regions` includes a region acceptable to you. Model availability is
   time-sensitive -- never hardcode a model id from prose; a model is callable
   only when it appears in the current `GET /models` response. Optionally call
   `listRegions` to see live API surfaces and the default region.
2. **Authenticate server-side.** Send `Authorization: Bearer $MODELRUSH_API_KEY`.
   Keys are rejected in query strings; never expose the key to a browser client.
3. **Create the completion** with `createChatCompletion`
   (`POST /v1/chat/completions`): required fields `model` and `messages`; the
   request shape is OpenAI-compatible (`temperature`, `max_tokens`, `tools`,
   `tool_choice`, `response_format`). Pin execution with the `region` field when
   data residency matters. Set `stream: true` for an SSE `text/event-stream`.
4. **Handle errors by class** (`{error: {code, message, type}}` envelope): fix
   and do not retry 400/413 (`request_too_large`, `context_length_exceeded`);
   401 means fix credentials; 402 means top up credits; on 429 honor
   `Retry-After`; retry only 5xx/429 with bounded exponential backoff, jitter,
   and an overall deadline.
5. **Mind billing and tracing.** This operation is billable
   (`x-modelrush-billable: true`) and there is **no idempotency key** -- a blind
   retry of a completed request double-bills. Log the `x-request-id` response
   header for support and reconcile against the Requests dashboard.
