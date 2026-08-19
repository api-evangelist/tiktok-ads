---
name: Build and share a TikTok custom audience
description: Upload a customer file as a TikTok Custom Audience, expand it into a Lookalike, and share it with another ad account.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Dmp Custom_audience File Upload
  - Dmp Custom_audience Create
  - Dmp Custom_audience List
  - Dmp Custom_audience Get
  - Dmp Custom_audience Lookalike Create
  - Dmp Custom_audience Share
  - Dmp Saved_audience Create
---

# Build and share a TikTok custom audience

## Steps

1. **Upload the file.** `Dmp Custom_audience File Upload` —
   `POST /dmp/custom_audience/file/upload/`. Returns a file path/id used in the next step.
2. **Create the audience.** `Dmp Custom_audience Create` —
   `POST /dmp/custom_audience/create/`, referencing the uploaded file.
3. **Wait for it to become usable.** `Dmp Custom_audience Get` —
   `GET /dmp/custom_audience/get/`. An audience is not editable or targetable while it is
   still processing.
4. **Expand it.** `Dmp Custom_audience Lookalike Create` —
   `POST /dmp/custom_audience/lookalike/create/`.
5. **Save a reusable targeting set.** `Dmp Saved_audience Create` —
   `POST /dmp/saved_audience/create/`.
6. **Share across accounts.** `Dmp Custom_audience Share` —
   `POST /dmp/custom_audience/share/`. Sharing is a separate permission
   (scope 32, "Share Custom Audiences") from creating (scope 31).
7. **Enumerate what exists.** `Dmp Custom_audience List` —
   `GET /dmp/custom_audience/list/`.

## Rules an agent must follow

- **Handle PII deliberately.** The upload step carries hashed customer identifiers.
  Confirm the operator intends this before calling it, and never log the payload.
- **Size floor.** An audience created with fewer than ~1,000 matches lands at size 0 and
  becomes uneditable. TikTok returns `40002` with "Audience is not editable, please check
  audience status" — the fix is a larger source list, not a retry.
- **Check permissions before writing.** Reading audiences is scope 30; creating and
  updating is scope 31; sharing is scope 32. The token carries only what the ADVERTISER
  approved, which can be narrower than what the app requested. `40001` means the token
  lacks the permission — re-authorization is required, not a retry.
- **No idempotency.** A timed-out create may still have created the audience. Call
  `Dmp Custom_audience List` and reconcile before creating again.
- `code == 0` is the only success; HTTP 200 proves nothing.
