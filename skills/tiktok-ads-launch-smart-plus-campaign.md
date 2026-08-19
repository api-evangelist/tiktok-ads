---
name: Launch an Upgraded Smart+ campaign on TikTok
description: Create an Upgraded Smart+ campaign, ad group and ad on TikTok Ads, then check review status and early creative performance.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Advertiser Info
  - Smart_plus Campaign Create
  - Smart_plus Adgroup Create
  - Smart_plus Ad Create
  - Smart_plus Ad Review_info
  - Smart_plus Campaign Get
  - Smart_plus Material_report Overview
  - Smart_plus Ad Status Update
---

# Launch an Upgraded Smart+ campaign

Upgraded Smart+ is TikTok's automated campaign product. The object hierarchy is the
same as manual campaigns — campaign, then ad group, then ad — but TikTok makes the
targeting and creative decisions.

## Before you start

- Auth: send the long-lived token in the `Access-Token` **request header**. Not
  `Authorization: Bearer`, and not a query parameter, whatever the spec fragment says.
  See `authentication/tiktok-ads-authentication.yml`.
- Every path ends in a trailing slash and the slash is required. Dropping it returns
  `404 page not found`.
- Get `advertiser_id` first with `Advertiser Info` (`GET /advertiser/info/`). It scopes
  every call below.
- **Read the body, not the status code.** TikTok answers HTTP 200 for errors too. Treat
  a response as successful only when `code == 0` (or `20001` for partial success).
  See `errors/tiktok-ads-error-codes.yml`.

## Steps

1. **Confirm the account.** `Advertiser Info` — `GET /advertiser/info/` with
   `advertiser_ids`. Read currency and timezone; budgets are expressed in the account
   currency.
2. **Create the campaign.** `Smart_plus Campaign Create` —
   `POST /smart_plus/campaign/create/`. Record `campaign_id` from `data`.
3. **Create the ad group.** `Smart_plus Adgroup Create` —
   `POST /smart_plus/adgroup/create/`, referencing the `campaign_id`. Record
   `adgroup_id`.
4. **Create the ad.** `Smart_plus Ad Create` — `POST /smart_plus/ad/create/`,
   referencing the `adgroup_id`.
5. **Check the review outcome.** `Smart_plus Ad Review_info` —
   `GET /smart_plus/ad/review_info/`. Ads enter review before they deliver; do not
   treat a successful create as a live ad.
6. **Confirm the campaign state.** `Smart_plus Campaign Get` —
   `GET /smart_plus/campaign/get/`.
7. **Read early creative performance.** `Smart_plus Material_report Overview` —
   `GET /smart_plus/material_report/overview/`.
8. **Pause or resume when needed.** `Smart_plus Ad Status Update` —
   `POST /smart_plus/ad/status/update/`.

## Rules an agent must follow

- **No idempotency.** There is no idempotency key on this API. If step 2, 3 or 4 times
  out you cannot safely retry — a blind retry creates a duplicate campaign. Instead,
  re-read with `Smart_plus Campaign Get` and only create if nothing was written. Return
  code `40050` ("Duplicated requests have been sent") is TikTok rejecting duplicates
  server-side on its own terms, not a key you control.
- **Concurrency ceiling on Smart+ writes.** Keep `Smart_plus Ad Create`,
  `Smart_plus Ad Update` and `Smart_plus Ad Status Update` to **one operation per five
  seconds** for a single ad, or TikTok raises a concurrency error. This is TikTok's own
  published constraint for both the REST API and the MCP tools.
- **Rate limits are invisible until you hit them.** No `X-RateLimit-*` and no
  `Retry-After`. Code `40100` means throttled: back off a second for QPS, five minutes
  for QPM, until 00:00 UTC for QPD. `/ad/create/` is 5 QPS at the default Basic level.
- **Ids are strings.** `advertiser_id`, `campaign_id`, `adgroup_id` — keep them as
  strings; several exceed the JavaScript safe integer range.
- **Test on the sandbox host, not a test key.** There is no test-mode credential. Point
  at `https://sandbox-ads.tiktok.com/open_api/v1.3` instead. Note that Smart+ endpoints
  are **not** in the sandbox's 21 supported endpoints — see
  `sandbox/tiktok-ads-sandbox.yml` before assuming a dry run is possible.
- Log `request_id` from every response; it is what TikTok support asks for.
