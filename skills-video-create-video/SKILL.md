---
name: skills-video-create-video
description: Build and execute skills.video video generation REST requests from OpenAPI specs. Use when Codex needs to create, debug, or document video generation calls on open.skills.video, including SSE-first endpoint selection, payload validation, stream handling, and polling fallback when SSE is unavailable.
---

# skills.video Create Video

## Overview
Use this skill to turn OpenAPI definitions into working video-generation API calls for `skills.video`.
Prefer deterministic extraction from `openapi.json` instead of guessing fields.

## Workflow
1. Check API key and bootstrap environment on first use.
2. Identify the active spec.
3. Select the SSE endpoint pair for a video model.
4. Extract request schema and generate a payload template.
5. Execute `POST /generation/sse/...` as default.
6. Fall back to polling only when SSE fails or is unavailable.
7. Apply retry and failure handling.

## 0) Check API key (first run)
Run this check before any API call.

```bash
python scripts/ensure_api_key.py
```

If `ok` is `false`, tell the user to:

- Open `https://skills.video/dashboard/developer` and log in
- Click `Create API Key`
- Export the key as `SKILLS_VIDEO_API_KEY`

Example:

```bash
export SKILLS_VIDEO_API_KEY="<YOUR_API_KEY>"
```

## 1) Identify the spec
Load the most specific OpenAPI first.

- Prefer model-specific OpenAPI when available (for example `/v1/openapi.json` under a model namespace).
- Fall back to platform-level `openapi.json`.
- Use `references/open-platform-api.md` for base URL, auth, and async lifecycle.

## 2) Select a video endpoint
If `docs.json` exists, derive video endpoints from the `Videos` navigation group.
Use `default_endpoints` from the script output as the primary list (SSE first).

```bash
python scripts/inspect_openapi.py \
  --openapi /abs/path/to/openapi.json \
  --docs /abs/path/to/docs.json \
  --list-endpoints
```

When `docs.json` is unavailable, pass a known endpoint directly (for example `/generation/sse/kling-ai/kling-v2.6`).
Use `references/video-model-endpoints.md` as a snapshot list.

## 3) Extract schema and build payload
Inspect endpoint details and generate a request template from required/default fields.

```bash
python scripts/inspect_openapi.py \
  --openapi /abs/path/to/openapi.json \
  --endpoint /generation/sse/kling-ai/kling-v2.6 \
  --include-template
```

Use the returned `request_template` as the starting point.
Do not add fields not defined by the endpoint schema.
Use `default_create_endpoint` from output unless an explicit override is required.

## 4) Execute SSE request (default)
Use bearer authentication and keep the stream open.

```bash
curl -N -X POST "https://open.skills.video/api/v1/generation/sse/kling-ai/kling-v2.6" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY" \
  -H "Accept: text/event-stream" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A cinematic dolly shot of neon city rain at night"
  }'
```

Treat SSE as the default result channel.
Capture task `id` from stream events if present.

## 5) Fall back to polling
Use polling only if SSE cannot be established, disconnects early, or does not reach a terminal state.
Use `GET /generation/{id}` (or model-spec equivalent path if the OpenAPI uses `/v1/...`).

```bash
curl -X GET "https://open.skills.video/api/v1/generation/<GENERATION_ID>" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY"
```

Stop polling on terminal states:

- `COMPLETED`
- `FAILED`
- `CANCELED`

## 6) Handle errors and retries
Handle these response codes for create, SSE, and fallback poll operations:

- `400`: request format issue
- `401`: missing/invalid API key
- `402`: possible payment/credits issue in runtime
- `404`: endpoint or generation id not found
- `422`: schema validation failed

Classify non-2xx runtime errors with:

```bash
python scripts/handle_runtime_error.py \
  --status <HTTP_STATUS> \
  --body '<RAW_ERROR_BODY_JSON_OR_TEXT>'
```

If category is `insufficient_credits`, tell the user to recharge:

- Open `https://skills.video/dashboard` and go to Billing/Credits
- Recharge or purchase additional credits
- Retry after recharge

Optional balance check:

```bash
curl -X GET "https://open.skills.video/api/v1/credits" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY"
```

Apply retries only for transient conditions (network failure or temporary `5xx`).
Use bounded exponential backoff (for example `1s`, `2s`, `4s`, max `16s`, then fail).
Do not retry unchanged payloads after `4xx` validation errors.

## Rate limits and timeouts
Treat rate limits and server-side timeout windows as unknown unless documented in the active OpenAPI or product docs.
If unknown, explicitly note this in output and choose conservative client defaults.

## Resources
- `scripts/ensure_api_key.py`: validate `SKILLS_VIDEO_API_KEY` and show first-run setup guidance
- `scripts/handle_runtime_error.py`: classify runtime errors and provide recharge guidance for insufficient credits
- `scripts/inspect_openapi.py`: extract SSE/polling endpoint pair, contract, and payload template
- `references/open-platform-api.md`: SSE-first lifecycle, fallback polling, retry baseline
- `references/video-model-endpoints.md`: current video endpoint snapshot from `docs.json`
