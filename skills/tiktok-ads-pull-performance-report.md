---
name: Pull a TikTok Ads performance report
description: Retrieve TikTok advertising performance either synchronously for small pulls or through the asynchronous report task pattern for large ones.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Advertiser Info
  - Report Integrated Get
  - Report Task Create
  - Report Task Check
  - Report Task Cancel
---

# Pull a TikTok Ads performance report

TikTok exposes two reporting paths and picking the wrong one is the usual cause of
timeouts and throttling.

## Choose the path first

| Situation | Use |
|---|---|
| Small pull, a page or two, need it now | `Report Integrated Get` (synchronous) |
| Wide date range, many dimensions, bulk export | `Report Task Create` → `Report Task Check` (asynchronous) |

## Synchronous

1. `Advertiser Info` — `GET /advertiser/info/` to confirm the account, its currency and
   its timezone. Reporting dates are interpreted in the account timezone.
2. `Report Integrated Get` — `GET /report/integrated/get/`. Set `report_type`,
   `data_level`, `dimensions`, `metrics`, `start_date`, `end_date`, `page`, `page_size`.
3. Paginate on `data.page_info` (`page`, `page_size`, `total_page`, `total_number`).
   There are no cursors and no `Link` header.

## Asynchronous

1. `Report Task Create` — `POST /report/task/create/`. Record the task id.
2. `Report Task Check` — `GET /report/task/check/`. Poll until `status` is `SUCCESS`.
3. Download the produced file from the location the check response returns.
4. `Report Task Cancel` — `POST /report/task/cancel/` if the request is no longer wanted.

The async report task is capped hard at **2 QPS / 60 QPM / 4,500 QPD at every rate-limit
level** — Ultimate buys nothing here. Poll on a schedule, not in a tight loop.

## Rules an agent must follow

- **`code == 0` is the only success.** HTTP 200 with `"code": 40002` is a failure. Never
  branch on the HTTP status alone.
- **Batch instead of iterating.** Read endpoints take arrays of ids in `filtering`.
  TikTok's published guidance is one request with many ids, not many requests with one id
  each — limits are counted per request.
- **Back off on `40100`.** QPM throttling costs five minutes; QPD throttling costs the rest
  of the UTC day and resets at 00:00:00 UTC+0.
- **Sandbox reporting is mock data.** `https://sandbox-ads.tiktok.com/open_api/v1.3/report/integrated/get/`
  returns synthetic values only, restricted to 2020-12-08 → 2020-12-19, `service_type`
  `AUCTION`, `report_type` `BASIC`/`AUDIENCE`, ordering unsupported, and async reports are
  not mocked at all. Never validate business logic against sandbox numbers.
- Keep `request_id` from every response for support.
