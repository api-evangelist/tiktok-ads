---
name: Onboard a product catalog to TikTok
description: Create a TikTok catalog, attach a product feed, upload products, bind an event source, and verify catalog health.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Catalog Create
  - Catalog Get
  - Catalog Feed Create
  - Catalog Product File
  - Catalog Feed Log
  - Catalog Overview
  - Catalog Eventsource Bind
  - Catalog Available_country Get
---

# Onboard a product catalog to TikTok

## Steps

1. **Check where you can sell.** `Catalog Available_country Get` —
   `GET /catalog/available_country/get/`.
2. **Create the catalog.** `Catalog Create` — `POST /catalog/create/`. Record `catalog_id`.
3. **Attach a feed.** `Catalog Feed Create` — `POST /catalog/feed/create/` with the feed
   URL and schedule.
4. **Or push a product file directly.** `Catalog Product File` —
   `POST /catalog/product/file/`.
5. **Read ingestion results.** `Catalog Feed Log` — `GET /catalog/feed/log/`. Feed
   ingestion is asynchronous; a 200 on create says the feed was accepted, not that products
   landed.
6. **Verify health.** `Catalog Overview` — `GET /catalog/overview/`, and `Catalog Get` —
   `GET /catalog/get/`.
7. **Bind the event source.** `Catalog Eventsource Bind` —
   `POST /catalog/eventsource/bind/` so catalog ads can attribute conversions to a pixel
   or app.

## Rules an agent must follow

- **Catalog writes are the tightest limits on the API.** At the default Basic level:
  `/catalog/product/file/` and `/catalog/product/upload/` are 5 QPS / 300 QPM / 432,000 QPD;
  `/catalog/product/update/` and `/catalog/product/delete/` are 2 QPS / 120 QPM / 172,800 QPD.
  Batch products into a file rather than looping single-product calls.
- **Feed acceptance is not feed success.** Always read `Catalog Feed Log` before telling
  an operator the catalog is live.
- **`40001` on a catalog id is a permissions answer, not a transient error.** The message
  is literally "You don't have permission to operate this catalog:{catalog_id}." Do not
  retry; check DPA Catalog Management scope (9) and Product Management (91).
- **No idempotency.** Re-posting a product file after a timeout may double-write. Read
  `Catalog Overview` first.
- Catalog endpoints are **not** available in the sandbox — the sandbox supports 21
  endpoints and none of them are catalog endpoints. Plan for production testing on a
  disposable catalog.
- `code == 0` is the only success.
