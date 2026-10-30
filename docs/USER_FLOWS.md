# Ramu Kaka — User Flows

| Field | Value |
|---|---|
| Status | **DRAFT** |

> Flows are built from the confirmed requirements. Branches that depend on an open decision are marked **PA-xx**. Diagrams use Mermaid.

---

## UF-01 Customer Onboarding (FR-CU-01/02/03)

```mermaid
flowchart TD
  A[Open app] --> B[Choose language: Hindi / English / Bhojpuri]
  B --> C[Enter mobile number]
  C --> D[Receive OTP by SMS]
  D --> E{OTP valid?}
  E -- No --> D2[Show error, allow resend after cooldown] --> D
  E -- Yes --> F{New user?}
  F -- Yes --> G[Enter name]
  F -- No --> H
  G --> H[Add or select delivery address]
  H --> I{Address in a service area?}
  I -- No --> J[Show 'Not serviceable yet' screen. Can save address. Notify later? PA-01]
  I -- Yes --> K[Home screen]
```

## UF-02 Voice Order (FR-AI-01…08, BR-04, BR-12, BR-13)

```mermaid
sequenceDiagram
  actor C as Customer
  participant App
  participant AI as ai-turn (server)
  participant BE as Quote/Order logic
  C->>App: Taps mic: "Ramu Kaka, ek 20 litre paani ka jar bhej do"
  App->>AI: audio + session
  AI->>BE: search_products("20 litre jar"), get addresses
  alt Missing info (e.g. 2 saved addresses)
    AI-->>App: "Kaun sa pata – Ghar ya Dukaan?"
    C->>App: "Ghar"
    App->>AI: audio
  end
  AI->>BE: get_quote(items, address)
  alt Store closed / out of stock / out of area
    BE-->>AI: error
    AI-->>App: Explains the problem (no order placed)
  else OK
    BE-->>AI: quote (price, charges, total)
    AI-->>App: Reads back product, qty, address, price + shows summary card
    C->>App: "Haan" or taps Confirm
    App->>AI: confirmation
    AI->>BE: place_order(quote_id)
    BE-->>AI: order_id, status=placed
    AI-->>App: "Order ho gaya. Order number ..."
  end
```

**Fallback (FR-AI-06):** after repeated recognition failures (threshold RECOMMENDED as 2), or if the customer taps "Order manually", the app opens UF-03 with any items already understood pre-filled.

## UF-03 Manual Order (FR-CU-04/06/07/12)

1. Home → choose a category (Water) → choose a product (e.g. 20 L jar).
2. Set quantity → **Review order**.
3. The app requests a server quote (`quote-order`). The summary shows items, price, delivery charge, surge (if any), discount, total, address and payment method (COD).
4. Optional: apply a coupon, which triggers a new quote. Wallet use is **PA-27**.
5. Tap **Confirm order**, which calls `place-order(quote_id)`.
6. If the price or stock changed since the quote, show the new summary and ask the customer to confirm again.
7. Confirmation screen → Order tracking.

## UF-04 Dispatch and Acceptance (FR-DP-04/05/06, BR-03, BR-10)

```mermaid
sequenceDiagram
  participant BE as Backend
  participant DP1 as Delivery Partner 1
  participant DP2 as Delivery Partner 2
  participant SM as Store Manager
  BE->>DP1: Offer (push + in-app)
  BE->>DP2: Offer (push + in-app)
  DP2->>BE: Accept
  BE->>BE: Atomic assign: only if unassigned
  BE-->>DP2: Assigned ✅
  DP1->>BE: Accept
  BE-->>DP1: 409 Already taken
  BE->>SM: Order accepted by DP2
  Note over BE,SM: Who receives offers (all on-duty vs nearest),<br/>timeout and escalation = PA-03 / PA-40
  alt Nobody accepts within timeout (PA-03)
    BE->>SM: Alert: unassigned order
    SM->>BE: Manually assign (FR-SM-02)
  end
```

## UF-05 Delivery Execution and COD (FR-DP-07…10, BR-11)

```mermaid
stateDiagram-v2
  [*] --> PLACED
  PLACED --> ACCEPTED: DP accepts / SM assigns
  ACCEPTED --> PICKED_UP: DP taps Picked Up
  PICKED_UP --> OUT_FOR_DELIVERY: DP taps Out for Delivery
  OUT_FOR_DELIVERY --> DELIVERED: DP taps Delivered + records COD
  PLACED --> CANCELLED: PA-16
  ACCEPTED --> CANCELLED: PA-16
  ACCEPTED --> FAILED: DP reports issue (PA-17)
  PICKED_UP --> FAILED: PA-17
  OUT_FOR_DELIVERY --> FAILED: PA-17
  DELIVERED --> [*]
  CANCELLED --> [*]
  FAILED --> [*]
```
> `PLACED`, `CANCELLED` and `FAILED` are proposed statuses (**PA-39**). The PDF confirms only the four DP statuses.

At **Delivered**, the DP enters the cash collected. If it differs from the order total, the record is flagged for the manager (**PA-08**).

## UF-06 Delivery Partner Shift (FR-DP-02/03/12)

1. Login → Home.
2. Toggle **On Duty** → the app requests the work-start photo at the warehouse.
3. Photo uploaded (plus location and time, **PA-06**). Attendance is `pending verification`.
4. The DP becomes eligible for offers. Whether offers wait until the photo is verified is **PA-06**.
5. During the shift: UF-04 and UF-05.
6. Toggle **Off Duty** → attendance closed. Daily summary shows deliveries and COD collected.
7. COD hand-over to the manager (**PA-08**).

## UF-07 Failed Delivery (FR-DP-11)

1. On an active order, DP taps **Problem**.
2. Selects a reason (list **PA-17**, e.g. customer not reachable, wrong address, customer refused) and adds a note or photo.
3. The order moves to `FAILED` (or a retry state, **PA-17**). Customer and manager are notified.
4. Manager reviews it. Stock restoration and customer communication follow **PA-17**.

## UF-08 Store Manager Daily Operations (FR-SM-01…15)

1. Login → Overview.
2. **Open store** (manual toggle; schedule is **PA-33**).
3. Check DP attendance and verify start photos.
4. Update stock for water products.
5. Monitor the orders board. Handle unassigned-order alerts by assigning manually.
6. Optionally apply surge or extra delivery charges within admin limits (**PA-05**).
7. Review complaints and reviews.
8. End of day: reconcile COD per DP → mark settled (**PA-08**). **Close store.**

## UF-09 Product and Category Management (FR-SM-06…10)

1. Categories → **Add category** (name in each language, icon) → Save.
2. Products → **Add product** → category, names and descriptions, unit, price (**PA-02**), stock, images → Save.
3. **Deactivate** instead of delete. Deactivated items disappear from the customer app and the AI catalog.
4. Stock adjustment: enter a new quantity or a +/- change with a reason (audit trail).

## UF-10 Super Admin: Store & Manager Setup (FR-SA-01/03/04/12)

1. Stores → **Create store** (name, address, location, service area **PA-01**).
2. Configure store settings and limits (delivery charges **PA-19**, surge cap **PA-05**, offer timeout **PA-03**).
3. Managers → **Create manager** (phone) → assign to store → set permissions (**PA-32**) → approve.
4. The manager logs in by OTP and sees only that store.
5. Suspend a store or manager at any time. Suspension blocks access immediately.

## UF-11 Wallet Credit by Super Admin (FR-SA-07, BR-05, BR-18)

1. Customers → select a customer → **Credit wallet**.
2. Enter amount, type (cashback / refund / compensation) and a required reason or reference (e.g. a complaint).
3. Confirmation dialog shows the amount and the customer.
4. Server creates an append-only ledger entry recording the approver. Customer is notified.
5. Mistakes are fixed with a **reversal** entry, never by editing.
6. Two-step approval is **PA-44**.

## UF-12 Coupon / Cashback Campaign (FR-SA-06, FR-SM-13)

1. Offers → **New coupon** or **New cashback campaign**.
2. Scope: platform (SA) or own store (SM, if permitted).
3. Rules: value, minimum order, validity, usage limits. Policy is **PA-04**, **PA-29**, **PA-30**.
4. Activate. The offer appears to eligible customers and is validated at quote time.
5. Spend is tracked in finance reports (FR-WP-07, KPI-08).

## UF-13 Review & Complaint (FR-CU-13, FR-SM-14, FR-SA-10)

1. After delivery, the customer is prompted to rate the product(s).
2. The review appears in the manager dashboard. Moderation and replies are **PA-25**.
3. Complaints: intake channel **PA-26** → manager handles → may escalate to SA → may lead to a wallet compensation (UF-11).

## UF-14 AI Cost Monitoring (FR-SA-11, BR-08)

1. Every AI turn logs usage server-side.
2. SA → AI Usage: total spend, spend per customer, spend per completed order, voice vs manual conversion.
3. Thresholds and alerts or caps are **PA-35**.
