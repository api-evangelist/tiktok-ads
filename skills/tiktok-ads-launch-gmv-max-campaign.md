---
name: Launch a TikTok GMV Max shop campaign
description: Set up a GMV Max campaign for a TikTok Shop, using store and identity discovery, bid recommendation, and GMV Max reporting.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Gmv_max Store List
  - Gmv_max Store Shop_ad_usage_check
  - Gmv_max Identity Get
  - Gmv_max Bid Recommend
  - Campaign Gmv_max Create
  - Campaign Gmv_max Info
  - Gmv_max Report Get
  - Gmv_max Video Get
---

# Launch a TikTok GMV Max shop campaign

GMV Max automates shop advertising against a TikTok Shop. Discovery comes before creation:
the store, the identity and the eligible videos all have to be resolved first.

## Steps

1. **Find the stores.** `Gmv_max Store List` — `GET /gmv_max/store/list/`.
2. **Check eligibility.** `Gmv_max Store Shop_ad_usage_check` —
   `GET /gmv_max/store/shop_ad_usage_check/`. A store already consumed by another shop ad
   cannot be used again.
3. **Resolve the posting identity.** `Gmv_max Identity Get` — `GET /gmv_max/identity/get/`.
4. **Find usable creative.** `Gmv_max Video Get` — `GET /gmv_max/video/get/`.
5. **Get a bid recommendation.** `Gmv_max Bid Recommend` — `GET /gmv_max/bid/recommend/`.
   Use this rather than guessing a target ROAS.
6. **Create the campaign.** `Campaign Gmv_max Create` — `POST /campaign/gmv_max/create/`.
7. **Confirm it.** `Campaign Gmv_max Info` — `GET /campaign/gmv_max/info/`.
8. **Report on it.** `Gmv_max Report Get` — `GET /gmv_max/report/get/`.

## Rules an agent must follow

- **GMV Max reporting has its own limits**, separate from ordinary reporting:
  8 QPS / 240 QPM / 20,000 QPD at Basic, rising to 20 / 600 / 50,000 at Premium.
  Budget report pulls against the QPD figure, not the QPS one.
- **Discovery before creation, always.** Creating against a store that fails the usage
  check, or an identity the account cannot post as, fails at create time with a `40002`
  that names the field. Read the message; do not retry.
- **No idempotency.** If create times out, call `Campaign Gmv_max Info` before creating
  again.
- Ids are strings. `code == 0` is the only success. Keep `request_id`.
