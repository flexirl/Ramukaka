# Ramu Kaka — UI/UX Design Guidelines

| Field | Value |
|---|---|
| Design tool | **Google Stitch** (CONFIRMED, `[PDF §7]`) |
| Status | **DRAFT – screen inventory and principles. Visual designs not yet produced** |
| Last updated | 2026-10-30 (single mobile app decision) |

> Labels: **CONFIRMED** = from the PDF; **RECOMMENDATION** = proposal; **PA-xx** = pending decision.
> Branding (name usage, logo, colours) is **PA-38**. Colours below are placeholders only.

---

## 1. Confirmed Design Principles `[PDF §2, §3]`

| ID | Principle |
|---|---|
| UX-01 | **Simple wording.** Use everyday Hindi, Bhojpuri or English, not technical terms. |
| UX-02 | **Large buttons.** |
| UX-03 | **Clear icons.** |
| UX-04 | **Minimal navigation.** |
| UX-05 | Usable by customers with **limited digital literacy**. |
| UX-06 | **Voice-first.** The AI assistant is the main way to order, with manual browsing as an equal alternative. |
| UX-07 | Language selectable: **Hindi, English, Bhojpuri**. |

## 2. Recommended Design Rules

| ID | Rule | Rationale |
|---|---|---|
| UXR-01 | Touch targets at least 48×48 dp. Primary actions full-width and at least 56 dp tall. | UX-02 |
| UXR-02 | Every icon has a text label. Never use an icon alone. | UX-03, UX-05 |
| UXR-03 | Body text at least 16 sp. Prices and order totals at least 20 sp. | Readability |
| UXR-04 | At most 3–4 bottom tabs in the Customer panel (for example, **Home/Order, My Orders, Wallet, Profile**). | UX-04 |
| UXR-05 | Use product photos, not only text, for water jars and bottles. | UX-05 |
| UXR-06 | A large microphone button is always visible on the Customer home screen. | UX-06 |
| UXR-07 | Every voice reply also appears as text, along with a visual **order summary card** before confirmation. | BR-12 |
| UXR-08 | The Confirm button is visually distinct. "Change" or "Cancel" is always offered next to it. | BR-04 |
| UXR-09 | Colour contrast meets WCAG AA. Status is never shown by colour alone. | Accessibility |
| UXR-10 | Devanagari font that renders well on low-end Android devices (for example, Noto Sans Devanagari). | Hindi/Bhojpuri |
| UXR-11 | Clear offline and error states, in the user's language, with one obvious retry action. | Patchy networks |
| UXR-12 | Show money as `₹` + Indian digit grouping (e.g. ₹1,20,000). | Local convention |
| UXR-13 | Each panel has its **own bottom navigation** and a clear panel name in the header (e.g. "Ramu Kaka – Store Manager"), so staff always know which panel they are in. | BR-28 |
| UXR-14 | Store Manager and Super Admin screens are designed **for phones**: summary cards, filters and short lists instead of wide tables; one main action per screen. | BR-27 (no web dashboard) |
| UXR-15 | Before any permission request, show a short screen explaining why it is needed, in the user's language. | BR-28 |

## 3. Screen Inventory

All screens below belong to **one Ramu Kaka app** (BR-27). The shared app-shell screens come first; then each panel has its own set of screens and its own navigation (BR-28).

### 3.0 Shared App Shell

| # | Screen | Requirements |
|---|---|---|
| SH-S01 | Splash + language picker (first launch) | FR-CU-03, FR-APP-01 |
| SH-S02 | Phone number entry (all roles) | FR-CU-01, BR-26 |
| SH-S03 | OTP verification (all roles) | FR-CU-01, BR-26 |
| SH-S04 | Checking access (loading while the role is read from the backend) | FR-APP-02 |
| SH-S05 | Account suspended | FR-APP-05 |
| SH-S06 | Waiting for approval (staff accounts not yet approved) | FR-APP-05 |
| SH-S07 | No access | FR-APP-05 |
| SH-S08 | Permission explanation sheet (reused before each permission request) | FR-APP-07, UXR-15 |
| SH-S09 | Panel switcher (only if **PA-60** allows more than one role) | PA-60 |

### 3.1 Customer Panel

| # | Screen | Requirements |
|---|---|---|
| CU-S01–03 | (Moved to the shared app shell: SH-S01–SH-S03) | — |
| CU-S03a | Name entry for new customers (DERIVED from FR-DP-07) | FR-CU-01 |
| CU-S04 | Add first address (map pin + form) | FR-CU-02, BR-02 (**PA-36**) |
| CU-S05 | "Not serviceable yet" state | BR-02 |
| CU-S06 | **Home**: big mic button, quick-order tiles (e.g. 20 L jar), categories | FR-CU-04, FR-CU-07 |
| CU-S07 | **Voice assistant** sheet: listening, thinking and speaking states, transcript, summary card, Confirm/Change | FR-AI-01…06 |
| CU-S08 | Category / product list | FR-CU-07/08 |
| CU-S09 | Product detail + reviews | FR-CU-13 |
| CU-S10 | Cart / order summary (server quote, coupon, wallet **PA-27**) | FR-CU-06, FR-CU-12 |
| CU-S11 | Order placed confirmation | BR-04 |
| CU-S12 | Order tracking (status steps, DP name, contact button; method **PA-24**) | FR-CU-09, FR-CU-14 |
| CU-S13 | My orders (history) | FR-CU-09 |
| CU-S14 | Rate and review | FR-CU-13 (**PA-25**) |
| CU-S15 | Wallet: balance + transactions | FR-CU-11 |
| CU-S16 | Offers / coupons | FR-CU-12 |
| CU-S17 | Addresses list/edit | FR-CU-02 |
| CU-S18 | Profile & settings (language, logout, delete account **PA-45**) | FR-CU-03 |
| CU-S19 | Notification inbox (list, mark read) | FR-CU-10 |
| CU-S20 | Help / complaint / feedback | **PA-26** |
| CU-S21 | Store closed / out-of-stock states | BR-09 |

### 3.2 Delivery Partner Panel

| # | Screen | Requirements |
|---|---|---|
| DP-S01 | (Uses the shared login, SH-S02/03; extra checks **PA-15**) | FR-DP-01 |
| DP-S02 | Home: large **On Duty / Off Duty** switch, today's summary | FR-DP-02 |
| DP-S03 | Start-of-work photo capture | FR-DP-03 (**PA-06**) |
| DP-S04 | New **delivery request** (full-screen card with Accept / Reject; contents before acceptance **PA-56**; timer **PA-03**) | FR-DP-04/05 |
| DP-S05 | "Order already taken" message | BR-03 |
| DP-S06 | Active delivery: customer, address, items, Call, Navigate, status button | FR-DP-07/08/09 |
| DP-S07 | Deliver + collect COD (amount due, amount collected) | FR-DP-10 |
| DP-S08 | Report failed delivery / issue | FR-DP-11 (**PA-17**) |
| DP-S09 | History & performance; attendance log | FR-DP-12 |
| DP-S10 | Profile (editable fields **PA-14**) | FR-DP-01 |

**Rule:** the status button changes its label at each step ("Picked Up" → "Out for Delivery" → "Delivered"), so the DP only ever sees one next action (BR-11).

### 3.3 Store Manager Panel (phone screens, UXR-14)

| # | Screen | Requirements |
|---|---|---|
| SM-S01 | (Uses the shared login, SH-S02/03) | — |
| SM-S02 | Overview: open/close toggle (precedence **PA-50**), order counters, DPs on duty, alerts (unaccepted orders) | FR-SM-01/04 |
| SM-S03 | Orders board (Pending / Active / Completed / Failed), assign action | FR-SM-01/02/03 |
| SM-S04 | Delivery partners: duty status, attendance, photo verification queue | FR-SM-02/15 |
| SM-S05 | Categories | FR-SM-06 |
| SM-S06 | Products & stock (saved through server functions, every change audit-logged) | FR-SM-07–10, DR-11 |
| SM-S07 | Surge / delivery charges | FR-SM-05 |
| SM-S08 | Reports: revenue, orders, cancellations, failures | FR-SM-03/11 |
| SM-S09 | COD reconciliation | FR-SM-12 |
| SM-S10 | Coupons & cashback (if permitted) | FR-SM-13 |
| SM-S11 | Reviews, complaints & feedback | FR-SM-14 (**PA-25**, **PA-26**) |

### 3.4 Super Admin Panel (phone screens, UXR-14)

| # | Screen | Requirements |
|---|---|---|
| SA-S01 | Platform overview (orders, revenue, store/manager/DP comparison; metrics **PA-55**) | FR-SA-02/09 |
| SA-S02 | Stores (list, create, suspend, open/close, settings & limits) | FR-SA-01/03/04/12 |
| SA-S03 | Managers (create, approve, suspend, permissions) | FR-SA-03/12 |
| SA-S04 | Delivery partners (create, approve, suspend, attendance and photos) | FR-SA-01 (**PA-14**) |
| SA-S05 | Customers (+ wallet credit action; other crediting actors **PA-52**) | FR-SA-01/07 |
| SA-S06 | Platform catalog / pricing / operational settings (list **PA-51**) | FR-SA-05 |
| SA-S07 | Offers, coupons, cashback | FR-SA-06 |
| SA-S08 | Finance: wallet ledger, COD settlements, promo spend | FR-SA-08 |
| SA-S09 | Reviews & complaints | FR-SA-10 |
| SA-S10 | AI usage & cost (reporting currency **PA-57**) | FR-SA-11 |
| SA-S11 | Audit log | NFR-SEC-07 |
| SA-S12 | User roles: pre-register a staff phone number and assign its role and store | FR-APP-04, FR-SA-03 |

## 4. Voice Assistant UX (RECOMMENDATION)

| State | Visual | Audio |
|---|---|---|
| Idle | Large mic button + hint text ("Boliye – 'ek jar paani bhejo'") | — |
| Listening | Pulsing mic, live level meter, "Sun raha hoon…" | Short start tone |
| Thinking | Animated indicator | — |
| Speaking | Reply text + summary card | TTS reply |
| Awaiting confirmation | Summary card with big **Haan, order karo** / **Badlo** buttons | "Confirm karein?" |
| Unclear reply | Summary card stays. "Kya order karun? Haan ya nahi boliye, ya button dabaiye" | Re-asks. **No order placed** |
| Order placed | Shown **only after the server confirms placement**, with the order number | Confirmation message |
| Failure (×2) | "Samajh nahi aaya" + **Order manually** button, with items understood so far pre-filled | Spoken apology |

The customer can confirm by voice **or** by tapping the button (DR-10):
- A **tap** confirms the exact quote on the card.
- A **spoken "yes"** is checked by a separate server-side yes/no check. Unclear replies never place an order.
- The AI model itself cannot place orders, so the UI never shows "order placed" based on the AI's words alone.

> Hindi and Bhojpuri phrases above are illustrative placeholders. Final copy should be reviewed by native speakers (**PA-07**).

## 5. Design Tokens (placeholders until branding **PA-38**)

| Token | Placeholder |
|---|---|
| color.primary | Water blue (TBD) |
| color.success / warning / danger | Green / Amber / Red (TBD) |
| radius.card | 12 dp |
| spacing.base | 8 dp grid |
| font.family | Noto Sans + Noto Sans Devanagari |

## 6. Design Deliverables (Level 1 – Design)

1. Low-fidelity wireframes of every screen in §3, made in Google Stitch.
2. High-fidelity designs for the shared app shell and the Customer and Delivery Partner panels (Hindi first, English and Bhojpuri variants).
3. Store Manager and Super Admin panel designs, for phone screens (UXR-14).
4. Usability test of the order flows with 5–8 target users in Jamui (RECOMMENDATION).
