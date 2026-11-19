<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Apple App Store API

[![SocQ Apple App Store API](https://img.shields.io/badge/API-SocQ%20Apple%20App%20Store%20API-F64C31)](https://socq.ai/apis/apple-app-store?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/apple-app-store/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/apple-app-store-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/apple-app-store-api/actions/workflows/check.yml)

3 endpoints for public Apple App Store data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Apple App Store APIs](https://socq.ai/apis/apple-app-store?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api) · [Documentation](https://docs.socq.ai/api-manual/apple-app-store/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Apple App Store App Detail API](#apple-app-store-app-detail-api) | `apple-app-store/app-detail` | Retrieve public Apple App Store listing details from known apps.apple.com URLs, returning normalized app identity, descriptive metadata, developer and category context, visible ratings, pricing, media, and collection time. | 0.15 credits ($0.00075) per result | 2026-08-17 |
| [Apple App Store App Reviews API](#apple-app-store-app-reviews-api) | `apple-app-store/app-reviews` | Collect public Apple App Store reviews from known apps.apple.com app URLs, returning normalized review text, ratings, reviewer details, timestamps, and app references. | 0.15 credits ($0.00075) per result | 2026-08-17 |
| [Apple App Store Rankings API](#apple-app-store-rankings-api) | `apple-app-store/rankings` | Collect public Apple App Store category rankings from known apps.apple.com chart URLs, returning normalized rank, app identity, developer, category, rating, review, price, and media fields. | 0.15 credits ($0.00075) per result | 2026-08-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/apple-app-store-api.git
cd apple-app-store-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs apple-app-store/app-detail

# Python 3.8+ (standard library only)
python3 examples/python/socq.py apple-app-store/app-detail payloads/app-detail.json

# cURL + jq
bash examples/curl/socq.sh apple-app-store/app-detail '{"urls":["https://apps.apple.com/us/app/chatgpt/id6448311069"]}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Apple App Store App Detail API

Retrieve public Apple App Store listing details from known apps.apple.com URLs, returning normalized app identity, descriptive metadata, developer and category context, visible ratings, pricing, media, and collection time.

- **Request:** `POST https://api.socq.ai/v1/apple-app-store/app-detail`
- **publicId:** `apple-app-store/app-detail`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-17
- **Example body:** [`payloads/app-detail.json`](payloads/app-detail.json)

Fetch normalized apps data from Apple App Store.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/apple-app-store/app-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://apps.apple.com/us/app/chatgpt/id6448311069"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/apple-app-store/app-detail](https://docs.socq.ai/api-manual/apple-app-store/app-detail) · Landing page: [socq.ai/apis/apple-app-store/app-detail](https://socq.ai/apis/apple-app-store/app-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api)

### Apple App Store App Reviews API

Collect public Apple App Store reviews from known apps.apple.com app URLs, returning normalized review text, ratings, reviewer details, timestamps, and app references.

- **Request:** `POST https://api.socq.ai/v1/apple-app-store/app-reviews`
- **publicId:** `apple-app-store/app-reviews`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-17
- **Example body:** [`payloads/app-reviews.json`](payloads/app-reviews.json)

Fetch normalized reviews data from Apple App Store.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/apple-app-store/app-reviews" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://apps.apple.com/us/app/chatgpt/id6448311069"
  ],
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/apple-app-store/app-reviews](https://docs.socq.ai/api-manual/apple-app-store/app-reviews) · Landing page: [socq.ai/apis/apple-app-store/app-reviews](https://socq.ai/apis/apple-app-store/app-reviews?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api)

### Apple App Store Rankings API

Collect public Apple App Store category rankings from known apps.apple.com chart URLs, returning normalized rank, app identity, developer, category, rating, review, price, and media fields.

- **Request:** `POST https://api.socq.ai/v1/apple-app-store/rankings`
- **publicId:** `apple-app-store/rankings`
- **Price:** 0.15 credits ($0.00075) per result
- **Launched:** 2026-08-17
- **Example body:** [`payloads/rankings.json`](payloads/rankings.json)

Fetch normalized rankings data from Apple App Store.

Billing is based on successfully returned results. List endpoints reserve credits based on `results_limit` and settle against the actual result count.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/apple-app-store/rankings" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://apps.apple.com/us/iphone/charts/6007"
  ],
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/apple-app-store/rankings](https://docs.socq.ai/api-manual/apple-app-store/rankings) · Landing page: [socq.ai/apis/apple-app-store/rankings](https://socq.ai/apis/apple-app-store/rankings?utm_source=github&utm_medium=readme&utm_campaign=socq-apple-app-store-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Apple App Store data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
