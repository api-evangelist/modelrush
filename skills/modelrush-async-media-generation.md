---
name: modelrush-async-media-generation
description: Generate images or video on ModelRush -- upload private input media,
  submit the generation, then poll the Prediction or receive a signed webhook,
  with cancellation and at-least-once delivery handled correctly.
api: ModelRush API
operations:
  - listModels
  - createUpload
  - getUpload
  - createImageGeneration
  - createImageEdit
  - createVideoGeneration
  - getVideoGeneration
  - getPrediction
  - cancelPrediction
  - createWebhookEndpoint
  - listWebhookEndpoints
generated: '2026-09-10'
method: generated
source: openapi/modelrush-public.openapi.yaml + https://modelrush.ai/docs/concepts/webhooks
---

# Generate media asynchronously on ModelRush

1. **Pick a live model.** `listModels` (keyless) -- filter `modality` to `image`
   or `video` and confirm the target region is in `available_regions`.
2. **Upload private inputs when needed.** `createUpload` (`POST /v1/uploads`)
   returns a one-time `upload_url` (with `upload_headers`); PUT the bytes there,
   then confirm with `getUpload` and pass the signed `input_url` into the
   generation request. Uploads expire at `expires_at`; `deleteUpload` removes one
   early.
3. **Submit the generation.** `createImageGeneration`
   (`POST /v1/images/generations`, prompt <= 5000 chars, `n` 1-4),
   `createImageEdit`, or `createVideoGeneration`
   (`POST /v1/videos/generations`). All are billable. Video jobs are always
   asynchronous and return a job with a `poll_url`.
4. **Wait the right way.** Either poll `getPrediction`
   (`GET /v1/predictions/{id}`) / `getVideoGeneration`
   (`GET /v1/videos/generations/{id}`) with backoff, or register a webhook once
   with `createWebhookEndpoint` (`POST /v1/webhooks/endpoints`, public HTTPS URL;
   the `signing_secret` is returned **only** in this response -- store it).
   Verify every delivery: `x-modelrush-signature` is `t=TIMESTAMP,v1=HMAC`
   (HMAC-SHA256 over the raw body, constant-time compare); dedupe on
   `x-modelrush-event-id` because delivery is at-least-once (up to 8 attempts,
   exponential backoff, 10s timeout, then dead-letter). Polling stays available
   as the fallback when webhooks are delayed.
5. **Cancel while you still can.** `cancelPrediction`
   (`DELETE /v1/predictions/{id}`) requests cancellation and returns the terminal
   state; a 409 means the job already reached a terminal state and will be
   billed. There is no reversal for a completed generation.
