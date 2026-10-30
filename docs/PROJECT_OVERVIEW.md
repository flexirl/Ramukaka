# Ramu Kaka — Project Overview (Simple Version)

| Field | Value |
|---|---|
| Who this is for | Anyone who wants to understand Ramu Kaka without technical details |
| Based on | Ramu Kaka PRD v1.0 (PDF) and approved decisions |
| Status | **DRAFT for review** |
| Last updated | 2026-10-30 |

Items marked **(pending)** are not decided yet. They are listed at the end.

---

## 1. What is Ramu Kaka?

- A mobile app for ordering daily essentials.
- You can **talk** to it, like talking to a helpful elder ("Ramu Kaka"), or tap through the screens.
- It starts with **drinking water**: bottles and 20-litre jars.
- It is built on **Prashant's existing water-supply business** and customers.
- The first area is **Bihar**, starting with **Jamui**.
- Goal: *"Make ordering everyday essentials as easy as talking to someone."*

## 2. What problem does it solve?

- Many people find shopping apps hard to use. Ramu Kaka lets them **just speak** in Hindi, English or Bhojpuri.
- Water orders today are often taken by phone call and noted by hand. Ramu Kaka **puts the whole business on the app**.
- Owners can't easily see sales, deliveries and cash. Ramu Kaka shows this **for each store and for the whole business**.
- It also tracks **cashback** and the **cost of the AI assistant**, so spending stays under control.

## 3. One app, four panels

- There is **only one app**: the Ramu Kaka app on Android.
- Everyone installs the same app: customers, delivery partners, store managers and the owner.
- Inside the app there are **four panels**, one for each type of user:
  1. **Customer**
  2. **Delivery Partner**
  3. **Store Manager**
  4. **Super Admin** (the owner)
- There is **no separate app and no website** for anyone.

## 4. Login and how the app knows which panel to show

**Login**
- Open the app and choose a language.
- Enter your mobile number.
- Type the one-time code (OTP) you receive by SMS.

**Choosing the panel (automatic)**
- After login, the app asks the server: *"What is this person's role?"*
- The server answers: customer, delivery partner, store manager or super admin.
- The app opens **that panel only**.

**Who gives the role?**
- Anyone who signs up by themselves becomes a **customer**.
- Delivery partners, store managers and admins are **added by the Super Admin**. Their phone number is registered with the right role and store before they log in.
- **Nobody can change their own role.**

**Safety**
- Each panel has its own screens and menus. One panel cannot open another panel's screens.
- The **server checks every action**. Even if someone tampers with their copy of the app, the server refuses anything their role is not allowed to do.
- If the Super Admin **suspends** someone, they lose access immediately.
- Special screens appear for: "Account suspended", "Waiting for approval" and "No access".
- The app asks for phone permissions (microphone, camera, location) **only when that user's work needs it**. For example, customers are never asked for the camera. Nobody's location is tracked in the background.

## 5. Panel 1 — Customer

- Log in with mobile number and OTP.
- Choose a language: Hindi, English or Bhojpuri.
- Save delivery addresses (home, shop and so on) and edit them.
- Order by **voice** ("Ramu Kaka, ek 20 litre jar bhej do") or by **tapping** through products.
- See product categories, prices and what is available.
- Order water bottles and jars (more products later).
- See a clear summary (item, quantity, address, price) and **confirm before ordering**.
- Pay by **Cash on Delivery**.
- Track the order live and see past orders.
- Get notifications about the order.
- Wallet: see cashback, promotional credit and approved refunds.
- Use coupons and offers.
- Rate and review products.
- Contact the delivery partner when needed.

## 6. Panel 2 — Delivery Partner

- Log in and manage their profile.
- Switch **On Duty / Off Duty** (attendance).
- Take a **start-of-work photo at the warehouse**, which is checked.
- Get notified when a new delivery is available.
- **Accept or reject** deliveries. The first partner to accept gets the order, and no two partners can get the same order.
- See the customer's name, phone, address and items (after accepting).
- Call the customer and open map directions.
- Update the order step by step: **Accepted → Picked Up → Out for Delivery → Delivered**.
- Record the **cash collected**.
- Report a failed delivery or other problem.
- See their attendance, delivery history and performance.

## 7. Panel 3 — Store Manager

Each store has its own manager. A manager sees **only their own store**.

**Orders and delivery**
- See total, pending, active and completed orders.
- See which delivery partners are on duty, and assign orders when needed.
- Track delivery performance, cancellations and failed deliveries.
- **Open or close the store** for new orders.
- Add surge or extra delivery charges, only if the owner allows it.

**Products and stock**
- Create, edit and switch off product categories.
- Add products with photos, descriptions, prices and stock.
- Manage water products now; add dairy, groceries and more later.
- Update stock and availability.

**Sales and customers**
- See store sales and order reports.
- Match the **cash collected** by delivery partners with what was handed over.
- Create store coupons, cashback and offers (if allowed).
- Handle customer reviews, complaints and feedback.
- Check delivery partner attendance and performance.

## 8. Panel 4 — Super Admin (Owner)

- See and manage **all stores, managers, customers and delivery partners**.
- Add staff and give them their role and store.
- See total orders, revenue and each store's performance.
- Create, approve, suspend and manage stores and manager accounts.
- Control store opening and closing settings.
- Set platform-wide product, pricing and operating settings.
- Create platform-wide offers, coupons and cashback.
- Add cashback, refunds or compensation to a customer's wallet.
- Check wallet activity, cash settlements and financial reports.
- Compare stores, managers and delivery partners.
- Monitor complaints and reviews.
- Track **AI cost**: total, per customer and per completed order.
- Control user permissions and limits for each store.

## 9. How ordering and delivery work

1. The customer chooses items by voice or by tapping.
2. The server works out the **real price**: item price, delivery charge, any surge, any coupon.
3. The customer sees the summary and **confirms**. Without confirmation, no order is placed.
4. The server checks again that the store is open, the item is in stock and the address is in the delivery area. Then the order is created.
5. On-duty delivery partners of that store get a **delivery request**.
6. The **first** partner to accept gets the order.
7. If nobody accepts in time, the store manager is alerted and can assign it **(timeout pending)**.
8. The partner picks up the order at the warehouse and marks **Picked Up**, then **Out for Delivery**.
9. At the door, the partner collects the cash and marks **Delivered**.
10. The customer is notified at every step.
11. Later, the partner hands the cash to the store manager, who records it **(rules pending)**.

## 10. How the AI voice assistant works

- The customer taps the **microphone** and speaks naturally, in Hindi, English or Bhojpuri (mixing languages is fine where possible).
- Ramu Kaka understands the product, quantity and address. If something is missing, it **asks a question** (e.g. "Ghar ka pata ya dukaan ka?").
- It gets the **real price and stock** from the system. It never guesses prices.
- It reads back the order and shows a summary card with **Confirm** and **Change** buttons.
- The customer confirms by tapping **Confirm** or by clearly saying **"haan"**.
- A separate safety check on the server makes sure it was really a "yes". If the answer is unclear, Ramu Kaka asks again. **The AI itself can never place an order or change a price.**
- If voice doesn't work, the customer can switch to **ordering by tapping**, with the items already filled in.
- The AI runs only on the server, never inside the phone, and every use is recorded so its **cost can be tracked**.

## 11. Technology used

| What | Tool |
|---|---|
| App design | Google Stitch |
| The app (all four panels) | React Native with TypeScript (Android, built with Android Studio) |
| Login | Supabase Auth with OTP |
| Database | Supabase (PostgreSQL) |
| Server functions | Supabase |
| AI assistant | Google Gemini |
| AI-assisted coding | Claude (Claude Code) |
| Code storage | GitHub |

- One shared server and database for everyone.
- Money, prices, order assignment and wallet changes are all **calculated and checked on the server**, never on the phone.
- Secret keys are **never put inside the app**.

## 12. Future plans

**Phase 1 — Water delivery launch (now)**
- The full app with all four panels, voice and manual ordering, three languages, cash on delivery, basic cashback, notifications and AI cost tracking.

**Phase 2 — Improvements**
- Better voice understanding and language handling.
- Better reports and analytics.
- More offers and customer engagement.
- Online payments (e.g. Razorpay), if needed.
- Better attendance checks and cash reconciliation.

**Phase 3 — Growth**
- Milk, curd, paneer, groceries and other daily essentials.
- More districts and towns.
- More stores, suppliers and delivery partners.
- More automation and business insights.

## 13. Decisions already made

- **One app** for all four roles, with **no website** (decided 2026-10-30).
- Login with **mobile number + OTP** for everyone (from the PRD).
- The **AI cannot place orders by itself**. The customer must confirm (approved during documentation review).

## 14. Business decisions still pending

These need a decision before or during development. The ID in brackets is the reference in [BUSINESS_RULES.md](./BUSINESS_RULES.md).

**Launch and locations**
- Which towns at launch, and how far each store delivers (PA-01).
- How an order is matched to a store (PA-13).
- Final first-version scope and launch date (PA-09).
- Whether iPhone is supported (PA-12). This matters more now, because managers and the owner can only use the app.
- App name, branding, Play Store account owner and who designs the screens (PA-38).
- Server location and plan (PA-42).

**Products, prices and stores**
- Water pricing model and supplier stock (PA-02); whether suppliers are part of the first version (PA-59).
- One shared product list, or a separate list per store (PA-31).
- Exactly what managers are allowed to do (PA-32).
- Store opening hours: manual switch, schedule or both (PA-33); who wins if manager and owner settings conflict (PA-50).
- Whether a store can have more than one manager (PA-41).
- Whether a store and its warehouse are the same place (PA-54).
- Empty jar return or jar deposit rules (PA-18).
- Platform-wide settings the owner controls (PA-51).

**Orders and delivery**
- How deliveries are offered to partners, and how long they have to accept (PA-03).
- What happens if nobody accepts, and whether orders can be reassigned (PA-40).
- What a partner sees before accepting (PA-56).
- The full list of order stages (PA-39).
- Cancellation rules (PA-16), failed delivery rules (PA-17), and whether orders can be edited (PA-46).
- When stock is reduced, and when it is returned (PA-49).
- Delivery charges and minimum order value (PA-19); surge charge limits (PA-05).
- Only instant delivery, or also scheduled/regular orders (PA-20).
- Partner onboarding, documents and editable profile details (PA-14); any extra partner login check (PA-15).
- How customers and partners contact each other, and whether phone numbers are hidden (PA-24).
- Maps and address pinning (PA-36).
- Start-of-work photo checks (PA-06).

**Money, wallet and offers**
- Cashback, refund and coupon policies (PA-04).
- Whether wallet money can pay for orders, and whether it expires (PA-27).
- When refunds happen with cash on delivery (PA-28).
- Whether coupons can be combined (PA-29).
- When cashback is credited, and limits (PA-30).
- Who besides the owner can add money to wallets (PA-52).
- Whether managers can see customer wallets (PA-58).
- Cash settlement and financial report rules (PA-08).
- GST invoices or receipts (PA-43).
- One owner account or several admins; two-person approval for wallet credits (PA-44).

**Reviews, complaints and notifications**
- What can be rated, and whether managers can reply or hide reviews (PA-25).
- How customers raise complaints and feedback (PA-26).
- Notification channels: app notifications only, or also SMS/WhatsApp (PA-21).
- SMS provider for OTP and the required DLT registration (PA-22).

**AI, privacy and app security**
- Expected voice quality in each language, especially Bhojpuri (PA-07).
- How voice is processed, which Gemini model, and the budget (PA-23).
- AI spending limits per customer (PA-35).
- Which currency to show AI costs in (PA-57).
- Privacy rules, and whether voice recordings and photos are kept (PA-34).
- Importing Prashant's existing customer list (PA-37).
- How customers delete their account (PA-45).
- Whether staff panels are translated into all three languages (PA-47).
- Allowing restricted map/notification keys in the app (PA-53).
- Whether one phone number can have more than one role (PA-60).
- App lock or auto-logout for managers and the owner (PA-61).
- Whether reports can be downloaded as files (PA-62).

**Success measures**
- How to measure success (definitions) (PA-55) and the launch targets (PA-48).
