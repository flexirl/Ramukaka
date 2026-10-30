# Ramu Kaka — API Documentation (Proposed Contract)

| Field | Value |
|---|---|
| Backend | Supabase: Auth, PostgREST, Postgres functions (RPC), Edge Functions (CONFIRMED, `[PDF §7]`) |
| Status | **PROPOSED, not implemented** |

> This document is the *planned* interface between clients and the backend. Names and payloads may change in Level 2 and Level 3. Endpoints that depend on open decisions cite the `PA-xx` item.

---

## 1. Access Patterns

| Pattern | Used for | Protection |
|---|---|---|
| **Supabase Auth** | OTP sign-in, sessions | Supabase Auth + SMS provider (**PA-22**) |
| **Direct table reads** (PostgREST via supabase-js) | Catalog, own orders, own wallet history, dashboards lists | RLS |
| **Edge Functions** `POST /functions/v1/<name>` | AI, quotes, order placement, wallet credits, notifications, any third-party call | JWT + role check inside the function |
| **Postgres functions (RPC)** `POST /rest/v1/rpc/<name>` | Atomic transactional operations called by Edge Functions or clients | `SECURITY DEFINER` + explicit role checks |
| **Realtime** | Order status, new offers | RLS-filtered channels |
| **Storage** | Product images, attendance photos | Bucket policies; signed URLs for private files |

**Rule (BR-21):** clients never INSERT or UPDATE orders, prices, wallet entries, assignments or COD records directly. These changes go only through the functions listed below.

### Common conventions
- Auth header: `Authorization: Bearer <supabase JWT>`.
- Money: integer paise.
- Errors: `{ "error": { "code": "STORE_CLOSED", "message": "...", "message_key": "errors.store_closed" } }`. `message_key` lets clients show localized text.
- Idempotency: write endpoints accept an `Idempotency-Key` header.

### Standard error codes

| Code | Meaning | Rule |
|---|---|---|
| `UNAUTHENTICATED` / `FORBIDDEN` | Missing or invalid session, or wrong role | BR-07, BR-23 |
| `OUT_OF_SERVICE_AREA` | Address not served | BR-02 |
| `STORE_CLOSED` | Store not accepting orders | BR-09 |
| `OUT_OF_STOCK` | Insufficient stock | BR-09 |
| `QUOTE_EXPIRED` / `QUOTE_CHANGED` | Price or stock changed since the quote | BR-12, BR-21 |
| `ORDER_ALREADY_ASSIGNED` | Another DP accepted first | BR-03 |
| `INVALID_STATUS_TRANSITION` | Status update out of sequence | BR-11 |
| `NOT_PERMITTED_FOR_STORE` | Manager acting outside their store or permissions | BR-01, BR-20 |
| `COUPON_INVALID` | Coupon not applicable | **PA-04** |
| `SURGE_LIMIT_EXCEEDED` | Surge above the admin limit | BR-19 |

## 2. Authentication

| Operation | Method | Notes |
|---|---|---|
| Request OTP | `supabase.auth.signInWithOtp({ phone })` | FR-CU-01. Rate limited |
| Verify OTP | `supabase.auth.verifyOtp({ phone, token, type: 'sms' })` | Returns a session |
| Get my profile and roles | `GET profiles?id=eq.me` + `user_roles` | Routes the user to the correct panel |
| Update profile / language | `PATCH profiles` (own row, safe fields only) | FR-CU-03 |
| Sign out | `supabase.auth.signOut()` | |

DP login method is **PA-15**. Account deletion is **PA-45**.

## 3. Customer APIs

| # | Endpoint | Type | Req ID | Description |
|---|---|---|---|---|
| C1 | `addresses` CRUD | Table (RLS: own) | FR-CU-02 | Add, edit, list and delete own addresses |
| C2 | `check-serviceability` | Edge Fn | BR-02 | Input: address_id or lat/lng. Output: serving store, or `OUT_OF_SERVICE_AREA` |
| C3 | `categories`, `products` read | Table (RLS: active) | FR-CU-07 | Catalog for the serving store |
| C4 | `quote-order` | Edge Fn | FR-CU-06, BR-12 | Server-side price breakdown (see below) |
| C5 | `place-order` | Edge Fn → RPC | FR-CU-04, BR-04 | Creates the order from a confirmed quote |
| C6 | `orders` read + Realtime | Table | FR-CU-09 | Own orders, history and live status |
| C7 | `cancel-order` | Edge Fn | **PA-16** | Allowed states and actors pending |
| C8 | `wallet` / `wallet_transactions` read | Table (own) | FR-CU-11 | Balance and history |
| C9 | `list-offers` | Edge Fn / view | FR-CU-12 | Coupons and offers the customer is eligible for |
| C10 | `reviews` create | Edge Fn | FR-CU-13 | Eligibility **PA-25** |
| C11 | `complaints` create | Edge Fn | **PA-26** | Intake channel pending |
| C12 | `get-delivery-contact` | Edge Fn | FR-CU-14 | Returns DP phone, or a masked number (**PA-24**) |
| C13 | `register-device` | Table | FR-CU-10 | Push token |

### C4 `quote-order` example
Request:
```json
{
  "address_id": "uuid",
  "items": [{ "product_id": "uuid", "quantity": 1 }],
  "coupon_code": "OPTIONAL",
  "use_wallet": false,
  "channel": "manual"
}
```
Response:
```json
{
  "quote_id": "uuid",
  "expires_at": "2026-10-30T10:05:00Z",
  "store": { "id": "uuid", "name": "Jamui Store 1", "is_open": true },
  "items": [{ "product_id": "uuid", "name": "20 L Water Jar", "unit_price_paise": 0, "quantity": 1, "line_total_paise": 0 }],
  "subtotal_paise": 0,
  "delivery_charge_paise": 0,
  "surge_paise": 0,
  "discount_paise": 0,
  "wallet_used_paise": 0,
  "total_paise": 0,
  "payment_method": "cod",
  "address_summary": "Home – ..."
}
```
> Amounts above are placeholders. No prices are defined yet (**PA-02**). `use_wallet` depends on **PA-27**.

### C5 `place-order` example
Request: `{ "quote_id": "uuid" }` with header `Idempotency-Key: <uuid>`.
Response: `{ "order_id": "uuid", "order_number": "RK-000123", "status": "placed" }`.
Server re-checks store open state, stock, service area and quote expiry (BR-09, ADR-03). Then it reserves stock and starts dispatch.

## 4. AI Assistant API

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| A1 | `ai-session-start` | Edge Fn | FR-AI-03 |
| A2 | `ai-turn` | Edge Fn | FR-AI-01…08 |
| A3 | `ai-session-end` | Edge Fn | FR-AI-07 |

### A2 `ai-turn`
Request (multipart or JSON; format depends on **PA-23**):
```json
{
  "session_id": "uuid",
  "input": { "type": "audio", "mime": "audio/aac", "data_base64": "..." },
  "language": "hi"
}
```
Response:
```json
{
  "reply_text": "Ek 20 litre jar, Home address par, ₹__. Order kar doon?",
  "reply_audio_url": "signed-url-or-null",
  "state": "awaiting_confirmation",
  "quote": { "quote_id": "uuid", "total_paise": 0 },
  "order": null,
  "fallback_to_manual": false
}
```
- `state` ∈ `collecting` | `clarifying` | `awaiting_confirmation` | `order_placed` | `fallback`.
- The model's server-side tools: `search_products`, `get_customer_addresses`, `get_quote` (same as C4), `place_order` (same as C5; allowed only after confirmation), `get_order_status`.
- Every call writes to `ai_usage_logs` (FR-AI-07). Caps are **PA-35**.

## 5. Delivery Partner APIs

| # | Endpoint | Type | Req ID | Description |
|---|---|---|---|---|
| D1 | `set-duty-status` | Edge Fn | FR-DP-02 | `{ "status": "on_duty" }`. Creates or closes the attendance record |
| D2 | `upload-attendance-photo` | Storage (private) + Edge Fn | FR-DP-03 | Uploads photo and links it to attendance. Location capture **PA-06** |
| D3 | `my-offers` read + Realtime | Table (own) | FR-DP-04 | Open offers |
| D4 | `accept-order` | Edge Fn → RPC (atomic) | FR-DP-05/06, BR-03 | `{ "order_id" }` → `200 { assigned: true }` or `409 ORDER_ALREADY_ASSIGNED` |
| D5 | `reject-order` | Edge Fn | FR-DP-05 | Records the rejection |
| D6 | `my-orders` read | Table | FR-DP-07 | Assigned orders with customer details |
| D7 | `update-order-status` | Edge Fn | FR-DP-09, BR-11 | `{ "order_id", "to_status": "picked_up" }` |
| D8 | `record-cod` | Edge Fn | FR-DP-10 | `{ "order_id", "collected_paise" }`. Usually sent together with `delivered` |
| D9 | `report-issue` | Edge Fn | FR-DP-11 | `{ "order_id", "type", "reason", "note", "photo_path?" }` (**PA-17**) |
| D10 | `my-performance` | View / RPC | FR-DP-12 | Attendance, history, counts |

## 6. Store Manager APIs (all scoped to the manager's store, BR-01)

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| M1 | `store-orders` read + Realtime | Table/view | FR-SM-01 |
| M2 | `store-delivery-partners` read | View | FR-SM-02, FR-SM-15 |
| M3 | `assign-order` | Edge Fn → RPC | FR-SM-02 (reassign **PA-40**) |
| M4 | `set-store-open` | Edge Fn | FR-SM-04 |
| M5 | `set-surge` | Edge Fn | FR-SM-05 (limits **PA-05**) |
| M6 | `categories` CRUD | Table (permission-gated) | FR-SM-06 |
| M7 | `products` CRUD + image upload | Table + Storage | FR-SM-07/08/09 |
| M8 | `adjust-stock` | Edge Fn / RPC | FR-SM-10 |
| M9 | `store-reports` | RPC/view | FR-SM-03, FR-SM-11 |
| M10 | `cod-reconciliation` read + `settle-cod` | View + Edge Fn | FR-SM-12 (**PA-08**) |
| M11 | `coupons` / `cashback-campaigns` (store scope) | Edge Fn | FR-SM-13 (**PA-32**) |
| M12 | `reviews`, `complaints` manage | Edge Fn | FR-SM-14 |
| M13 | `verify-attendance` | Edge Fn | FR-SM-15 (**PA-06**) |

Price edits through M7 may need to go through an Edge Function so admin pricing limits can be enforced (**PA-02**, **PA-32**).

## 7. Super Admin APIs

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| S1 | `stores` create/update/suspend | Edge Fn | FR-SA-01/03 |
| S2 | `managers` create/approve/suspend, assign to store | Edge Fn | FR-SA-03 |
| S3 | `delivery-partners` manage | Edge Fn | FR-SA-01 (onboarding **PA-14**) |
| S4 | `customers` list/view/suspend | Table/Edge Fn | FR-SA-01 |
| S5 | `store-settings` / `platform-settings` update | Edge Fn | FR-SA-04/05/12 |
| S6 | `set-manager-permissions` | Edge Fn | FR-SA-12 (**PA-32**) |
| S7 | `coupons` / `cashback-campaigns` (platform) | Edge Fn | FR-SA-06 |
| S8 | `wallet-credit` | Edge Fn → RPC | FR-SA-07, BR-18 |
| S9 | `finance-reports` (wallet, COD, promo spend) | RPC/views | FR-SA-08 |
| S10 | `performance-compare` | RPC/views | FR-SA-02/09 |
| S11 | `reviews` / `complaints` platform view | Table/views | FR-SA-10 |
| S12 | `ai-usage-report` | RPC/view | FR-SA-11, KPI-09 |
| S13 | `audit-log` read | Table | NFR-SEC-07 |

### S8 `wallet-credit` example
```json
{
  "customer_id": "uuid",
  "amount_paise": 0,
  "type": "refund",
  "reason": "Damaged jar – complaint #123",
  "reference": { "type": "complaint", "id": "uuid" }
}
```
Header `Idempotency-Key` is required. The response includes the ledger entry and the new balance. Approval workflow is **PA-44**.

## 8. Realtime Channels (proposed)

| Channel | Subscribers | Events |
|---|---|---|
| `order:<id>` | Customer, assigned DP, store manager | status changes |
| `store:<id>:orders` | Store manager | new/updated orders |
| `dp:<id>:offers` | Delivery partner | new offer, offer withdrawn |

## 9. Versioning
Edge Functions are versioned by path suffix if a breaking change is needed (e.g. `quote-order-v2`). Generated TypeScript types are shared through `packages/shared-types` (RECOMMENDATION).
