---
name: Manage TikTok Business Center assets and members
description: Enumerate Business Centers, group and assign assets to members or partners, and check balances and transfers.
api: openapi/tiktok-ads-marketing-api-openapi.yml
base_url: https://business-api.tiktok.com/open_api/v1.3
operations:
  - Bc Get
  - Bc Asset Get
  - Bc Asset_group Create
  - Bc Asset Assign
  - Bc Asset Unassign
  - Bc Member Get
  - Bc Member Invite
  - Bc Balance Get
  - Bc Transfer
  - Bc Transaction Get
---

# Manage TikTok Business Center assets and members

Business Center is the agency-scale layer: ad accounts, pixels, catalogs and identities are
assets held by a BC and assigned to members or partner organisations. It is the largest
single capability group in this API — 33 of the 202 published operations.

## Steps

1. **Enumerate Business Centers.** `Bc Get` — `GET /bc/get/`. Note `bc_id`; it replaces
   `advertiser_id` as the scoping key for every call below.
2. **List assets.** `Bc Asset Get` — `GET /bc/asset/get/`.
3. **Group assets.** `Bc Asset_group Create` — `POST /bc/asset_group/create/` so
   permissions are granted by group rather than one asset at a time.
4. **List members.** `Bc Member Get` — `GET /bc/member/get/`; invite with
   `Bc Member Invite` — `POST /bc/member/invite/`.
5. **Assign.** `Bc Asset Assign` — `POST /bc/asset/assign/`. Reverse with
   `Bc Asset Unassign` — `POST /bc/asset/unassign/`.
6. **Money.** `Bc Balance Get` — `GET /bc/balance/get/`, `Bc Transfer` —
   `POST /bc/transfer/`, and `Bc Transaction Get` — `GET /bc/transaction/get/`.

## Rules an agent must follow

- **`Bc Transfer` moves real money between accounts.** Treat it as an irreversible,
  human-confirmed action. There is no idempotency key on this API, so a retried transfer
  after a timeout can move the funds twice. Always call `Bc Transaction Get` to establish
  what actually happened before retrying anything on this path — never retry blind.
- **Unassign is destructive.** `Bc Asset Unassign` and `Bc Asset Admin Delete` remove
  access for real people. Confirm before calling.
- **Scope 15 ("Business Center") is required** and sits under Ad Account Management. A
  `40001` here means the advertiser did not grant BC access.
- Business Center endpoints are **not** in the sandbox's supported set. There is no safe
  rehearsal environment for this flow.
- `code == 0` is the only success; HTTP 200 is not.
