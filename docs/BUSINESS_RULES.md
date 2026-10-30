# Ramu Kaka — Business Rules

| Field | Value |
|---|---|
| Source of truth | Ramu Kaka PRD v1.0 (PDF) |
| Status | **DRAFT – awaiting approval** |

Labels used in this document:
- **CONFIRMED**: stated in the PDF. The source section is cited.
- **DERIVED**: follows directly from a confirmed requirement. It adds no new business policy.
- **PENDING APPROVAL**: needs a decision from the business. The ID is `PA-xx`.
- **RECOMMENDATION**: a proposal only. It does not apply until approved.

---

## 1. Confirmed Business Rules `[PDF §9]`

| ID | Rule | Enforcement point (proposed) |
|---|---|---|
| BR-01 | Each manager can access **only their assigned store**. | Database Row-Level Security (RLS) |
| BR-02 | Customers can order **only from supported service areas**. | Server-side order validation |
| BR-03 | An order must **not** be assigned to more than one delivery partner. | Atomic server-side assignment |
| BR-04 | Placing an order requires **customer confirmation**. | Order API + AI flow |
| BR-05 | Wallet transactions must be **recorded and auditable**. | Append-only ledger |
| BR-06 | Managers can create products and categories **within their permissions**. | RLS + permission flags |
| BR-07 | The Super Admin has **platform-wide oversight**. | Role `super_admin` |
| BR-08 | AI usage must be **monitored** to control operating costs. | Server-side AI usage log |
| BR-09 | Store closure and stock unavailability must **prevent invalid order placement**. | Server-side order validation |

## 2. Additional Confirmed Rules (from other PDF sections)

| ID | Rule | Source |
|---|---|---|
| BR-10 | An order goes to the **first eligible delivery partner who accepts it**, subject to server-side checks. | §4 Panel 2 |
| BR-11 | Delivery status sequence: **Accepted → Picked Up → Out for Delivery → Delivered**. | §4 Panel 2 |
| BR-12 | Before an order is placed, the customer confirms the **product, quantity, address and price**. | §4 Panel 1, §5 |
| BR-13 | The AI assistant must use **real backend prices, stock and delivery rules**. It must never invent or estimate them. | §5 |
| BR-14 | When voice recognition fails, the customer can fall back to manual ordering. | §5 |
| BR-15 | COD is the **only payment method at launch**. | §6 |
| BR-16 | The wallet holds only **cashback, promotional credits and approved refunds**. | §4 Panel 1, §6 |
| BR-17 | Every wallet credit and debit has a secure transaction record. | §6 |
| BR-18 | Only the Super Admin can issue **approved** wallet credits (cashback, refunds, compensation). | §4 Panel 4, §6 |
| BR-19 | Managers apply surge pricing or extra delivery charges **only if authorized**. | §4 Panel 3 |
| BR-20 | Managers create coupons, cashback and offers only **within assigned permissions**. | §4 Panel 3 |
| BR-21 | Wallet credits, order assignment, pricing and financial calculations are **validated server-side**. | §7 |
| BR-22 | API keys (for example, Gemini) are **never** shipped inside the mobile app. | §7 |
| BR-23 | All panels share **one backend and database** with role-based access control. | §7 |
| BR-24 | The delivery partner's work-start photo is taken **at the warehouse** and is **subject to verification**. | §4 Panel 2 |
| BR-25 | The Super Admin sets **store-level operating limits** and user permissions. | §4 Panel 4 |

## 3. Derived Rules (logical consequences, no new policy)

| ID | Rule | Derived from |
|---|---|---|
| DR-01 | A delivery partner who is Off Duty receives no order offers and cannot accept orders. | FR-DP-02, BR-10 ("eligible") |
| DR-02 | When a delivery partner accepts an order that is already assigned, the acceptance is rejected and they are told the order is taken. | BR-03, BR-10 |
| DR-03 | Status changes only move forward through the BR-11 sequence. Failure and issue reports use a separate path (FR-DP-11). | BR-11 |
| DR-04 | Order prices are recalculated on the server when the order is placed. The price shown to the customer is checked against it. | BR-12, BR-21 |
| DR-05 | A wallet balance is computed from ledger entries. It is never edited directly. | BR-05, BR-17 |
| DR-06 | Every AI request goes through a server-side function, which logs usage (tokens/audio, estimated cost, customer). | BR-08, BR-22, FR-SA-11 |
| DR-07 | A closed store and an out-of-stock product are both shown to the customer and to the AI before confirmation. | BR-09, BR-13 |

## 4. Pending Approval Register

### 4.1 Listed as undecided in the PDF `[PDF §11]`

| ID | Decision needed | Why it blocks development | Recommendation (not confirmed) |
|---|---|---|---|
| **PA-01** | Initial launch locations and service radius | Needed for BR-02 service-area validation | Start with Jamui. Define each store's service area as a radius (km) from the store. A polygon or pincode list can be added later. |
| **PA-02** | Supplier inventory and water pricing model | Needed for product pricing and stock setup | One price per product per store, set by the manager within admin limits |
| **PA-03** | Delivery assignment rules and acceptance timeout | Needed for the order offer flow | Offer the order to all On-Duty partners of the store. First accept wins. If nobody accepts within *N* minutes, alert the manager to assign manually. |
| **PA-04** | Cashback, wallet refund and coupon policies | Needed for wallet and coupon logic | See questions PA-27 to PA-30 |
| **PA-05** | Store-level surge pricing permissions | Needed for FR-SM-05 | Admin sets the maximum surge (% or ₹) for each store. The manager can apply up to that limit. |
| **PA-06** | Attendance photo verification requirements | Needed for FR-DP-03 | Phase 1: photo + GPS + timestamp, reviewed by the manager. Automatic face verification in Phase 2. |
| **PA-07** | Hindi, English and Bhojpuri voice support expectations | Needed for AI acceptance criteria | Treat Bhojpuri as "best effort" and measure it in a pilot (see TESTING_STRATEGY) |
| **PA-08** | COD settlement and financial reporting rules | Needed for COD reconciliation | Delivery partner hands cash to the manager at the end of each shift. Manager marks it settled. Admin views outstanding amounts. |
| **PA-09** | Final MVP scope and launch timeline | Needed for the roadmap | See DEVELOPMENT_ROADMAP |

### 4.2 Gaps found during documentation review

**Platform and product structure**

| ID | Question |
|---|---|
| **PA-10** | Do the Store Manager and Super Admin dashboards run as a **web app**, inside the **React Native app**, or both? The PDF lists only React Native. *Recommendation:* web dashboards built with React + TypeScript, sharing types with the mobile app. |
| **PA-11** | Should there be **one app with role-based screens**, or **separate apps** (Customer app and Delivery Partner app)? *Recommendation:* two Android apps, built from one codebase or monorepo. |
| **PA-12** | Is **iOS** in scope? The PDF mentions only Android Studio. *Assumption until decided:* Android only. |
| **PA-38** | App name and branding, the Play Store developer account owner, and who creates the Google Stitch designs. |
| **PA-42** | Supabase hosting region and plan. *Recommendation:* the Mumbai region (`ap-south-1`) for latency and data residency. |

**Stores, service areas and catalog**

| ID | Question |
|---|---|
| **PA-13** | How is a customer's order routed to a store: nearest store, service-area match, or customer's choice? What happens when two service areas overlap? |
| **PA-31** | Is the **product catalog** owned by each store, or is there a global catalog with store-level price and stock? (The PDF lets both the manager and the admin manage products.) |
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
| **PA-39** | Full order status list. The PDF lists only the delivery partner statuses. *Proposed:* `PLACED → ACCEPTED → PICKED_UP → OUT_FOR_DELIVERY → DELIVERED`, plus `CANCELLED` and `FAILED`. |
| **PA-40** | If no delivery partner accepts, does the manager assign manually (FR-SM-02)? Can a manager **reassign** an order that was already accepted? |
| **PA-46** | Can a customer **edit** an order after placing it? |
| **PA-14** | **Delivery partner onboarding:** who creates the accounts (manager or admin), what KYC documents are required, and is each partner tied to one store? |
| **PA-15** | **Delivery partner login method:** phone OTP (like customers) or another method? |
| **PA-24** | **Phone number privacy:** should calls between customer and delivery partner use the real numbers or masked calling? |
| **PA-36** | Maps and navigation provider (for example, opening Google Maps by deep link) and how addresses are geocoded or pinned. |

**Wallet, coupons and finance**

| ID | Question |
|---|---|
| **PA-27** | Can a **wallet balance be used to pay** for orders? Fully or partly? Is there a maximum per order? Does the balance expire? Can it be withdrawn? |
| **PA-28** | When is a **refund** given in a COD-only model, and who approves it? |
| **PA-29** | Can coupons be **combined** with each other or with wallet usage or cashback? |
| **PA-30** | **Cashback timing:** credited on delivery, or after a delay? Is there a cap per order or per customer? |
| **PA-43** | **GST and invoices:** is a tax invoice or receipt needed for each order? |
| **PA-44** | Is there **one Super Admin** or several admin users? Are there sub-roles (for example, a finance-only admin)? |

**Reviews, complaints and notifications**

| ID | Question |
|---|---|
| **PA-25** | Ratings: **products only** (as the PDF says), or also delivery partner and order? Can managers hide or reply to reviews? |
| **PA-26** | **Complaints:** how does a customer raise one? The PDF has managers and admin handling complaints but lists no complaint screen in the Customer App. |
| **PA-21** | **Notification channels:** push only, or also SMS or WhatsApp? |
| **PA-22** | **OTP SMS provider** (for example, MSG91 or Twilio) and the Indian DLT template registration owner. |

**AI, data and compliance**

| ID | Question |
|---|---|
| **PA-23** | **Voice pipeline:** Gemini end-to-end (audio in, audio out), or separate speech-to-text and text-to-speech services? Confirm Gemini model and cost budget. |
| **PA-35** | **AI cost controls:** monitoring only (as stated in the PDF), or also per-customer daily limits or caps? |
| **PA-34** | Data retention and privacy: compliance with India's DPDP Act 2023, how long to keep attendance photos and voice recordings, and whether raw audio is stored at all. |
| **PA-37** | Should **Prashant's existing customer data** be imported? In what format? Is consent needed? |
| **PA-45** | Customer **account deletion** process (required by Play Store policy). |
| **PA-47** | Are the Delivery Partner app and the dashboards translated, or English/Hindi only? |
| **PA-48** | Numeric launch targets for the success metrics (KPIs). |

## 5. Rule Change Process

1. A business owner approves the change.
2. This file is updated: the PA item becomes a `BR-xx` rule with the approval date and approver.
3. Dependent documents are updated in the same commit (SRS, schema, API, tests).
