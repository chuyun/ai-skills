# skills.video Open Platform API Contract

## Scope
Use this reference for generation flows that create videos or images through `open.skills.video`.

## Base URL and endpoint shape
- Platform base URL: `https://open.skills.video/api/v1`
- Default generation endpoint pattern: `POST /generation/sse/{provider}/{model}`
- Polling create endpoint pattern (fallback): `POST /generation/{provider}/{model}`
- Task query endpoint pattern (fallback): `GET /generation/{id}`

Some model-scoped OpenAPI files use:
- `servers[0].url = /api`
- paths prefixed with `/v1/...`

Treat these as equivalent path representations under the same host.

## Authentication
Use bearer auth on every API call:

```http
Authorization: Bearer <API_KEY>
```

Security scheme in OpenAPI: `bearerAuth` (`type: http`, `scheme: bearer`).

## Async generation lifecycle
1. Submit generation with SSE endpoint `POST /generation/sse/{provider}/{model}`.
2. Read stream events (`text/event-stream`) until terminal event.
3. Fall back to polling only if SSE cannot connect, disconnects early, or does not reach terminal state.
4. Poll fallback result with `GET /generation/{id}` (or model-scoped equivalent path).
5. Stop polling at terminal status.

## Status values from OpenAPI
### PredictionStatus
- `starting`
- `processing`
- `succeeded`
- `failed`
- `canceled`

### QueueState
- `IN_QUEUE`
- `IN_PROGRESS`
- `COMPLETED`
- `FAILED`
- `CANCELED`

## Error response shape
`ErrorResponse` defines:
- `message` (`string`)

Common documented HTTP errors on generation endpoints:
- `400`
- `401`
- `404`
- `422`

## Retry baseline
- Retry only transient failures (`429`, `5xx`, transport timeouts, connection resets).
- Do not retry unchanged payloads on validation/auth errors (`400`, `401`, `404`, `422`).
- Use bounded exponential backoff when retrying.
- Preserve SSE-first behavior on retries; use polling only as fallback.

## Unknowns
OpenAPI in this workspace does not declare concrete rate-limit windows or global timeout budgets.
Mark both as unknown unless an endpoint-specific OpenAPI explicitly documents them.
