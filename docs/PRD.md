# Ramu Kaka — Product Requirements Document (PRD)

| Field | Value |
|---|---|
| Source of truth | *Ramu Kaka PRD v1.0* (PDF), prepared by Anubhav Kumar, FlexIRL |
| Source status | Initial Draft – For Discussion and Approval |
| This document | Structured restatement of the source PRD with traceability IDs |
| Level | Level 0 – Project Planning and Documentation |
| Status | **DRAFT – awaiting stakeholder approval** |
| Last updated | 2026-10-30 (single mobile app decision; no PDF requirement changed) |

> **How to read this document**
> - **CONFIRMED** = stated in the source PDF. The section reference is shown as `[PDF §n]`.
> - **PENDING APPROVAL** = not decided in the PDF. Each item has a `PA-xx` ID. The full list is in [BUSINESS_RULES.md §4](./BUSINESS_RULES.md#4-pending-approval-register).
> - **APPROVED DECISION** = a pending item the project owner has decided (date and approver recorded in BUSINESS_RULES §4.0).
> - **RECOMMENDATION** = a proposal from the documentation team. It is **not** a requirement until it is approved.

---

## 1. Product Overview — CONFIRMED `[PDF §1]`

Ramu Kaka is a **voice-first, AI-powered delivery platform** for ordering daily essentials. Customers order in one of two ways:

1. Through a natural voice conversation with the **Ramu Kaka AI assistant**.
2. Through a **manual ordering interface**.

| Item | Confirmed detail |
|---|---|
| Initial category | Water delivery: bottles and jars |
| Launch business | Prashant's existing water-supply business and customer base |
| Future categories | Milk, curd, paneer, groceries and other daily essentials |
| Vision | "Make ordering everyday essentials as easy as talking to someone." |
| Initial market | Selected districts and towns in Bihar, starting with areas such as **Jamui** |

## 2. Product Objectives — CONFIRMED `[PDF §2]`

| ID | Objective |
|---|---|
| OBJ-01 | Simplify ordering through voice commands. |
| OBJ-02 | Support customers with limited digital literacy. |
| OBJ-03 | Digitize existing water-supply and delivery operations. |
| OBJ-04 | Provide store-wise management and centralized business monitoring. |
| OBJ-05 | Track revenue, delivery performance, cashback and AI usage costs. |
| OBJ-06 | Build a scalable platform for future product categories and locations. |

## 3. Language Support — CONFIRMED `[PDF §3]`

- Supported languages: **Hindi, English, Bhojpuri**.
- Customers can select their preferred language.
- The AI assistant understands voice commands and responds in the selected language, *"subject to recognition and model capabilities"*.
- Mixed-language conversations (for example, Hinglish) are supported *"where feasible"*.
- The interface uses **simple wording, large buttons, clear icons and minimal navigation**.

PENDING APPROVAL:
- **PA-07**: the expected voice quality for each language, especially Bhojpuri.
- **PA-47**: whether the Delivery Partner, Store Manager and Super Admin panels are also translated, or only the Customer panel.

## 4. Users and Panels — CONFIRMED `[PDF §4]`

> **APPROVED DECISION (2026-10-30, BR-27, BR-28):** all four panels are parts of **one React Native mobile app**. After login, the app opens the panel that matches the user's backend-assigned role. The PDF's names "Customer App", "Delivery Partner App", "Store Manager Dashboard" and "Super Admin Dashboard" are kept below as the panel names. They are not separate applications, and there is no web dashboard.

| Panel | Primary user | Scope |
|---|---|---|
| 1. Customer App | End customer | Ordering, wallet, reviews |
| 2. Delivery Partner App | Delivery staff | Attendance, accepting orders, delivery, COD |
| 3. Store / Warehouse Manager Dashboard | Store manager (one store each) | Orders, inventory, offers, store control |
| 4. Super Admin / Owner Dashboard | Owner / platform admin | Platform-wide control and finance |

Open questions about the panels:
- The PDF did not say where the dashboards run (**PA-10**). **Decided 2026-10-30:** inside the single mobile app (BR-27). PA-11 (one app or several) was decided the same way.
- The PDF says "Store / Warehouse Manager" and "work-start photo from the warehouse". Whether a store and a warehouse are always the same place → **PA-54**.
- Authentication is "Supabase Auth with OTP verification" `[PDF §7]`, with no per-panel exception, so OTP applies to all four roles (BR-26).
- "Offers" in this PRD means promotional offers. Dispatching an order to delivery partners is called a **delivery request** in the technical documents.

### 4.1 Panel 1 — Customer App `[PDF §4, Panel 1]`

| ID | Requirement |
|---|---|
| FR-CU-01 | OTP-based registration and login. |
| FR-CU-02 | Add, edit and save delivery addresses. |
| FR-CU-03 | Select Hindi, English or Bhojpuri. |
| FR-CU-04 | Place orders using voice commands or manual browsing. |
| FR-CU-05 | Talk to the Ramu Kaka AI assistant through voice conversations. |
| FR-CU-06 | Confirm product, quantity, address and price before placing an order. |
| FR-CU-07 | Browse product categories, prices and availability. |
| FR-CU-08 | At launch, order water bottles and jars. |
| FR-CU-09 | View order status, delivery details and order history. |
| FR-CU-10 | Receive order notifications. |
| FR-CU-11 | Use an in-app wallet for cashback, promotional credits and approved refunds. |
| FR-CU-12 | Apply coupons and available offers. |
| FR-CU-13 | Submit product ratings and reviews. |
| FR-CU-14 | Contact delivery partners when necessary. |

### 4.2 Panel 2 — Delivery Partner App `[PDF §4, Panel 2]`

| ID | Requirement |
|---|---|
| FR-DP-01 | Secure login and profile management. |
| FR-DP-02 | Mark attendance and switch between On Duty and Off Duty. |
| FR-DP-03 | Upload a work-start photo from the warehouse, subject to verification. |
| FR-DP-04 | Receive notifications for available delivery orders. |
| FR-DP-05 | Accept or reject orders. |
| FR-DP-06 | An order is assigned to the **first eligible delivery partner who accepts it**, subject to server-side checks. |
| FR-DP-07 | View customer name, phone number, delivery address and ordered items. |
| FR-DP-08 | Call customers and open navigation. |
| FR-DP-09 | Update order status: **Accepted → Picked Up → Out for Delivery → Delivered**. |
| FR-DP-10 | Record Cash on Delivery (COD) collections. |
| FR-DP-11 | Report failed deliveries and other issues. |
| FR-DP-12 | View daily attendance, delivery history and performance. |

### 4.3 Panel 3 — Store / Warehouse Manager Dashboard `[PDF §4, Panel 3]`

Each store has its own manager and a separate dashboard. A manager can access **only their assigned store**. (In this project, the "separate dashboard" is the Store Manager panel of the single app, showing only that manager's store.)

| ID | Area | Requirement |
|---|---|---|
| FR-SM-01 | A. Orders | View total, pending, active and completed orders. |
| FR-SM-02 | A. Orders | Monitor delivery partners and assign orders when required. |
| FR-SM-03 | A. Orders | Track delivery performance, cancellations and failed orders. |
| FR-SM-04 | A. Orders | Open or close the store and control order acceptance. |
| FR-SM-05 | A. Orders | Apply **authorized** surge pricing or additional delivery charges. |
| FR-SM-06 | B. Catalog | Create, edit and deactivate product categories. |
| FR-SM-07 | B. Catalog | Add products, images, descriptions, prices and stock quantities. |
| FR-SM-08 | B. Catalog | Manage water products during the initial phase. |
| FR-SM-09 | B. Catalog | Add future categories such as dairy, groceries and daily essentials. |
| FR-SM-10 | B. Catalog | Update product availability and manage stock. |
| FR-SM-11 | C. Sales | View store revenue and order reports. |
| FR-SM-12 | C. Sales | Reconcile cash collected through COD. |
| FR-SM-13 | C. Sales | Create coupons, cashback campaigns and offers **within assigned permissions**. |
| FR-SM-14 | C. Sales | Manage customer reviews, complaints and feedback. |
| FR-SM-15 | C. Sales | Monitor delivery partner attendance and performance. |

### 4.4 Panel 4 — Super Admin / Owner Dashboard `[PDF §4, Panel 4]`

| ID | Requirement |
|---|---|
| FR-SA-01 | View and manage all stores, managers, customers and delivery partners. |
| FR-SA-02 | Monitor total orders, revenue and store-wise performance. |
| FR-SA-03 | Create, approve, suspend and manage store and manager accounts. |
| FR-SA-04 | Control store opening and closing settings. |
| FR-SA-05 | Manage platform-wide product, pricing and operational settings. |
| FR-SA-06 | Create platform-wide offers, coupons and cashback campaigns. |
| FR-SA-07 | Credit approved cashback, refunds or compensation to customer wallets. |
| FR-SA-08 | Monitor wallet transactions, COD settlements and financial reports. |
| FR-SA-09 | Compare store, manager and delivery partner performance. |
| FR-SA-10 | Monitor customer complaints and reviews. |
| FR-SA-11 | Track AI usage, estimated cost per customer and total AI expenditure. |
| FR-SA-12 | Manage user permissions and store-level operating limits. |

## 5. AI Voice Assistant — CONFIRMED `[PDF §5]`

The AI assistant is the **core differentiating feature**.

**Reference interaction (from PDF):**
1. Customer: *"Ramu Kaka, ek 20 litre paani ka jar bhej do."* ("Ramu Kaka, send one 20-litre water jar.")
2. Assistant: confirms product, quantity, delivery address and applicable price.
3. Customer: confirms.
4. Assistant: places the order and gives the order confirmation and status.

| ID | Requirement |
|---|---|
| FR-AI-01 | Voice input and spoken responses. |
| FR-AI-02 | Hindi, English and Bhojpuri support. |
| FR-AI-03 | Context-aware conversations for product selection, quantity and saved addresses. |
| FR-AI-04 | Asks clarifying questions when required details are missing. |
| FR-AI-05 | Explicit customer confirmation before an order is placed. |
| FR-AI-06 | Manual ordering fallback when voice recognition fails. |
| FR-AI-07 | Secure, server-side AI usage logging and cost monitoring. |
| FR-AI-08 | The AI must use **actual product prices, stock availability and delivery rules from the backend**. |

## 6. Wallet, Payments and Offers — CONFIRMED `[PDF §6]`

| ID | Requirement |
|---|---|
| FR-WP-01 | Cash on Delivery (COD) is the initial payment method. |
| FR-WP-02 | Customer wallets hold cashback, promotional credits and approved refunds. |
| FR-WP-03 | Cashback and coupon eligibility follow predefined business rules (rules are **PENDING**, PA-04). |
| FR-WP-04 | Every wallet credit and debit has a secure transaction record. |
| FR-WP-05 | The Super Admin can issue approved wallet credits. (Whether others may also credit wallets is **PA-52**.) |
| FR-WP-06 | Online payments (Razorpay or another gateway) **may** be introduced later (Phase 2, "if required"). |
| FR-WP-07 | The dashboard tracks COD collections, outstanding settlements and promotional expenses. |

## 7. Technology Stack — CONFIRMED `[PDF §7]`

| Layer | Confirmed choice |
|---|---|
| UI/UX design | Google Stitch |
| Mobile application | React Native with TypeScript |
| Authentication | Supabase Auth with OTP verification |
| Database | Supabase PostgreSQL |
| Backend | Supabase services and server-side functions |
| AI assistant | Gemini API |
| AI-assisted coding | Claude AI (vibe-coding workflow) |
| Android build and deployment | Android Studio |
| Version control | GitHub |

**App packaging (APPROVED DECISION, BR-27):** one React Native app containing all four panels. No separate apps, no web dashboard.

**Architecture requirement (CONFIRMED):** all panels share one backend and one database, with role-based access control. Wallet credits, order assignment, pricing and financial calculations must be validated **server-side**. "API keys must not be exposed in the mobile application." Secret keys never ship. Whether platform-restricted public keys (Maps, FCM) are acceptable is **PA-53**.

See [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) for the proposed design.

## 8. Development Phases — CONFIRMED `[PDF §8]`

| Phase | Scope |
|---|---|
| **Phase 1 – Water Delivery MVP** | Customer app (OTP, addresses, voice/manual ordering), 3 languages; Delivery Partner app (acceptance, status, attendance); Store Manager dashboard (products, inventory, delivery ops); Super Admin dashboard (stores, revenue, offers, wallet controls); COD management, basic cashback, notifications; basic AI usage and cost tracking. |
| **Phase 2 – Optimization** | Better voice recognition and language handling; better analytics and reporting; advanced offers and engagement; online payments if required; better attendance verification and financial reconciliation. |
| **Phase 3 – Business Expansion** | Milk, curd, paneer, groceries and daily essentials; more districts and service areas; more stores, suppliers and delivery partners; more automation and BI. |

In this project, each "app" and "dashboard" in the phase list is a panel of the single Ramu Kaka app (BR-27). The engineering breakdown of these phases into levels is in [DEVELOPMENT_ROADMAP.md](./DEVELOPMENT_ROADMAP.md).

## 9. Key Business Rules — CONFIRMED `[PDF §9]`

These rules are listed with IDs `BR-01` to `BR-09` in [BUSINESS_RULES.md](./BUSINESS_RULES.md).

## 10. Success Metrics — CONFIRMED `[PDF §10]`

| ID | Metric |
|---|---|
| KPI-01 | Registered and active customers |
| KPI-02 | Total orders and successful delivery rate |
| KPI-03 | Average order fulfilment time |
| KPI-04 | Order cancellation and failure rate |
| KPI-05 | Store revenue and average order value |
| KPI-06 | Delivery partner attendance and productivity |
| KPI-07 | Repeat purchase rate and customer ratings |
| KPI-08 | Cashback and promotional expenditure |
| KPI-09 | AI cost per customer and per completed order |

The PDF does not define these metrics or set numeric targets:
- **PA-55**: definitions (e.g. "active customer", "fulfilment time", manager performance). Candidate definitions are in [SRS §3.8](./SRS.md#38-kpi-definitions--pending-approval-pa-55-pa-48).
- **PA-48**: should launch targets be set?

## 11. Items Requiring Final Approval — from PDF `[PDF §11]`

The PDF itself lists these as undecided:

| ID | Item |
|---|---|
| PA-01 | Initial launch locations and service radius |
| PA-02 | Supplier inventory and water pricing model (supplier scope split out as **PA-59**) |
| PA-03 | Delivery assignment rules and acceptance timeout |
| PA-04 | Cashback, wallet refund and coupon policies |
| PA-05 | Store-level surge pricing permissions |
| PA-06 | Attendance photo verification requirements |
| PA-07 | Hindi, English and Bhojpuri voice support expectations |
| PA-08 | COD settlement and financial reporting rules |
| PA-09 | Final MVP scope and launch timeline |

The documentation review found further gaps, `PA-10` to `PA-62` (PA-10 and PA-11 are now decided). They are listed in [BUSINESS_RULES.md §4](./BUSINESS_RULES.md#4-pending-approval-register).

## 12. Out of Scope for Phase 1 (per PDF phasing)

- Online payments (Phase 2, "if required").
- Non-water categories (Phase 3). The data model must still *allow* them (OBJ-06, FR-SM-09).
- Expansion beyond the initial launch districts (Phase 3).
