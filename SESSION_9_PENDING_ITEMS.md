# SESSION 9 — Pending Items & Enhancement Roadmap
**Date**: March 30, 2026
**Platform Status**: 85% complete — all infrastructure deployed, payments pending

---

## 🔴 BLOCKING — Owner Action Required

### 1. Create Stripe Products (Test Mode)
**Impact**: Billing page checkout buttons won't work without price IDs.

| Product | Monthly | Annual | Env Var (Monthly) | Env Var (Annual) |
|---------|---------|--------|-------------------|------------------|
| FlowState | $5 | $45 | `STRIPE_YOGA_MONTHLY_PRICE_ID` | `STRIPE_YOGA_ANNUAL_PRICE_ID` |
| CWG | $15 | $135 | `STRIPE_CWG_MONTHLY_PRICE_ID` | `STRIPE_CWG_ANNUAL_PRICE_ID` |
| Studio Works All Access | $10 | $90 | `STRIPE_SW_MONTHLY_PRICE_ID` | `STRIPE_SW_ANNUAL_PRICE_ID` |
| Arcade All Access | $10 | $90 | `STRIPE_ARCADE_MONTHLY_PRICE_ID` | `STRIPE_ARCADE_ANNUAL_PRICE_ID` |
| Inner Lab All Access | $20 | $180 | `STRIPE_IL_MONTHLY_PRICE_ID` | `STRIPE_IL_ANNUAL_PRICE_ID` |
| MBS All Access | $30 | $270 | `STRIPE_MBS_MONTHLY_PRICE_ID` | `STRIPE_MBS_ANNUAL_PRICE_ID` |

**Steps**:
1. Go to `dashboard.stripe.com/test/products`
2. Create each product with monthly + annual recurring prices (USD)
3. Copy 12 price IDs to Coolify MBS B env vars
4. Redeploy MBS B

**Note**: Existing `STRIPE_MONTHLY_PRICE_ID` and `STRIPE_ANNUAL_PRICE_ID` (old CWG prices) still work as fallbacks but should be replaced with the new `STRIPE_CWG_*` vars.

### 2. Regenerate BTCPay API Key
**Impact**: Lightning payments return 403 — insufficient permissions.

**Steps**:
1. Log into BTCPay admin
2. Account → API Keys → Create new key with full store permissions
3. Update `BTCPAY_API_KEY` in Coolify MBS B
4. Redeploy MBS B
5. Test: `POST /api/billing/btcpay/checkout` with a test amount

---

## 🟡 HIGH PRIORITY — Agent Work

### 3. Real-Time Entitlement Sync After Stripe Checkout
**Impact**: After paying, user redirects to `/billing?success=true` but entitlements don't refresh. Manual page reload required.

**Fix**:
- In BillingPage.jsx, on `?success=true` param, fetch `/api/entitlements/my-subscriptions`
- Update AuthContext with fresh entitlements
- Show toast: "Subscription active!"

**Files**: `src/pages/BillingPage.jsx`, `src/contexts/AuthContext.jsx`

### 4. Referral Email Invites
**Impact**: Referral system exists (routes, codes, tracking) but email invite is a TODO stub.

**Fix**:
- `POST /api/referrals/invite` should call SendGrid
- Template: "You're invited by [name] to join MagicBusStudios"
- Include referral link + CTA to `/auth/signup?ref=[code]`

**File**: `server/routes/referrals.js` line 88

### 5. Promo Code Checkout Flow
**Impact**: Promo code CRUD exists in admin but no way to apply codes during checkout.

**Fix**:
- Add promo code input field to BillingPage checkout flow
- Validate code before creating Stripe session
- Apply discount to line_items or metadata
- Record `promo_code_applied` in Transaction

**Files**: `src/pages/BillingPage.jsx`, `server/routes/billing.js`, `server/routes/promos.js`

### 6. CWG: Merge test → main
**Impact**: CWG entitlement enforcement (`f9c38ab`) is on `test` branch. Not on production.

**Decision needed**: When to promote. CWG stays on `test` indefinitely per Session 8 decision.

---

## 🟢 ENHANCEMENTS — Medium Priority

### Payments & Billing

#### 7. Stripe Subscription Management Portal
Test the "Manage Subscription" button once Stripe products are created. Verify upgrade/downgrade/cancel flows work end-to-end.

#### 8. Invoice Generation (PDF)
Transactions are recorded but no downloadable invoices. Needed for VAT compliance if going international.

#### 9. BTCPay Recurring Payment Reminders
Lightning is a 30-day one-time pass with manual renewal. Add email reminders when the 30 days are about to expire.

### User Experience

#### 10. Onboarding Flow Per Product
Current onboarding modal says "Welcome to MagicBusStudios" but doesn't guide users to their first product. Add step-by-step: Pick a product → Start trial → Open app.

#### 11. Social Login Linking UI
Backend supports linking Nostr/LNURL to an email account. No UI on Account page to manage connected auth methods.

#### 12. Friends System Enhancement
Friends exist at platform level but no products use them. Could add friend activity feeds, shared achievements, or multiplayer invites in Arcade games.

### Admin & Analytics

#### 13. Activity Log for Subscription Events
ActivityLog tracks signups/logins but not subscription changes (created, upgraded, cancelled, expired). Add to Activity tab.

**Files**: `server/routes/entitlements.js`, `server/routes/billing.js`

#### 14. User Segmentation
Admin shows flat user list. Add segments: trial users, paid users, churned users, power users (most active).

#### 15. Revenue Forecasting
Analytics show current month revenue but no MRR trend, projected growth, or churn rate.

---

## 🔵 PRE-LAUNCH — Security & Architecture

### 16. JWT Upgrade to RS256
**Priority**: Critical pre-launch.

Currently HS256 symmetric — if `JWT_SECRET` leaks, attacker can forge tokens for all 13 apps. RS256 means only the platform signs, child apps only verify with a public key.

**Estimate**: 2-3 days across MBS + all child apps.

### 17. ADMIN_EMAILS Env Var
Replace `is_admin: true` DB field with `ADMIN_EMAILS` environment variable in Coolify. Easier to manage, no DB access needed.

**Current admins**: `terracedreamer@gmail.com`, `1984.abhinav@gmail.com`

### 18. GDPR Delete Confirmation
Currently just requires auth token. Should add "type DELETE to confirm" step or password re-entry before nuking all user data.

---

## 🟣 CODE QUALITY — Low Priority

### 19. Response Helpers Adoption
`sendSuccess()`, `sendError()`, `sendNotFound()`, `sendBadRequest()`, `sendUnauthorized()`, `sendForbidden()` are created at `server/utils/responseHelpers.js` but not used in routes yet. Gradual migration when touching any route.

### 20. AdminPage Splitting
At 2,058 lines it's the largest single file. Split into `AdminOverview.jsx`, `AdminUsers.jsx`, `AdminEntitlements.jsx`, `AdminFlags.jsx`, `AdminActivity.jsx`.

### 21. Test Coverage Expansion
22 auth tests exist (Jest + supertest). No frontend tests, no billing tests, no entitlement tests. LazyChef is the gold standard.

---

## Session 9 Completed Work (for reference)

| Commit | Description |
|--------|-------------|
| `65d0612` | Fix opacity on ProductPickerPage + BillingPage |
| `d460c36` | Update pricing — 6 products with 25% annual discount |
| `f7418d8` | Fix CWG annual price to $135/yr |
| `0e542c5` | Modern pricing page with monthly/annual toggle + IndividualPlansPage |
| `a67c631` | BillingPage opacity animations under ProtectedRoute |
| `ff2a4c9` | Premium UI polish across all 5 platform pages |
| `f66205e` | Replace SectionHeading with inline headings on protected pages |
| `f9aa92d` | Price AnimatePresence initial={false} |
| `6ba2e57` | Remaining opacity fixes on IndividualPlansPage + AdminPage |

### Key Learnings This Session
- **SectionHeading component** uses `whileInView` / `useInView` which never fires under ProtectedRoute (content starts hidden during loading overlay). Use inline headings with the same gradient styling on protected pages.
- **AnimatePresence** needs `initial={false}` prop to skip mount animation on protected pages. Without it, keyed children (price toggles, tab content) start invisible.
- **Framer Motion `initial={{ opacity: 0 }}`** is the root cause of all "blank page" bugs under ProtectedRoute. Always use `initial={false}` on page-level motion.divs in protected routes.
