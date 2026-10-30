# Ramu Kaka — Software Requirements Specification (SRS)

| Field | Value |
|---|---|
| Based on | Ramu Kaka PRD v1.0 (PDF), [PRD.md](./PRD.md) |
| Standard | Loosely follows the IEEE 830 structure |
| Status | **DRAFT – awaiting approval** |

Labels: **CONFIRMED** (from the PDF), **PENDING APPROVAL** (`PA-xx`), **RECOMMENDATION** (not binding).

---

## 1. Introduction

### 1.1 Purpose
This document turns the business PRD into requirements that can be tested. It covers all four panels, the AI voice assistant, wallets and payments, and the shared backend.

### 1.2 Scope
Phase 1 is the Water Delivery MVP `[PDF §8]`. Phase 2 and Phase 3 requirements are noted so the design can support them later. They are not built in Phase 1.

### 1.3 Definitions

| Term | Meaning |
|---|---|
| COD | Cash on Delivery |
| DP | Delivery Partner |
| SM | Store / Warehouse Manager |
| SA | Super Admin / Owner |
| RLS | PostgreSQL Row-Level Security (used by Supabase) |
| Edge Function | A Supabase server-side function (Deno/TypeScript) |
| Service area | The geographic area a store delivers to (definition **PA-01**, **PA-13**) |
| Offer (dispatch) | A notification inviting eligible DPs to accept an order |
| Ledger | An append-only list of wallet transactions |

### 1.4 References
- Ramu Kaka PRD v1.0 PDF (source of truth)
- [BUSINESS_RULES.md](./BUSINESS_RULES.md), [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md), [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md), [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)

## 2. Overall Description

### 2.1 Product Perspective
One shared backend (Supabase: Postgres, Auth, Storage, Edge Functions, Realtime) serves four client panels `[PDF §7]`. The AI assistant (Gemini API) is reachable **only** through server-side functions `[PDF §7]`.

### 2.2 User Classes

| Class | Technical comfort | Main device | Notes |
|---|---|---|---|
| Customer | Low to medium (OBJ-02) | Android phone, often low-end (RECOMMENDATION: design for it) | Voice-first; Hindi, English or Bhojpuri |
| Delivery Partner | Low to medium | Android phone | Works outdoors, needs large controls |
| Store Manager | Medium | Platform **PA-10** | Access limited to one store |
| Super Admin | Medium to high | Platform **PA-10** | Platform-wide access |

### 2.3 Operating Environment
- Mobile: React Native + TypeScript, Android build through Android Studio `[PDF §7]`. iOS is **PA-12**.
- Backend: Supabase managed cloud `[PDF §7]`. Region is **PA-42**.
- Network: Bihar districts may have patchy 3G/4G. **RECOMMENDATION:** design for slow and intermittent connectivity (see NFR-PERF, NFR-REL).

### 2.4 Constraints (CONFIRMED)
- C-01: The technology stack is fixed by `[PDF §7]`.
- C-02: No API keys in the mobile app `[PDF §7]`.
- C-03: Sensitive operations (wallet, assignment, pricing, finance) run server-side `[PDF §7]`.
- C-04: COD is the only payment method in Phase 1 `[PDF §6]`.

### 2.5 Assumptions (to be confirmed)
- A-01: Android only for Phase 1 (**PA-12**).
- A-02: Immediate delivery only; no scheduled orders (**PA-20**).
- A-03: Delivery partners belong to a single store (**PA-14**).
- A-04: Prices are in INR, including any taxes (**PA-43**).

## 3. Functional Requirements

Requirement IDs match [PRD.md](./PRD.md). Each requirement below adds **acceptance criteria (AC)**. An AC that depends on an open decision cites the relevant `PA-xx` item.

### 3.1 Authentication and Accounts

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-CU-01 | Customer OTP registration and login | AC1: User enters an Indian mobile number and receives an OTP by SMS (provider **PA-22**). AC2: A valid OTP creates or logs into the account. AC3: An invalid or expired OTP is rejected with a clear message in the selected language. AC4: OTP rate limiting applies (limits are a RECOMMENDATION, see NFR-SEC-05). |
| FR-DP-01 | DP secure login and profile | AC1: Login method is **PA-15**. AC2: Only accounts created or approved by SM/SA can log in (**PA-14**). AC3: DP can view and edit their allowed profile fields (field list **PENDING**). |
| FR-SA-03 | SA creates, approves and suspends store and manager accounts | AC1: A suspended manager cannot log in or call manager APIs. AC2: Every create, approve and suspend action is written to the audit log. |
| — | Role resolution | AC1: Each authenticated user has one or more roles from {customer, delivery_partner, store_manager, super_admin}. AC2: Each panel rejects users who lack its role. |

### 3.2 Customer App

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-CU-02 | Address management | AC1: Customer can add, edit and save multiple addresses. AC2: Each address records a location used for the service-area check (method **PA-36**). AC3: An address outside every service area is flagged as "not serviceable" (BR-02). |
| FR-CU-03 | Language selection | AC1: Choice is offered at first launch and in Settings. AC2: The choice is saved to the profile and applied to the UI and the AI assistant. |
| FR-CU-07 / 08 | Browse catalog | AC1: Shows categories, products, prices and availability for the store serving the selected address. AC2: Phase 1 shows water bottles and jars. AC3: Unavailable items are visibly disabled. |
| FR-CU-04 / 06 | Manual ordering | AC1: Customer picks product(s), quantity and address. AC2: A server-computed summary (items, price, charges, discounts, total) is shown. AC3: The order is created only after an explicit **Confirm** tap (BR-04). AC4: Placement fails cleanly if the store is closed or stock is short (BR-09). |
| FR-CU-09 | Order status and history | AC1: Status updates appear in near real time. AC2: History lists past orders with their details. |
| FR-CU-10 | Notifications | AC1: Customer is notified on every order status change. Channel is **PA-21**. |
| FR-CU-11 | Wallet | AC1: Shows balance and transaction history. AC2: Whether the balance can be used at checkout is **PA-27**. |
| FR-CU-12 | Coupons and offers | AC1: Shows offers the customer is eligible for. AC2: Coupon is validated server-side. Stacking rules are **PA-29**. |
| FR-CU-13 | Ratings and reviews | AC1: Customer can rate and review products. Eligibility (for example, only delivered items) and scope are **PA-25**. |
| FR-CU-14 | Contact delivery partner | AC1: Customer can call the assigned DP during an active delivery. Number masking is **PA-24**. |

### 3.3 AI Voice Assistant

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-AI-01 | Voice in, voice out | AC1: Customer holds or taps a mic button and speaks. AC2: The assistant answers with speech **and** on-screen text (RECOMMENDATION: text helps understanding). |
| FR-AI-02 | Languages | AC1: Responds in the selected language. AC2: Handles code-mixed input where feasible. Quality targets are **PA-07**. |
| FR-AI-03 | Context awareness | AC1: Can use the customer's saved addresses and the store's product list. AC2: Remembers context across turns within a session. |
| FR-AI-04 | Clarification | AC1: If product, quantity or address is missing or unclear, the assistant asks instead of guessing. |
| FR-AI-05 | Explicit confirmation | AC1: The assistant reads back product, quantity, address and price. AC2: The order is placed only after an affirmative confirmation. AC3: The confirmation step is enforced **by the backend** (a two-step quote → confirm process), not only by the prompt. |
| FR-AI-06 | Manual fallback | AC1: When speech is not recognized (threshold RECOMMENDATION: after 2 failed attempts) or the customer asks, the app opens manual ordering with any items already understood pre-filled. |
| FR-AI-07 | Usage logging | AC1: Every AI call logs customer, session, model, input/output units, estimated cost and timestamp, all server-side. |
| FR-AI-08 | Ground truth from backend | AC1: Prices, stock, store open/closed state and delivery rules come from backend tool/function calls. AC2: The quote the assistant reads out equals the server quote. |

### 3.4 Delivery Partner App

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-DP-02 | Attendance and duty | AC1: DP can switch On Duty / Off Duty. AC2: Each switch is recorded with a timestamp (location capture is **PA-06**). |
| FR-DP-03 | Work-start photo | AC1: Camera capture (gallery upload is **PA-06**). AC2: Photo is stored privately and linked to the attendance record. AC3: Verification status: pending, verified or rejected. The verifier and rules are **PA-06**. |
| FR-DP-04 | Order notifications | AC1: Eligible On-Duty DPs get a push notification for new orders. Eligibility and timeout are **PA-03**. |
| FR-DP-05 / 06 | Accept or reject | AC1: Accept is an atomic server operation. AC2: The first valid accept wins. Other DPs see "already taken" (BR-03, BR-10). AC3: A reject is recorded and the order is no longer offered to that DP. |
| FR-DP-07 / 08 | Order details, call, navigate | AC1: The assigned DP sees name, phone, address and items. AC2: Tap to call; tap to navigate (provider **PA-36**). AC3: Unassigned DPs cannot see customer phone or address (RECOMMENDATION, privacy). |
| FR-DP-09 | Status updates | AC1: Only forward moves through Accepted → Picked Up → Out for Delivery → Delivered (DR-03). AC2: Each change is timestamped and pushed to the customer and manager. |
| FR-DP-10 | COD collection | AC1: On Delivered, the DP records the amount collected. AC2: A mismatch with the order total is flagged (handling **PA-08**). |
| FR-DP-11 | Failed delivery and issues | AC1: DP picks a reason and adds an optional note or photo. AC2: Order moves to failed or issue state (handling **PA-17**). |
| FR-DP-12 | History and performance | AC1: Shows daily attendance, delivery list, counts and COD totals. Performance metric definitions are **PENDING** (RECOMMENDATION: deliveries completed, average delivery time, failure rate). |

### 3.5 Store Manager Dashboard

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-SM-01 | Order overview | AC1: Shows counts and lists of total, pending, active and completed orders for **their store only** (BR-01). |
| FR-SM-02 | Monitor and assign | AC1: Sees DP duty status and current load. AC2: Can manually assign an unassigned order, still under BR-03. Reassignment is **PA-40**. |
| FR-SM-03 | Performance tracking | AC1: Shows cancellation and failure counts and rates. |
| FR-SM-04 | Open/close store | AC1: A toggle immediately blocks or allows new orders (BR-09). AC2: SA settings may override this (FR-SA-04). Precedence is **PENDING**. |
| FR-SM-05 | Surge / extra charges | AC1: Allowed only if SA has authorized it, and only up to the limit (**PA-05**). AC2: Shown to the customer before confirmation. |
| FR-SM-06–10 | Catalog and stock | AC1: Create, edit and deactivate categories and products (with images, descriptions, prices, stock). AC2: Deactivated items are hidden from customers and the AI. AC3: Catalog ownership is **PA-31**. AC4: Stock decrement timing is **PENDING** (RECOMMENDATION: reserve at placement, release on cancel or failure). |
| FR-SM-11 | Revenue and order reports | AC1: Date-range filter. AC2: Totals match the delivered-order data. |
| FR-SM-12 | COD reconciliation | AC1: Shows COD collected per DP versus settled. AC2: Manager marks amounts received. Rules are **PA-08**. |
| FR-SM-13 | Store coupons and offers | AC1: Only if permitted (BR-20, **PA-32**). AC2: Scoped to their store. |
| FR-SM-14 | Reviews and complaints | AC1: Can view and respond to their store's reviews and complaints. Actions allowed are **PA-25** and **PA-26**. |
| FR-SM-15 | DP attendance and performance | AC1: Attendance log with photo verification queue (**PA-06**). |

### 3.6 Super Admin Dashboard

| ID | Requirement | Acceptance criteria |
|---|---|---|
| FR-SA-01 | Manage all entities | AC1: Full listing, search and status control for stores, managers, customers and DPs. |
| FR-SA-02 / 09 | Platform analytics | AC1: Totals and per-store breakdowns. AC2: Compare stores, managers and DPs. |
| FR-SA-04 / 12 | Store settings and limits | AC1: Set store operating settings, permissions and limits (surge caps, coupon permissions). |
| FR-SA-05 | Platform settings | AC1: Product, pricing and operational settings that apply platform-wide. The exact list is **PENDING**. |
| FR-SA-06 | Platform offers | AC1: Create platform-wide coupons and cashback campaigns. |
| FR-SA-07 | Wallet credits | AC1: Credit a customer wallet with a reason (cashback, refund or compensation). AC2: The ledger entry records the approving admin. AC3: Approval workflow (single or two-person) is **PA-44**. |
| FR-SA-08 | Finance monitoring | AC1: Wallet ledger, COD collected versus settled, promotional spend. |
| FR-SA-10 | Reviews and complaints | AC1: Platform-wide view. |
| FR-SA-11 | AI cost tracking | AC1: Total AI spend, spend per customer, spend per completed order (KPI-09), by date range. |

## 4. Non-Functional Requirements

> Requirements marked **CONFIRMED** come from the PDF. All numeric targets are **RECOMMENDATIONS** to be approved under **PA-09**.

### 4.1 Security (NFR-SEC)

| ID | Requirement | Type |
|---|---|---|
| NFR-SEC-01 | Role-based access on every table and API. Managers are limited to their own store. | CONFIRMED |
| NFR-SEC-02 | Gemini and other secret keys exist only in server-side environments | CONFIRMED |
| NFR-SEC-03 | Wallet, assignment, pricing and finance logic runs server-side | CONFIRMED |
| NFR-SEC-04 | Wallet ledger is append-only and auditable | CONFIRMED |
| NFR-SEC-05 | OTP request rate limiting (for example, 5 per hour per number) | RECOMMENDATION |
| NFR-SEC-06 | Attendance photos and any voice audio stored in private buckets with signed URLs | RECOMMENDATION |
| NFR-SEC-07 | Audit log for admin and manager actions on money, prices and accounts | RECOMMENDATION |

### 4.2 Performance (NFR-PERF), RECOMMENDATIONS

| ID | Requirement |
|---|---|
| NFR-PERF-01 | App cold start under 4 s on a low-end Android device (2 GB RAM) |
| NFR-PERF-02 | Catalog loads in under 2 s on 4G |
| NFR-PERF-03 | Voice turn (end of speech → start of spoken reply) under 4 s at p75 |
| NFR-PERF-04 | Order-accept round trip under 1.5 s at p95 |
| NFR-PERF-05 | Android APK/AAB download size under 40 MB |

### 4.3 Reliability (NFR-REL), RECOMMENDATIONS
- NFR-REL-01: Order placement is idempotent. A retry after a network drop does not create a duplicate order.
- NFR-REL-02: The app handles going offline gracefully: queued status updates for DPs, a clear offline banner.
- NFR-REL-03: Daily database backups (Supabase plan dependent, **PA-42**).

### 4.4 Usability (NFR-UX)
- NFR-UX-01 (CONFIRMED): simple wording, large buttons, clear icons, minimal navigation `[PDF §3]`.
- NFR-UX-02 (CONFIRMED): usable by customers with limited digital literacy `[PDF §2]`.
- NFR-UX-03 (RECOMMENDATION): minimum touch target 48 dp, minimum body text 16 sp, icons always paired with labels.

### 4.5 Localization (NFR-L10N)
- NFR-L10N-01 (CONFIRMED): Hindi, English and Bhojpuri UI and voice.
- NFR-L10N-02 (RECOMMENDATION): all UI strings in translation files. No hard-coded text.

### 4.6 Scalability (NFR-SCALE)
- NFR-SCALE-01 (CONFIRMED, OBJ-06): the data model supports multiple categories, stores and districts without schema redesign.

### 4.7 Observability and Cost (NFR-OBS)
- NFR-OBS-01 (CONFIRMED): AI usage and cost are logged per call.
- NFR-OBS-02 (RECOMMENDATION): crash reporting and error logging for the mobile apps and Edge Functions.

### 4.8 Compliance (NFR-COMP), PENDING
- Data privacy (DPDP Act 2023), retention periods, and account deletion: **PA-34**, **PA-45**.
- SMS DLT registration: **PA-22**.
- GST invoicing: **PA-43**.

## 5. External Interfaces and Dependencies

| Dependency | Purpose | Status |
|---|---|---|
| Supabase (Auth, Postgres, Storage, Edge Functions, Realtime) | Backend | CONFIRMED |
| SMS gateway for OTP (configured in Supabase Auth) | OTP delivery | Provider **PA-22** |
| Gemini API | AI assistant | CONFIRMED. Model and voice pipeline **PA-23** |
| Push notifications (for example, Firebase Cloud Messaging) | Order notifications | RECOMMENDATION. Channel **PA-21** |
| Maps / navigation (for example, Google Maps deep link, geocoding) | Navigation, addresses | **PA-36** |
| Google Play Console | Distribution | Owner **PA-38** |
| Razorpay or other gateway | Online payments | Phase 2, "if required" |
| GitHub | Version control | CONFIRMED |
| Google Stitch | UI design | CONFIRMED |

## 6. Traceability

Every requirement ID in this SRS maps to:
- the PDF section (via [PRD.md](./PRD.md))
- API endpoints ([API_DOCUMENTATION.md](./API_DOCUMENTATION.md))
- data entities ([DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md))
- test cases ([TESTING_STRATEGY.md](./TESTING_STRATEGY.md))
- roadmap level ([DEVELOPMENT_ROADMAP.md](./DEVELOPMENT_ROADMAP.md))
