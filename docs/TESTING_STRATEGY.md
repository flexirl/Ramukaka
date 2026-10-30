# Ramu Kaka — Testing Strategy

| Field | Value |
|---|---|
| Status | **DRAFT – RECOMMENDATION** (the PDF sets no testing requirements; the confirmed rules below set what must be tested) |

---

## 1. Goals

1. Prove every **confirmed business rule** (BR-01…BR-25) with automated tests.
2. Protect money and order integrity: wallet, COD, pricing, assignment.
3. Measure AI voice ordering quality in **Hindi, English and Bhojpuri** against the targets set in **PA-07**.
4. Check the app works on the **low-end Android devices and weak networks** expected in Bihar.

## 2. Test Levels

| Level | Scope | Tooling (RECOMMENDATION) | When |
|---|---|---|---|
| Static checks | TypeScript strict mode, ESLint, Prettier | `tsc`, ESLint | Every commit (CI) |
| Unit | Pure logic: price calculation, coupon rules, status transitions, i18n keys | Jest / Vitest | Every commit |
| Database / RLS | RLS policies, Postgres functions, constraints | pgTAP or SQL tests via Supabase CLI | Every commit touching `supabase/` |
| Edge Function integration | Functions against a local Supabase instance | Deno test / Vitest + Supabase CLI | Every PR |
| Component | React Native screens and components | React Native Testing Library | Every PR |
| End-to-end (mobile) | Main user flows on an emulator or device | Maestro or Detox | Nightly + before release |
| End-to-end (dashboards) | Manager/Admin flows | Playwright (if web, **PA-10**) | Nightly + before release |
| AI evaluation | Voice/text conversation test sets | Custom eval harness against the staging AI function | Each prompt or model change |
| Performance | Latency and load on critical endpoints | k6 or similar | Before pilot |
| Security | RLS bypass attempts, secret scanning, dependency audit | Manual + automated | Each level gate |
| UAT | Real users (customers, DPs, manager) in Jamui | Scripted sessions | Level 11 |

## 3. Business-Rule Test Matrix (must pass before pilot)

| Rule | Test cases |
|---|---|
| BR-01 Store isolation | Manager A cannot read or write Store B's orders, products, DPs, reports, coupons (positive + negative tests for every table) |
| BR-02 Service area | Address inside the area → quote OK. Outside → `OUT_OF_SERVICE_AREA`. Boundary cases (**PA-01**) |
| BR-03 / BR-10 Single assignment | **Concurrency test:** N delivery partners accept the same order at the same moment → exactly 1 succeeds, N-1 get `ORDER_ALREADY_ASSIGNED`. Off-duty DP cannot accept. |
| BR-04 / BR-12 Confirmation | Order cannot be created without a valid confirmed quote. AI cannot place an order without a confirmation turn. |
| BR-05 / BR-17 Ledger | No UPDATE/DELETE on wallet transactions through any role. Balance = sum of the ledger. Idempotent credit (same key twice → one entry). |
| BR-09 Closed / out of stock | Closed store → `STORE_CLOSED`. Stock less than quantity → `OUT_OF_STOCK`. Store closes between quote and confirm → rejected. |
| BR-11 Status sequence | Every invalid transition is rejected (e.g. Accepted → Delivered). Only the assigned DP can update. |
| BR-13 AI grounding | The price spoken in the AI reply equals the server quote in 100% of eval cases. |
| BR-18 Admin-only credits | Customer, DP and manager calls to `wallet-credit` → `FORBIDDEN`. |
| BR-19 / BR-20 Manager permissions | Surge above the cap rejected. Coupon creation without permission rejected. |
| BR-21 Server-side pricing | A tampered client price is ignored. The server total is used. |
| BR-22 No secrets in app | CI scans the built APK and JS bundle for key patterns (Gemini, service-role key). |
| BR-08 AI logging | Every `ai-turn` call writes exactly one usage-log row with a cost estimate. |

## 4. AI Voice Assistant Evaluation (RECOMMENDATION)

### 4.1 Test set
- At least **100 utterances per language** (Hindi, English, Bhojpuri), plus code-mixed (Hinglish) samples.
- Recorded by native speakers from the target districts, with varied ages, background noise and phone microphones.
- Scenarios: simple order, quantity variants ("do jar", "2 bottle"), choice of address, missing information, change of mind, cancelling during the conversation, store closed, out of stock, out-of-scope requests.

### 4.2 Metrics (targets set under **PA-07**)

| Metric | Definition |
|---|---|
| Intent accuracy | Correct product and quantity understood |
| Task success rate | Conversation ends with the correct order placed (or correctly refused) |
| Confirmation compliance | Orders placed with explicit confirmation (must be **100%**) |
| Price fidelity | Spoken price = server quote (must be **100%**) |
| Avg turns to order | Number of conversation turns |
| Fallback rate | Share of sessions that switch to manual ordering |
| Latency p50/p75 | Customer stops speaking → reply audio starts |
| Cost per session / per order | From `ai_usage_logs` |

### 4.3 Regression
Any change to the prompt, tools or model re-runs the whole eval set. A drop in task success, or any confirmation or price failure, blocks release.

## 5. Device and Network Matrix (RECOMMENDATION)

| Dimension | Coverage |
|---|---|
| Android versions | Minimum supported version (to be decided in Level 1) up to the latest |
| Device tier | At least one 2 GB RAM low-end device, one mid-range device |
| Network | 4G, throttled 3G profile, offline → online transitions |
| Languages | UI checked in all three languages (text overflow, Devanagari rendering) |

## 6. Test Data
- Seed data for local and staging only: sample stores (Jamui), water products, test DPs, test managers.
- **No real customer data in non-production environments** (**PA-34**, **PA-37**).
- Test phone numbers configured in Supabase Auth so OTP can be automated in testing.

## 7. Entry and Exit Criteria per Level
- **Entry:** level scope approved; relevant PA items decided.
- **Exit:** all automated tests pass in CI; the business-rule tests for that level pass; no open critical or high bugs; documentation updated; **user approval received**.

## 8. Defect Severity

| Severity | Example |
|---|---|
| Critical | Money wrong, order assigned twice, data visible across stores, secret exposed |
| High | Cannot place an order, OTP failing, AI places an order without confirmation |
| Medium | Wrong translation, report total mismatch, UI broken on a supported device |
| Low | Cosmetic issues |
