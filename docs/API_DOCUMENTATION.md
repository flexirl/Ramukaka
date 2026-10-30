# Ramu Kaka — API Documentation (Proposed Contract)

| Field | Value |
|---|---|
| Backend | Supabase: Auth, PostgREST, Postgres functions (RPC), Edge Functions (CONFIRMED, `[PDF §7]`) |
| Status | **PROPOSED, not implemented** |
| Last updated | 2026-10-30 (single mobile app decision) |

> There is **one client**: the Ramu Kaka mobile app, which contains all four panels (BR-27). The endpoints below are grouped by the panel that uses them. **Every endpoint lists the roles allowed to call it, and the backend rejects every other role.** All panels ship in the same app, so any endpoint can be discovered (NFR-SEC-14).
>
> This document is the *planned* interface between the app and the backend. Names and payloads may change in Level 2 and Level 3. Endpoints that depend on open decisions cite the `PA-xx` item.

---

## 1. Access Patterns

| Pattern | Used for | Protection |
|---|---|---|
| **Supabase Auth** | OTP sign-in for all roles (BR-26), sessions | Supabase Auth + SMS provider (**PA-22**) |
| **Direct table reads** (PostgREST via supabase-js) | Catalog, own orders, own wallet history, notifications, Manager/Admin panel lists | RLS (role + active status + store scope) |
| **Direct table writes** | **Only** low-risk own data: own addresses, own device tokens, marking own notifications read | RLS |
| **Edge Functions** `POST /functions/v1/<name>` | Everything else that writes: AI, quotes, orders, delivery requests, catalog, stock, settings, wallet, accounts, notifications | JWT + explicit role/status/scope check inside the function (NFR-SEC-08, NFR-SEC-11) |
| **Postgres functions (RPC)** | Atomic transactional operations, called by Edge Functions | `SECURITY DEFINER` + explicit role checks |
| **Realtime** | Order status, delivery requests | RLS-filtered channels |
| **Storage** | Product images, attendance and issue photos | Bucket policies; signed URLs for private files |

**Rule (BR-21, ADR-01, DR-11):** clients never INSERT or UPDATE orders, quotes, prices, stock, categories, products, wallet entries, assignments, COD records, settings, roles or account status directly.

### Common conventions
- Auth header: `Authorization: Bearer <supabase JWT>`.
- Money: integer paise.
- Status values: lowercase snake_case (`out_for_delivery`).
- Errors: `{ "error": { "code": "STORE_CLOSED", "message": "...", "message_key": "errors.store_closed" } }`. `message_key` lets clients show localized text.
- Idempotency: write endpoints accept an `Idempotency-Key` header. It is **required** for `place-order` and `wallet-credit`.

### Server-function security checklist (applies to every Edge Function)
1. Verify the JWT.
2. Load the caller's roles and **check the account is active** (and, for managers, that the store assignment is active) (NFR-SEC-08).
3. Check that the caller's role is allowed for this endpoint, and check the store scope (BR-01, BR-28). Never rely on the app having hidden the screen (NFR-SEC-14).
4. Validate every input. Never trust amounts sent by the client (BR-21).
5. Only then use the service-role key for privileged queries (NFR-SEC-11).
6. Write an audit entry for money, price, stock and account actions (NFR-SEC-07).

### Standard error codes

| Code | Meaning | Rule |
|---|---|---|
| `UNAUTHENTICATED` / `FORBIDDEN` | Missing or invalid session, or wrong role | BR-07, BR-23 |
| `ACCOUNT_SUSPENDED` | Caller's account or store assignment is suspended | DR-08, NFR-SEC-08 |
| `RATE_LIMITED` | Too many OTP or AI requests | NFR-SEC-05, NFR-SEC-10 |
| `OUT_OF_SERVICE_AREA` | Address not served | BR-02 |
| `STORE_CLOSED` | Store not accepting orders | BR-09 |
| `OUT_OF_STOCK` | Insufficient stock | BR-09 |
| `QUOTE_EXPIRED` / `QUOTE_CHANGED` | Price or stock changed since the quote | BR-12, BR-21 |
| `QUOTE_NOT_CONFIRMABLE` | Quote not owned by the caller, already used, or not the session's pending quote | DR-10 |
| `ORDER_ALREADY_ASSIGNED` | Another DP accepted first | BR-03 |
| `INVALID_STATUS_TRANSITION` | Status update out of sequence | BR-11 |
| `NOT_PERMITTED_FOR_STORE` | Manager acting outside their store or permissions | BR-01, BR-20 |
| `COUPON_INVALID` | Coupon not applicable | **PA-04** |
| `SURGE_LIMIT_EXCEEDED` | Surge above the admin limit | BR-19 |

## 2. Authentication (all roles)

| Operation | Method | Notes |
|---|---|---|
| Request OTP | `supabase.auth.signInWithOtp({ phone })` | FR-CU-01, BR-26. Rate limited (NFR-SEC-05) |
| Verify OTP | `supabase.auth.verifyOtp({ phone, token, type: 'sms' })` | Returns a session |
| Get my access | `get-my-access` (Edge Fn): returns roles, account status, staff approval status and store assignment | Used by the app shell to route to a panel and to re-check access (FR-APP-02, FR-APP-06). Suspended → `ACCOUNT_SUSPENDED`. Read-only: no endpoint lets a user change their own role (FR-APP-04) |
| Update my profile | `update-my-profile` (Edge Fn) | Only `full_name` and `preferred_language` (NFR-SEC-09). Used by customers (FR-CU-03) and, within **PA-14** limits, by DPs (FR-DP-01) |
| Sign out | `supabase.auth.signOut()` | |

Extra DP login checks are **PA-15**. Account deletion is **PA-45**.

## 3. Customer Panel APIs (role: `customer`)

| # | Endpoint | Type | Req ID | Description |
|---|---|---|---|---|
| C1 | `addresses` CRUD | Table (RLS: own) | FR-CU-02 | Add, edit, list and delete own addresses |
| C2 | `check-serviceability` | Edge Fn | BR-02 | Input: address_id or lat/lng. Output: serving store, or `OUT_OF_SERVICE_AREA` |
| C3 | `categories`, `products` read | Table (RLS: active) | FR-CU-07 | Catalog for the serving store |
| C4 | `quote-order` | Edge Fn | FR-CU-06, FR-WP-03, BR-12 | Server-side price breakdown, including coupon and cashback eligibility (see below) |
| C5 | `place-order` | Edge Fn → RPC | FR-CU-04, FR-WP-01, BR-04, DR-10 | Places a COD order for a quote **after the customer's Confirm tap** |
| C6 | `orders` read + Realtime | Table | FR-CU-09 | Own orders, history and live status |
| C7 | `cancel-order` | Edge Fn | **PA-16** | Allowed states and actors pending |
| C8 | `wallets` / `wallet_transactions` read | Table (own) | FR-CU-11, FR-WP-02 | Balance and history |
| C9 | `list-offers` | Edge Fn / view | FR-CU-12 | Promotional offers and coupons the customer is eligible for |
| C10 | `create-review` | Edge Fn | FR-CU-13 | Eligibility **PA-25** |
| C11 | `create-complaint` | Edge Fn | FR-SM-14 | Intake channel and feedback type **PA-26** |
| C12 | `get-delivery-contact` | Edge Fn | FR-CU-14 | Contact details for the assigned DP. Method and masking **PA-24** |
| C13 | `device_tokens` upsert | Table (own) | FR-CU-10 | Push token |
| C14 | `notifications` read + mark read | Table (own) | FR-CU-10 | In-app notification inbox (CU-S19) |

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
Request: `{ "quote_id": "uuid" }` with the required header `Idempotency-Key: <uuid>`.

Response: `{ "order_id": "uuid", "order_number": "RK-000123", "status": "placed", "confirmation_method": "tap" }`.

Server checks:
- The quote belongs to the caller, is `active` and unexpired.
- If the quote came from a voice session, it must be that session's `pending_quote_id` (DR-10).
- Store is still open, stock is still available, address is still in the service area (BR-09, ADR-03).

It then creates the order in one transaction. Stock is handled per **PA-49**. Delivery requests are sent to eligible DPs (**PA-03**).

## 4. AI Assistant API (Customer panel, role: `customer`)

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| A1 | `ai-session-start` | Edge Fn | FR-AI-03 |
| A2 | `ai-turn` | Edge Fn | FR-CU-05, FR-AI-01…08 |
| A3 | `ai-session-end` | Edge Fn | FR-AI-07 |
| — | Confirm by tap | Uses `place-order` (C5) with the session's pending quote | FR-AI-05, DR-10 |

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
  "reply_audio": { "mime": "audio/mpeg", "data_base64": "..." },
  "state": "awaiting_confirmation",
  "quote": { "quote_id": "uuid", "total_paise": 0 },
  "order": null,
  "fallback_to_manual": false
}
```
- `state` ∈ `collecting` | `awaiting_confirmation` | `order_placed` | `fallback`. The **server** sets it (ADR-04). The model never sets it.
- `reply_audio` is returned inline and not stored. Storing audio needs **PA-34** approval.
- **Model tools:** `search_products`, `get_customer_addresses`, `get_quote` (same logic as C4), `get_order_status`. **There is no order-placing tool** (DR-10).
- **Voice confirmation:** when the session is `awaiting_confirmation`, the server first runs a separate yes/no check on the customer's turn.
  - Clear "yes" → the server places the order for `pending_quote_id` (same logic as C5, `confirmation_method = voice`) and returns `state: order_placed` with the order.
  - "No" → `collecting`.
  - Unclear → asks again or asks for a tap.
- **Rate limits and account-status check** run before any Gemini call. If exceeded → `RATE_LIMITED` (NFR-SEC-10). Business caps are **PA-35**.
- **Usage logging:** one `ai_usage_logs` row per Gemini call (conversation, tool loop, confirmation check), grouped by `turn_id` (FR-AI-07).

## 5. Delivery Partner Panel APIs (role: `delivery_partner`)

| # | Endpoint | Type | Req ID | Description |
|---|---|---|---|---|
| D1 | `set-duty-status` | Edge Fn | FR-DP-02 | `{ "status": "on_duty" }`. Creates or closes the attendance record |
| D2 | `upload-attendance-photo` | Storage (private) + Edge Fn | FR-DP-03 | Uploads photo and links it to attendance. Location capture **PA-06** |
| D3 | `my-delivery-requests` read + Realtime | Restricted view (own) | FR-DP-04 | Pending delivery requests. Visible fields **PA-56** |
| D4 | `accept-delivery-request` | Edge Fn → RPC (atomic) | FR-DP-05/06, BR-03 | `{ "order_id" }` → `200 { assigned: true }` or `409 ORDER_ALREADY_ASSIGNED` |
| D5 | `reject-delivery-request` | Edge Fn | FR-DP-05 | Records the rejection |
| D6 | `my-orders` read | Table/view | FR-DP-07/08 | Assigned orders with customer details. Calling and navigation open on the device (provider **PA-36**) |
| D7 | `update-order-status` | Edge Fn | FR-DP-09, BR-11 | `{ "order_id", "to_status": "picked_up" }` |
| D8 | `record-cod` | Edge Fn | FR-DP-10 | `{ "order_id", "collected_paise" }`. Usually sent together with `delivered` |
| D9 | `report-issue` | Edge Fn | FR-DP-11 | `{ "order_id", "type", "reason", "note", "photo_path?" }` (**PA-17**) |
| D10 | `my-performance` | View / RPC | FR-DP-12 | Attendance, history, counts. Metrics **PA-55** |
| D11 | `update-my-profile` | Edge Fn | FR-DP-01 | Shared with §2. Editable fields **PA-14** |

## 6. Store Manager Panel APIs (role: `store_manager`, scoped to the manager's store, BR-01)

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| M1 | `store-orders` read + Realtime | Table/view | FR-SM-01 |
| M2 | `store-delivery-partners` read | View | FR-SM-02, FR-SM-15 |
| M3 | `assign-order` | Edge Fn → RPC | FR-SM-02 (reassign **PA-40**) |
| M4 | `set-store-open` | Edge Fn | FR-SM-04 (precedence **PA-50**) |
| M5 | `set-surge` | Edge Fn | FR-SM-05 (limits **PA-05**) |
| M6 | `manage-category` (create / update / deactivate) | Edge Fn | FR-SM-06, BR-06 (permissions **PA-32**) |
| M7 | `manage-product` (create / update / deactivate, incl. price) + image upload | Edge Fn + Storage | FR-SM-07/08/09, DR-11 (pricing **PA-02**, permissions **PA-32**) |
| M8 | `adjust-stock` | Edge Fn → RPC | FR-SM-10, DR-11 (writes `stock_movements`) |
| M9 | `store-reports` | RPC/view | FR-SM-03, FR-SM-11 |
| M10 | `cod-reconciliation` read + `settle-cod` | View + Edge Fn | FR-SM-12 (**PA-08**) |
| M11 | `manage-coupon` / `manage-cashback-campaign` (store scope) | Edge Fn | FR-SM-13 (**PA-32**, wallet crediting **PA-52**) |
| M12 | `store-reviews` / `store-complaints` read + `respond-review` / `update-complaint` | View + Edge Fn | FR-SM-14 (**PA-25**, **PA-26**) |
| M13 | `verify-attendance` | Edge Fn | FR-SM-15 (**PA-06**) |

## 7. Super Admin Panel APIs (role: `super_admin`)

| # | Endpoint | Type | Req ID |
|---|---|---|---|
| S1 | `manage-store` (create / update / suspend) | Edge Fn | FR-SA-01/03 |
| S2 | `manage-manager` (create / approve / suspend, assign to store) | Edge Fn | FR-SA-03 |
| S3 | `manage-delivery-partner` (create / approve / suspend, assign to store) | Edge Fn | FR-SA-01 (onboarding **PA-14**) |
| S4 | `customers` list/view + `set-customer-status` | View + Edge Fn | FR-SA-01 |
| S5 | `update-store-settings` / `update-platform-settings` | Edge Fn | FR-SA-04/05/12 (settings list **PA-51**, open/close precedence **PA-50**) |
| S6 | `set-manager-permissions` | Edge Fn | FR-SA-12 (**PA-32**) |
| S7 | `manage-coupon` / `manage-cashback-campaign` (platform scope) | Edge Fn | FR-SA-06 |
| S8 | `wallet-credit` | Edge Fn → RPC | FR-SA-07, FR-WP-04/05, BR-18 |
| S9 | `finance-reports` (wallet, COD, promo spend) | RPC/views | FR-SA-08, FR-WP-07 |
| S10 | `performance-compare` (stores, managers, DPs) | RPC/views | FR-SA-02/09 (metrics **PA-55**) |
| S11 | `reviews` / `complaints` platform view | Views | FR-SA-10 |
| S12 | `ai-usage-report` | RPC/view | FR-SA-11, KPI-09 (currency **PA-57**) |
| S13 | `audit-log` read | Table | NFR-SEC-07 |
| S14 | `partner-attendance` read (any store, with photos via signed URLs) | View | FR-SA-01 |
| S15 | `set-user-role` (add or remove a staff role for a phone number; pre-registration before the person's first login) | Edge Fn | FR-SA-03, FR-APP-04 (more than one role per account **PA-60**) |

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
- Header `Idempotency-Key` is required.
- The write is serialized per wallet (NFR-SEC-12).
- The response includes the ledger entry and the new balance.
- Approval workflow is **PA-44**. Whether any caller other than the Super Admin may credit is **PA-52**. Until decided, only `super_admin` is allowed.

## 8. Realtime Channels (proposed)

| Channel | Subscribers | Events |
|---|---|---|
| `order:<id>` | Customer, assigned DP, store manager | status changes |
| `store:<id>:orders` | Store manager | new/updated orders |
| `dp:<id>:delivery-requests` | Delivery partner | new delivery request, request withdrawn |

## 9. Versioning
Edge Functions are versioned by path suffix if a breaking change is needed (e.g. `quote-order-v2`). Generated TypeScript types are kept in the app's shared folder (`app/src/shared`) (RECOMMENDATION). Because old app versions stay installed on phones, Edge Functions must keep working with the previous app version until most users have updated.
