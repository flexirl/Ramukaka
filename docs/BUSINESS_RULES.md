# Ramu Kaka — Business Rules

| Field | Value |
|---|---|
| Source of truth | Ramu Kaka PRD v1.0 (PDF) |
| Status | **DRAFT – awaiting approval** |
| Last updated | 2026-10-30 (documentation audit corrections, see [CHANGELOG](../CHANGELOG.md)) |

Labels used in this document:
- **CONFIRMED**: stated in the PDF. The source section is cited.
- **DERIVED**: follows directly from a confirmed requirement. It adds no new business policy.
- **APPROVED DESIGN RULE**: a technical rule the user approved during Level 0. It is not a business policy.
- **PENDING APPROVAL**: needs a decision from the business. The ID is `PA-xx`.
- **RECOMMENDATION**: a proposal only. It does not apply until approved.

---

## 1. Confirmed Business Rules `[PDF §9]`

| ID | Rule | Enforcement point (proposed) |
|---|---|---|
| BR-01 | Each manager can access **only their assigned store**. | Database Row-Level Security (RLS) + server functions |
| BR-02 | Customers can order **only from supported service areas**. | Server-side order validation |
| BR-03 | An order must **not** be assigned to more than one delivery partner. | Atomic server-side assignment |
| BR-04 | Placing an order requires **customer confirmation**. | Server-side confirmation step (DR-10) |
| BR-05 | Wallet transactions must be **recorded and auditable**. | Append-only ledger |
| BR-06 | Managers can create products and categories **within their permissions**. | Server functions + permission flags |
| BR-07 | The Super Admin has **platform-wide oversight**. | Role `super_admin` |
| BR-08 | AI usage must be **monitored** to control operating costs. | Server-side AI usage log |
| BR-09 | Store closure and stock unavailability must **prevent invalid order placement**. | Server-side order validation |

## 2. Additional Confirmed Rules (from other PDF sections)

| ID | Rule | Source |
|---|---|---|
| BR-10 | An order goes to the **first eligible delivery partner who accepts it**, subject to server-side checks. | §4 Panel 2 |
| BR-11 | Delivery status sequence: **Accepted → Picked Up → Out for Delivery → Delivered** (system values `accepted`, `picked_up`, `out_for_delivery`, `delivered`). | §4 Panel 2 |
| BR-12 | Before an order is placed, the customer confirms the **product, quantity, address and price**. | §4 Panel 1, §5 |
| BR-13 | The AI assistant must use **real backend prices, stock and delivery rules**. It must never invent or estimate them. | §5 |
| BR-14 | When voice recognition fails, the customer can fall back to manual ordering. | §5 |
| BR-15 | COD is the **only payment method at launch**. | §6 |
| BR-16 | The wallet holds only **cashback, promotional credits and approved refunds**. | §4 Panel 1, §6 |
| BR-17 | Every wallet credit and debit has a secure transaction record. | §6 |
| BR-18 | The Super Admin **can** issue approved wallet credits (cashback, refunds, compensation). Whether any other actor or automatic process (for example, a manager's cashback campaign) may also credit wallets is **PA-52**. | §4 Panel 4, §6 |
| BR-19 | Managers apply surge pricing or extra delivery charges **only if authorized**. | §4 Panel 3 |
| BR-20 | Managers create coupons, cashback and offers only **within assigned permissions**. | §4 Panel 3 |
| BR-21 | Wallet credits, order assignment, pricing and financial calculations are **validated server-side**. | §7 |
| BR-22 | "API keys must not be exposed in the mobile application." Secret keys (Gemini, Supabase service-role, SMS provider) are **never** shipped. Whether platform-restricted public client keys (for example, Google Maps Android key, FCM config) are allowed is **PA-53**. Until it is approved, no API key other than the Supabase anon key ships in a client. | §7 |
| BR-23 | All panels share **one backend and database** with role-based access control. | §7 |
| BR-24 | The delivery partner's work-start photo is taken **at the warehouse** and is **subject to verification**. | §4 Panel 2 |
| BR-25 | The Super Admin sets **store-level operating limits** and user permissions. | §4 Panel 4 |
| BR-26 | Authentication uses **Supabase Auth with OTP verification**. The PDF gives no per-panel exception, so this applies to all four roles. | §7 |

## 3. Derived Rules and Approved Design Rules

| ID | Rule | Type | Derived from |
|---|---|---|---|
| DR-01 | A delivery partner who is Off Duty receives no delivery requests and cannot accept orders. | DERIVED | FR-DP-02, BR-10 ("eligible") |
| DR-02 | When a delivery partner accepts an order that is already assigned, the acceptance is rejected and they are told the order is taken. | DERIVED | BR-03, BR-10 |
| DR-03 | Status changes only move forward through the BR-11 sequence. Failure and issue reports use a separate path (FR-DP-11). | DERIVED | BR-11 |
| DR-04 | Order prices are recalculated on the server when the order is placed. The price shown to the customer is checked against it. | DERIVED | BR-12, BR-21 |
| DR-05 | A wallet balance is computed from ledger entries. It is never edited directly. | DERIVED | BR-05, BR-17 |
| DR-06 | Every AI request goes through a server-side function, which logs usage (tokens/audio, estimated cost, customer) for **each Gemini API call**. | DERIVED | BR-08, BR-22, FR-AI-07, FR-SA-11 |
| DR-07 | A closed store and an out-of-stock product are both shown to the customer and to the AI before confirmation. | DERIVED | BR-09, BR-13 |
| DR-08 | A suspended account loses access **immediately**, including any login session that has not expired yet. | DERIVED | FR-SA-03 ("suspend") |
| DR-09 | The AI assistant can never set or change a price, discount, charge or wallet amount. It can only pass customer inputs (product, quantity, address, coupon code) to server functions, which compute every amount. | DERIVED | BR-13, BR-21 |
| DR-10 | **The AI model cannot place an order.** An order is placed only by the server, for a quote the customer was shown, after one of two server-recorded confirmation events: (a) the customer taps **Confirm** for that quote, or (b) a separate server-side yes/no check classifies the customer's next voice turn as a clear "yes" for that pending quote. Anything unclear → ask again, or ask for a tap. | APPROVED DESIGN RULE (audit, 2026-10-30) | BR-04, BR-12, BR-13, BR-21 |
| DR-11 | Every change to a product price, stock quantity or availability goes through a server function and is audit-logged. | APPROVED DESIGN RULE (audit, 2026-10-30) | BR-06, BR-21 |

## 4. Pending Approval Register

### 4.1 Listed as undecided in the PDF `[PDF §11]`

| ID | Decision needed | Why it blocks development | Recommendation (not confirmed) |
|---|---|---|---|
| **PA-01** | Initial launch locations and service radius | Needed for BR-02 service-area validation | Start with Jamui. Define each store's service area as a radius (km) from the store. A polygon or pincode list can be added later. |
| **PA-02** | Supplier inventory and water pricing model | Needed for product pricing and stock setup | One price per product per store, set by the manager within admin limits. (Supplier scope: see PA-59.) |
| **PA-03** | Delivery assignment rules and acceptance timeout | Needed for the delivery request flow | Send a delivery request to all On-Duty partners of the store. First accept wins. If nobody accepts within *N* minutes, alert the manager to assign manually. |
| **PA-04** | Cashback, wallet refund and coupon policies | Needed for wallet and coupon logic | See questions PA-27 to PA-30, PA-52 |
| **PA-05** | Store-level surge pricing permissions | Needed for FR-SM-05 and the quote engine | Admin sets the maximum surge (% or ₹) for each store. The manager can apply up to that limit. |
| **PA-06** | Attendance photo verification requirements | Needed for FR-DP-03 | Phase 1: photo + GPS + timestamp, reviewed by the manager. Automatic face verification in Phase 2. |
| **PA-07** | Hindi, English and Bhojpuri voice support expectations | Needed for AI acceptance criteria | Treat Bhojpuri as "best effort" and measure it in a pilot (see TESTING_STRATEGY) |
| **PA-08** | COD settlement and financial reporting rules | Needed for COD reconciliation | Delivery partner hands cash to the manager at the end of each shift. Manager marks it settled. Admin views outstanding amounts. |
| **PA-09** | Final MVP scope and launch timeline | Needed for the roadmap | See DEVELOPMENT_ROADMAP |

### 4.2 Gaps found during documentation review

**Platform and product structure**

| ID | Question |
|---|---|
| **PA-10** | Do the Store Manager and Super Admin dashboards run as a **web app**, inside the **React Native app**, or both? The PDF lists only React Native. *Recommendation:* web dashboards built with React + TypeScript, sharing types with the mobile app. **Blocks Level 2** (repository layout). |
| **PA-11** | Should there be **one app with role-based screens**, or **separate apps** (Customer app and Delivery Partner app)? *Recommendation:* two Android apps, built from one codebase or monorepo. |
| **PA-12** | Is **iOS** in scope? The PDF mentions only Android Studio. *Assumption until decided:* Android only. |
| **PA-38** | App name and branding, the Play Store developer account owner, and who creates the Google Stitch designs. |
| **PA-42** | Supabase hosting region and plan. *Recommendation:* the Mumbai region (`ap-south-1`) for latency and data residency. |

**Stores, service areas and catalog**

| ID | Question |
|---|---|
| **PA-13** | How is a customer's order routed to a store: nearest store, service-area match, or customer's choice? What happens when two service areas overlap? |
| **PA-31** | Is the **product catalog** owned by each store, or is there a global catalog with store-level price and stock? (The PDF lets both the manager and the admin manage products. This also decides how FR-SA-05 platform-wide products are stored.) |
| **PA-32** | Exactly which **manager permissions** can the admin turn on or off? Examples: create categories, set prices, create coupons, apply surge, assign orders. |
| **PA-33** | Store hours: manual open/close toggle only, a schedule, or both? |
| **PA-41** | Can a store have **more than one manager**? Can a manager run more than one store? (The PDF says "each store will have its own manager".) |
| **PA-18** | Are there **empty jar return or jar deposit** rules? The PDF does not mention them. Please confirm whether they apply. |

**Orders and delivery**

| ID | Question |
|---|---|
| **PA-16** | **Cancellation rules:** who can cancel (customer, manager, admin), until which status, and what are the consequences? |
| **PA-17** | **Failed delivery handling:** reason list, retry policy, what happens to stock, and who closes the order. |
| **PA-19** | **Delivery charges and minimum order value:** the base delivery fee, the minimum order amount, and how they combine with surge. |
| **PA-20** | Immediate delivery only, or also **scheduled or recurring** orders? (Not in the PDF. *Assumption:* immediate only.) |
| **PA-39** | Full order status list. The PDF lists only the delivery partner statuses. *Proposed:* `placed → accepted → picked_up → out_for_delivery → delivered`, plus `cancelled` and `failed`. |
| **PA-40** | If no delivery partner accepts, does the manager assign manually (FR-SM-02)? Can a manager **reassign** an order that was already accepted? |
| **PA-46** | Can a customer **edit** an order after placing it? |
| **PA-14** | **Delivery partner onboarding:** who creates the accounts (manager or admin), what KYC documents are required, is each partner tied to one store, and which profile fields can a partner edit themselves (FR-DP-01 "profile management")? |
| **PA-15** | **Extra delivery partner login checks.** OTP is CONFIRMED for all roles (BR-26). Are any extra checks needed for delivery partners beyond OTP? |
| **PA-24** | **Customer ↔ delivery partner contact:** how does the customer contact the partner (call, chat, or both; FR-CU-14 says only "contact"), and should calls use the real numbers or masked calling? |
| **PA-36** | Maps and navigation provider (for example, opening Google Maps by deep link) and how addresses are geocoded or pinned. |

**Wallet, coupons and finance**

| ID | Question |
|---|---|
| **PA-27** | Can a **wallet balance be used to pay** for orders? Fully or partly? Is there a maximum per order? Does the balance expire? Can it be withdrawn? |
| **PA-28** | When is a **refund** given in a COD-only model, and who approves it? |
| **PA-29** | Can coupons be **combined** with each other or with wallet usage or cashback? |
| **PA-30** | **Cashback timing:** credited on delivery, or after a delay? Is there a cap per order or per customer? |
| **PA-43** | **GST and invoices:** is a tax invoice or receipt needed for each order? |
| **PA-44** | Is there **one Super Admin** or several admin users? Are there sub-roles (for example, a finance-only admin)? Is two-person approval needed for wallet credits? |

**Reviews, complaints and notifications**

| ID | Question |
|---|---|
| **PA-25** | Ratings: **products only** (as the PDF says), or also delivery partner and order? Can managers hide or reply to reviews? |
| **PA-26** | **Complaints and feedback:** how does a customer raise a complaint? (The PDF has managers and admin handling complaints but lists no complaint screen in the Customer App.) Is "feedback" (FR-SM-14) separate from reviews and complaints? |
| **PA-21** | **Notification channels:** push only, or also SMS or WhatsApp? |
| **PA-22** | **OTP SMS provider** (for example, MSG91 or Twilio) and the Indian DLT template registration owner. |

**AI, data and compliance**

| ID | Question |
|---|---|
| **PA-23** | **Voice pipeline:** Gemini end-to-end (audio in, audio out), or separate speech-to-text and text-to-speech services? Confirm Gemini model and cost budget. |
| **PA-35** | **AI cost controls:** monitoring only (as stated in the PDF), or also per-customer daily limits or caps? (Technical rate limiting against abuse is NFR-SEC-10 and is separate from business caps.) |
| **PA-34** | Data retention and privacy: compliance with India's DPDP Act 2023, how long to keep attendance photos and voice recordings, and whether raw audio (customer input or AI reply) is stored at all. |
| **PA-37** | Should **Prashant's existing customer data** be imported? In what format? Is consent needed? |
| **PA-45** | Customer **account deletion** process (required by Play Store policy). |
| **PA-47** | Are the Delivery Partner app and the dashboards translated, or English/Hindi only? |
| **PA-48** | Numeric launch targets for the success metrics (KPIs). |

### 4.3 Added by the documentation audit (2026-10-30)

| ID | Decision needed | Affects |
|---|---|---|
| **PA-49** | **Stock reservation timing:** is stock reduced when the order is placed, when it is accepted, or when it is delivered? Is it released on cancellation or failed delivery? | FR-SM-10, BR-09, place-order, UF-05/07 |
| **PA-50** | **Open/close precedence:** if the manager's open/close toggle (FR-SM-04) conflicts with the admin's store opening settings (FR-SA-04), which one wins? | FR-SM-04, FR-SA-04 |
| **PA-51** | **Platform-wide settings list:** which product, pricing and operational settings does the Super Admin control (FR-SA-05)? | FR-SA-05, `platform_settings` |
| **PA-52** | **Who can credit wallets:** only the Super Admin manually, or also automatic cashback from platform or store campaigns, or managers? | BR-18, FR-SM-13, FR-SA-07, wallet API |
| **PA-53** | **Public client keys:** may platform-restricted, non-secret keys (Google Maps Android key, FCM configuration) ship in the app, given the PDF says "API keys must not be exposed"? | BR-22, push notifications, maps |
| **PA-54** | **Store vs warehouse:** is every store also its own warehouse (the PDF uses "Store / Warehouse Manager" and "photo from the warehouse"), or can they be separate places? | Schema `stores`, attendance |
| **PA-55** | **KPI and performance definitions:** definitions of "active customer", "fulfilment time", "productivity", delivery partner performance (FR-DP-12, FR-SM-15) and **manager** performance (FR-SA-09). | SRS §3.8, analytics, Level 10 |
| **PA-56** | **Delivery request contents:** what a delivery partner sees *before* accepting (for example, area or distance, items, order value), given that customer contact details are shown only after assignment. | FR-DP-04/07, DP-S04 |
| **PA-57** | **AI cost reporting currency:** Gemini bills in USD. Should dashboards report in ₹, and if so, which exchange rate (fixed or periodic)? | FR-SA-11, KPI-09, `ai_usage_logs` |
| **PA-58** | **Manager wallet visibility:** can a store manager see customer wallet balances or transactions (for example, cashback from their store campaigns)? | Access matrix, FR-SM-13 |
| **PA-59** | **Supplier scope:** is supplier inventory (PDF §11, "Supplier inventory and water pricing model") part of Phase 1, and does it need its own records? | Schema, PA-02 |

## 5. Rule Change Process

1. A business owner approves the change.
2. This file is updated: the PA item becomes a `BR-xx` rule with the approval date and approver.
3. Dependent documents are updated in the same commit (SRS, schema, API, tests).
4. The change is recorded in [CHANGELOG.md](../CHANGELOG.md).
