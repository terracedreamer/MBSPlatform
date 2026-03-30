# Plan: 12 Enhancements — Session 9 Part 2

## Context
12 items from the pending items list, grouped into 5 execution batches for efficiency. RS256 (#16) and test coverage (#21) deferred to next session.

## Batch 1: Quick Backend Fixes (3 items)

### #17 — ADMIN_EMAILS Env Var
**File**: `server/middleware/auth.js` (124 lines)
- In `requireAdmin` middleware (line ~85), currently queries DB for `is_admin`
- Change: check `process.env.ADMIN_EMAILS` first (comma-separated), fall back to DB `is_admin`
- Also update `issueToken()` to set `isAdmin` from env var check
- **No frontend changes needed**

### #19 — Response Helpers Adoption
**Files**: All 10 route files in `server/routes/`
- Import `{ sendSuccess, sendError, sendNotFound, sendBadRequest, sendUnauthorized, sendForbidden }` from `../utils/responseHelpers`
- Replace `res.json({ success: true, ... })` → `sendSuccess(res, { ... })`
- Replace `res.status(4xx).json({ success: false, message })` → appropriate helper
- Do NOT touch webhook endpoints (Stripe/BTCPay return specific formats)
- Target files: auth.js, admin.js, entitlements.js, billing.js, referrals.js, friends.js, promotions.js, push.js, announcements.js, emailPreferences.js

### #13 — Activity Log for Subscription Events
**File**: `server/routes/entitlements.js` + `server/routes/billing.js`
- Import ActivityLog model
- Add logging after: subscribe-free (line ~180), trial downgrade, webhook subscription events
- Actions: `subscription_created`, `subscription_trial_started`, `subscription_trial_expired`, `subscription_cancelled`, `subscription_upgraded`
- Include metadata: product, type, category, amount, period

## Batch 2: Account Page Enhancements (3 items)

### #11 — Social Login Linking UI
**File**: `src/pages/AccountPage.jsx`
- Add "Connected Accounts" section after Profile card
- Show current auth methods from `user.auth_methods` array
- For each NOT connected: show "Connect" button
  - "Connect Nostr" → opens modal with npub input → calls `POST /api/auth/link-nostr`
  - "Connect LNURL" → opens modal with QR/polling → calls LNURL flow
  - "Connect Google" → redirect to Google OAuth → calls `POST /api/auth/google` with link flag
- For each connected: show green checkmark + "Connected" badge
- Backend routes already exist: `POST /api/auth/link-nostr`, `POST /api/auth/link-lnurl`

### #18 — GDPR Delete Confirmation
**File**: `src/pages/AccountPage.jsx` (lines 1431-1473)
- Current: two clicks (button → confirm button)
- New: add text input "Type DELETE to confirm" between the two steps
- Disable confirm button until input === "DELETE"
- Same pattern for per-app and per-category deletion modals

### #10 — Onboarding Flow Per Product
**File**: `src/components/OnboardingModal.jsx` (170 lines)
- Current: shows 3 category cards with "Explore Products" CTA
- Enhance: add step indicator (1/3)
  - Step 1: Welcome + category cards (current)
  - Step 2: "Pick your first product" — show top 3 recommended products with "Start Free Trial" buttons (calls subscribe-free API)
  - Step 3: Success — "You're all set! Open [product name]" with link to the app
- Keep "Maybe Later" dismiss on all steps

## Batch 3: Friends Enhancement (#12)

### #12 — Friends System Enhancement
**File**: `src/pages/AccountPage.jsx` (friends section) + `server/routes/friends.js`
- Current: basic list + invite code
- Enhance frontend:
  - Show friend avatars in a grid instead of plain list
  - Add "Remove Friend" button per friend
  - Show friend's subscribed products (if shared)
  - Copy invite link with toast feedback
- Enhance backend:
  - `DELETE /api/friends/:friendId` — remove friendship
  - `GET /api/friends/:friendId/shared-products` — find common subscriptions
- Add friend activity: "Your friend [name] just joined!" in notification bell

## Batch 4: Admin Analytics + Segmentation (3 items)

### #14 — User Segmentation (Full)
**Backend** (`server/routes/admin.js`):
- New endpoint: `GET /api/admin/users/segments` — returns counts per segment
- Segments: `new` (< 7 days), `trial` (has trial entitlement), `paid` (has stripe_subscription_id), `free` (active but no paid), `churned` (had entitlement, now expired/cancelled), `inactive` (no login > 30 days)
- Add `segment` filter param to existing `GET /api/admin/users` endpoint

**Frontend** (new `src/pages/admin/AdminUsers.jsx` after splitting):
- Segment filter chips at top of users tab (color-coded)
- Segment counts in chip badges
- User rows show segment badge

### #15 — Revenue Forecasting (Full Analytics Tab)
**Backend** (`server/routes/admin.js`):
- New endpoint: `GET /api/admin/analytics/revenue` — returns:
  - MRR (Monthly Recurring Revenue) current + last 6 months trend
  - ARR (Annual Recurring Revenue)
  - Revenue by product (pie chart data)
  - Revenue by payment method (Stripe vs Lightning)
  - Churn rate (cancelled / total active, last 30 days)
  - ARPU (Average Revenue Per User)
  - Growth rate (MRR month-over-month %)
  - Subscription growth (new vs churned per month, last 6 months)

**Frontend** (new `src/pages/admin/AdminAnalytics.jsx`):
- New 6th tab: "Analytics" with chart icon
- MRR trend line chart (CSS-based, no library needed)
- Revenue by product horizontal bars
- Churn rate card with trend indicator
- ARPU card
- Growth rate with up/down arrow

### #20 — AdminPage Splitting
**Current**: 2,067 lines in one file
**Split into**:
- `src/pages/AdminPage.jsx` — shell with tabs, auth check, socket (~200 lines)
- `src/pages/admin/AdminOverview.jsx` — stats, auth breakdown, analytics (~200 lines)
- `src/pages/admin/AdminUsers.jsx` — user list, search, detail panel (~350 lines)
- `src/pages/admin/AdminEntitlements.jsx` — entitlements list, grant/revoke (~200 lines)
- `src/pages/admin/AdminFlags.jsx` — feature flags CRUD (~150 lines)
- `src/pages/admin/AdminActivity.jsx` — activity log (~100 lines)
- `src/pages/admin/AdminAnalytics.jsx` — NEW revenue/forecasting tab (~250 lines)
- `src/components/admin/GrantModal.jsx` — grant entitlement modal (~150 lines)
- `src/components/admin/ConfirmDialog.jsx` — reusable confirm dialog (~50 lines)
- `src/components/admin/PushSendModal.jsx` — push notification modal (~150 lines)
- `src/components/admin/CreateFlagModal.jsx` — create flag modal (~100 lines)

**Approach**: Extract tab components first, then add Analytics tab. Keep all state in AdminPage shell, pass as props.

## Batch 5: Referrals + Promos (#4, #5)

### #4 — Referral Email Invites
**File**: `server/routes/referrals.js` (line 88 TODO)
- Import email service (SendGrid)
- Build HTML template: "You're invited by {name} to join MagicBusStudios"
- Include: referral link with utm params, CTA button, product highlights
- Check email preferences before sending
- Rate limit: max 10 invites per day per user

### #5 — Promo Code Checkout Flow
**Backend** (`server/routes/billing.js`):
- Add `promoCode` to checkout POST body validation
- Before creating Stripe session, call `POST /api/promotions/validate` internally
- If valid:
  - `percentage` type: create Stripe coupon + apply to session
  - `fixed` type: create Stripe coupon with amount_off
  - `trial_extension` type: extend trial_period_days on subscription
- After successful checkout: call `POST /api/promotions/redeem`
- Store promo_code in Transaction record

**Frontend** (`src/pages/BillingPage.jsx`):
- Add collapsible "Have a promo code?" input below plan cards
- Validate on blur/submit → show green checkmark + discount preview
- Pass promoCode to checkout API call

## Implementation Order
1. **Batch 1** (backend-only, quick) → #17, #19, #13
2. **Batch 4 part 1** (#20 AdminPage split) → enables #14, #15
3. **Batch 4 part 2** (#14 segmentation, #15 analytics) → new tab + endpoints
4. **Batch 2** (#11, #18, #10) → AccountPage + OnboardingModal
5. **Batch 3** (#12) → Friends enhancement
6. **Batch 5** (#4, #5) → Referrals + Promos

## Verification
1. `npm run build` after each batch — zero errors
2. Chrome extension screenshots of each modified page
3. Test admin dashboard new Analytics tab with real data
4. Test GDPR delete with "Type DELETE" confirmation
5. Test promo code flow end-to-end (if Stripe products exist)
