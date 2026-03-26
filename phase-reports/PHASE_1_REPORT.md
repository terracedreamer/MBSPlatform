# PHASE 1 COMPLETION REPORT

**Date**: 2026-03-26
**Branch**: `main`
**Commits**: `ad2e2af`, `675bc57`, `4ac0213`

---

## 1. What Was Built

### Backend — New Files (27)

| File | Purpose |
|---|---|
| `server/config/database.js` | MongoDB connection with graceful degradation |
| `server/config/products.js` | Product catalog — all 22 products with freeTier flags |
| `server/middleware/auth.js` | `issueToken`, `requireAuth`, `requireAdmin`, `validateRedirectUrl` |
| `server/models/User.js` | User model (email, google_id, stripe_customer_id, is_admin, etc.) |
| `server/models/Entitlement.js` | Entitlement model (product_pass, category_access, mbs_all_access) |
| `server/models/Transaction.js` | Transaction records (Stripe + BTCPay) |
| `server/models/ActiveSession.js` | Active session tracking |
| `server/models/NostrChallenge.js` | Nostr auth challenges (Phase 2 — model only) |
| `server/models/LnurlChallenge.js` | LNURL-Auth challenges (Phase 2 — model only) |
| `server/models/BtcpayInvoice.js` | BTCPay Lightning invoices |
| `server/models/Friend.js` | Friend pairs (sorted user ID pair) |
| `server/models/Invite.js` | Friend invite codes |
| `server/models/FeatureFlag.js` | Feature flags |
| `server/models/ConsentAuditLog.js` | GDPR consent audit (user_id is String, not ObjectId) |
| `server/models/DataRequest.js` | GDPR data requests (access, deletion, portability, rectification) |
| `server/models/PushSubscription.js` | Push notification subscriptions |
| `server/models/Announcement.js` | System announcements |
| `server/models/EmailPreference.js` | Email preferences per user |
| `server/models/Promotion.js` | Promo codes (Phase 2 — model only) |
| `server/models/Referral.js` | Referral tracking (Phase 2 — model only) |
| `server/models/ActivityLog.js` | Activity logging (login, logout, etc.) |
| `server/routes/forms.js` | Extracted existing form handler routes (contact, subscribe, waitlist) |
| `server/routes/auth.js` | Google SSO, /me, /logout, /account (GDPR cascade delete) |
| `server/routes/entitlements.js` | Entitlement checks with free tier logic and priority resolution |
| `server/routes/billing.js` | Stripe checkout, portal, webhook, transaction history |
| `server/routes/btcpay.js` | BTCPay Lightning checkout + webhook with HMAC signature verification |
| `server/routes/friends.js` | Friends list, create invite, accept invite |

### Backend — Modified Files (4)

| File | Change |
|---|---|
| `server/index.js` | Complete rewrite: MongoDB connect, defensive route loading, raw body bypass for Stripe, expanded CORS, dual health check paths |
| `server/package.json` | Added mongoose, jsonwebtoken, google-auth-library, stripe. Renamed to `mbs-platform` v2.0.0 |
| `server/package-lock.json` | Regenerated for all new dependencies |
| `server/.env.example` | Added all platform env vars (MongoDB, JWT, Google, Stripe, BTCPay) |

### Frontend — New Files (3)

| File | Purpose |
|---|---|
| `src/pages/AuthLoginPage.jsx` | Branded login page — reads `?brand=` and `?redirect=` params |
| `src/pages/BillingPage.jsx` | Plans grid, active subscriptions, Stripe portal, transaction history |
| `src/pages/AccountPage.jsx` | Profile, entitlements, friends, invite links, logout, GDPR account deletion |

### Frontend — Modified Files (1)

| File | Change |
|---|---|
| `src/App.jsx` | Added 3 platform routes with defensive `import().catch()` loading + FallbackError component |

---

## 2. What Changed from the Plan

| Spec Item | What Changed | Why |
|---|---|---|
| BTCPay env vars | Added `BTCPAY_WEBHOOK_SECRET` (4th var) | CWG app already uses webhook secret for HMAC verification — security consistency |
| BTCPay webhook | Added HMAC-SHA256 signature verification with `btcpay-sig` header | Not in spec but required for secure webhook processing |
| MongoDB connection | Made non-blocking (forms work without it) | Spec said "existing endpoints keep working" — this ensures a DB outage can't kill forms |
| Health check | Registered on both `/health` AND `/api/health` | Original form handler used `/health`, spec says `/api/health`. Both kept to avoid breaking Coolify health check |
| CORS config | Added `Authorization` header, `PUT`/`DELETE` methods, `credentials: true` | Original form handler only needed `Content-Type` + `POST`. Platform needs auth headers and DELETE |
| Stripe webhook | Added raw body middleware bypass in `server/index.js` | Stripe signature verification requires unparsed body — not mentioned in spec |
| ConsentAuditLog.user_id | Typed as `String` not `ObjectId` | GDPR deletion replaces user_id with SHA-256 hash — String avoids type conflicts |
| `VITE_GOOGLE_CLIENT_ID` | Added as frontend build arg | Not in spec but required for Google Sign-In button to render |
| Defensive route loading | Added both server-side (try/catch) and frontend (.catch) | Spec mentioned server-side only. Added frontend too for extra safety |
| Nostr/LNURL auth routes | Models created, routes deferred to Phase 2 | Per user instruction: "Defer to Phase 2: Nostr auth, LNURL auth" |
| Admin panel | Not built | Per user instruction: "Defer to Phase 2" |
| Email preferences routes | Model created, routes deferred to Phase 2 | Per user instruction |
| Promos/referrals routes | Models created, routes deferred to Phase 2 | Per user instruction |
| Platform env var logging | Added startup warnings for missing optional vars | Not in spec — added for deployment debugging |

---

## 3. Env Vars Required

### Backend (Coolify — MBS B container)

| Var | Required | Example | Notes |
|---|---|---|---|
| `SENDGRID_API_KEY` | Yes (crash) | `SG.xxxx` | Existing — form handler |
| `FROM_EMAIL` | Yes (crash) | `noreply@magicbusstudios.com` | Existing |
| `TO_EMAIL` | Yes (crash) | `support@magicbusstudios.com` | Existing |
| `PORT` | No | `3001` | Default: 3001 |
| `CORS_ORIGINS` | No | See full list below | Comma-separated |
| `MONGODB_URI` | No (warn) | `mongodb://user:pass@host:27017` | Platform features disabled without it |
| `DB_NAME` | No | `mbs_platform` | Default: `mbs_platform` |
| `JWT_SECRET` | No (warn) | 64-char hex string | Generate: `openssl rand -hex 32` |
| `JWT_EXPIRY` | No | `7d` | Default: `7d` |
| `GOOGLE_CLIENT_ID` | No (warn) | `xxx.apps.googleusercontent.com` | Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | No | `GOCSPX-xxx` | Google Cloud Console |
| `STRIPE_SECRET_KEY` | No (warn) | `sk_live_xxx` or `sk_test_xxx` | Stripe Dashboard |
| `STRIPE_WEBHOOK_SECRET` | No | `whsec_xxx` | Stripe webhook config |
| `BTCPAY_URL` | No (warn) | `https://btcpay.yourdomain.com` | BTCPay Server |
| `BTCPAY_API_KEY` | No | `xxx` | BTCPay API key |
| `BTCPAY_STORE_ID` | No | `xxx` | BTCPay store ID |
| `BTCPAY_WEBHOOK_SECRET` | No | `xxx` | BTCPay webhook HMAC secret |

**"Yes (crash)"** = Server won't start without it.
**"No (warn)"** = Logged at startup, platform features degraded.

### Frontend (Coolify — MBS F container, BUILD ARGS)

| Var | Required | Example |
|---|---|---|
| `VITE_API_URL` | Yes | `https://api.magicbusstudios.com` |
| `VITE_GOOGLE_CLIENT_ID` | No | `xxx.apps.googleusercontent.com` |

### CORS_ORIGINS — Full Value

```
https://magicbusstudios.com,https://www.magicbusstudios.com,https://innerlab.ai,https://www.innerlab.ai,https://conversationswithgod.ai,https://www.conversationswithgod.ai,https://yoga.magicbusstudios.com,https://brokenchain.magicbusstudios.com,https://mindhacker.magicbusstudios.com,https://triviaroast.magicbusstudios.com,https://whisperinghouse.magicbusstudios.com,https://fakeartist.magicbusstudios.com,https://wildlens.magicbusstudios.com,https://lazy-chef.magicbusstudios.com,https://tasktracker.magicbusstudios.com,https://tutor.magicbusstudios.com,https://smartcart.magicbusstudios.com,https://moviepicker.magicbusstudios.com
```

---

## 4. Database Collections Created

Database: `mbs_platform`

| Mongoose Model | Collection Name | Key Indexes |
|---|---|---|
| User | `users` | `email` (unique), `google_id` (unique sparse), `stripe_customer_id` (unique sparse) |
| Entitlement | `entitlements` | `user_id + product`, `user_id + category`, `stripe_subscription_id` |
| Transaction | `transactions` | `user_id` |
| ActiveSession | `activesessions` | `session_token` (unique), `user_id + created_at` |
| NostrChallenge | `nostrchallenges` | `challenge` (unique), `expires_at` (TTL) |
| LnurlChallenge | `lnurlchallenges` | `k1` (unique), `expires_at` (TTL) |
| BtcpayInvoice | `btcpayinvoices` | `invoice_id` (unique), `user_id` |
| Friend | `friends` | `users` (unique pair) |
| Invite | `invites` | `code` (unique), `from_user_id` |
| FeatureFlag | `featureflags` | `flag_name` (unique) |
| ConsentAuditLog | `consentauditlogs` | `user_id` (String, not ObjectId) |
| DataRequest | `datarequests` | `user_id` |
| PushSubscription | `pushsubscriptions` | `user_id` |
| Announcement | `announcements` | none |
| EmailPreference | `emailpreferences` | `user_id` (unique) |
| Promotion | `promotions` | `code` (unique) |
| Referral | `referrals` | `referrer_id` |
| ActivityLog | `activitylogs` | `user_id + created_at` |

---

## 5. API Routes

### Existing (unchanged)

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/health` | No | Health check (legacy path) |
| POST | `/api/contact` | No | Contact form → SendGrid |
| POST | `/api/subscribe` | No | Newsletter signup → SendGrid |
| POST | `/api/waitlist` | No | Waitlist form → SendGrid |

### New — Platform

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/api/health` | No | Health check with DB status |
| POST | `/api/auth/google` | No | Google SSO — create/find user, return JWT |
| GET | `/api/auth/me` | Yes | Current user + entitlements summary |
| POST | `/api/auth/logout` | Yes | Invalidate all sessions |
| DELETE | `/api/auth/account` | Yes | GDPR cascade delete (Stripe cancel, mbs_platform wipe, inner_lab wipe, consent anonymize) |
| GET | `/api/entitlements` | Yes | All active entitlements for current user |
| GET | `/api/entitlements/:product` | Yes | Check access for specific product → `{ hasAccess, reason }` |
| GET | `/api/entitlements/category/:cat` | Yes | All products in category with access status |
| POST | `/api/billing/checkout` | Yes | Create Stripe checkout session |
| POST | `/api/billing/portal` | Yes | Get Stripe customer portal URL |
| GET | `/api/billing/history` | Yes | Transaction history (last 50) |
| POST | `/api/billing/webhook` | No (Stripe sig) | Stripe webhook — raw body, signature verified |
| POST | `/api/billing/btcpay/checkout` | Yes | Create BTCPay Lightning invoice |
| POST | `/api/billing/btcpay/webhook` | No (HMAC sig) | BTCPay webhook — HMAC-SHA256 signature verified |
| GET | `/api/friends` | Yes | List friends |
| POST | `/api/friends/invite` | Yes | Create 7-day invite link |
| POST | `/api/friends/accept` | Yes | Accept invite code, create friendship |

**Auth = Yes** means `Authorization: Bearer <JWT>` header required.

---

## 6. Frontend Routes

### Existing (unchanged — 13 routes)

`/`, `/inner-lab`, `/waitlist`, `/subscribe`, `/about`, `/contact`, `/studio-works`, `/arcade`, `/privacy`, `/terms`, `/cookies`, `/refund-policy`, `/acceptable-use`

### New (3 routes)

| Path | Component | Auth-Gated | Notes |
|---|---|---|---|
| `/auth/login` | `AuthLoginPage` | No | Reads `?brand=mbs\|innerlab` and `?redirect=` |
| `/billing` | `BillingPage` | Yes (redirects to login) | Plans, subscriptions, history |
| `/account` | `AccountPage` | Yes (redirects to login) | Profile, friends, logout, delete |

Auth gating is done in-component: checks `localStorage.getItem("mbs_token")`, redirects to `/auth/login?redirect=...&brand=mbs` if missing.

---

## 7. JWT Payload

### Issued by `issueToken()` in `server/middleware/auth.js`:

```json
{
  "userId": "MongoDB ObjectId as string",
  "email": "user@example.com",
  "name": "User Name",
  "avatar": "https://lh3.googleusercontent.com/..." or null,
  "isAdmin": false,
  "iat": 1711432800,
  "exp": 1712037600
}
```

### Read by `requireAuth()` middleware:

Populates `req.user` with the decoded payload. Downstream routes access:
- `req.user.userId` — canonical user ID across all products
- `req.user.email`
- `req.user.name`
- `req.user.isAdmin` — UI hint only; admin routes re-check DB

### Token format:

```
Authorization: Bearer <JWT>
```

### Token storage (frontend):

```javascript
localStorage.getItem("mbs_token")   // JWT string
localStorage.getItem("mbs_user")    // JSON: { id, email, name, avatar }
```

---

## 8. Entitlement Response Format

### `GET /api/entitlements/:product`

```json
{
  "success": true,
  "hasAccess": true,
  "reason": "free_tier"
}
```

### Valid `reason` values:

| reason | Meaning | Priority |
|---|---|---|
| `mbs_all_access` | User has MBS All Access pass | 1 (highest) |
| `category_access` | User has access to entire category | 2 |
| `product_pass` | User has pass for this specific product | 3 |
| `free_tier` | Product allows free access, user has no paid entitlement | 4 (lowest) |
| `no_subscription` | No access, product has no free tier | N/A (hasAccess: false) |

### Priority resolution:

When multiple entitlements overlap, the **broadest** reason is returned:
`mbs_all_access > category_access > product_pass > free_tier`

### Error responses:

```json
// Product not found
{ "success": false, "message": "Product not found" }  // 404

// Not authenticated
{ "success": false, "message": "Authentication required" }  // 401
```

---

## 9. Assumptions Made

1. **Google Identity Services (not redirect flow)**: Used the newer GIS one-tap/button library (`google.accounts.id`) instead of the older OAuth redirect flow. The ID token is verified server-side via `google-auth-library`. This means the frontend loads the Google script from `accounts.google.com/gsi/client`.

2. **Token passed in URL on redirect**: After login, the JWT is appended as `?token=<JWT>` to the redirect URL. Product frontends must extract it, store it, and remove it from the URL with `history.replaceState`.

3. **localStorage for token storage**: Frontend stores token in `localStorage` (not cookies). This aligns with the CWG app pattern.

4. **Friend pairs sorted by ObjectId**: The Friend model stores a sorted pair of user ObjectIds to prevent duplicate friendships (A→B and B→A would both be `[A, B]`).

5. **Invite links expire in 7 days**: Not specified in platform docs. Chose 7 days as a reasonable default.

6. **BTCPay entitlements are 30-day passes**: BTCPay doesn't support recurring billing. Each Lightning payment creates a 30-day product pass. User must manually repurchase.

7. **Transaction amounts in cents (Stripe) or sats (BTCPay)**: The `amount` field stores cents for USD transactions and sats for Lightning transactions. The `currency` field distinguishes them (`usd` vs `sats`).

8. **Billing page plan prices are placeholder**: The plans grid shows $0 / $9.99 / $14.99 but doesn't link to real Stripe price IDs yet. The "Upgrade" button shows a toast. Real price IDs need to be created in Stripe Dashboard and wired in.

9. **GDPR cascade deletes from inner_lab database**: The account deletion endpoint connects to the `inner_lab` database (same MongoDB URI, different DB name) and deletes all documents where `user_id` matches. This handles both string and ObjectId user_id fields.

10. **Category values are lowercase no-separator slugs**: `innerlab`, `arcade`, `studioworks` — not `inner_lab` or `Inner Lab`.

---

## 10. Known Gaps

1. **No Stripe price IDs configured**: The billing page shows plan cards but can't actually start checkout until price IDs are created in Stripe Dashboard and wired into the frontend.

2. **No Nostr auth route**: Model exists (`NostrChallenge`), route deferred to Phase 2.

3. **No LNURL-Auth route**: Model exists (`LnurlChallenge`), route deferred to Phase 2.

4. **No admin panel**: Deferred to Phase 2. `requireAdmin` middleware is built and ready.

5. **No email preference routes**: Model exists, routes deferred to Phase 2.

6. **No promo/referral routes**: Models exist, routes deferred to Phase 2.

7. **No token refresh endpoint**: Phase 1 uses short-lived JWTs (7 days) with no refresh. When expired, user re-authenticates via Google SSO (typically auto-approves). Phase 2 should add `/api/auth/refresh`.

8. **No rate limiting on platform routes**: Form routes have rate limiting (10/15min). Platform routes (auth, entitlements, billing) don't have rate limiting yet. Should be added before production traffic ramps up.

9. **Billing page plan pricing is hardcoded in frontend**: Should eventually be fetched from the product catalog or Stripe API.

10. **No email notifications**: Account deletion, subscription changes, etc. don't send confirmation emails. Phase 2 should add transactional emails via SendGrid.

11. **Login page discoverability — no "Sign In" in MBS site navigation.** The login page exists at `/auth/login?brand=mbs` but is invisible to users browsing the MBS marketing site. The platform design assumes product apps redirect users to this URL, but several questions need the orchestrating agent's input:
    - Should the MBS nav bar include a "Sign In" link? If so, where does it go after login — `/account`?
    - Do Arcade games and Studio Works apps need auth at all? Currently they're free static sites with no user accounts.
    - For Inner Lab modules (CWG, FlowState), the flow is: user visits product → product redirects to `magicbusstudios.com/auth/login?brand=innerlab&redirect=<product-url>` → user logs in → redirected back with token. But this flow isn't implemented on the product side yet (that's Phase 3/4).
    - The login page supports `?brand=mbs` and `?brand=innerlab` branding but no brand-specific variants for individual products (e.g., CWG-branded login). Is that needed?
    - **Owner's vision (confirmed):** Two login experiences — `?brand=mbs` for Studio Works/Arcade (with a "Looking for Inner Lab?" link), and `?brand=innerlab` as a dedicated Inner Lab login for all Inner Lab modules. Both share the same backend/user table/JWT. The orchestrating agent should design the exact UX for both.

12. **Lightning/BTCPay payment option missing from billing UI.** The backend routes exist (`POST /api/billing/btcpay/checkout`, `POST /api/billing/btcpay/webhook`) and BTCPay env vars are configured, but the BillingPage.jsx only shows Stripe checkout buttons. There is no "Pay with Lightning" option visible to users. CWG's settings page (Preferences → Lightning) shows this as a precedent — the platform billing page should offer the same.

13. **Billing page is a skeleton — needs full product catalog design.** The current BillingPage shows 3 generic plan cards (Product Pass $9.99, Category Access $19.99, All Access $29.99) but does not reflect the real product catalog. The orchestrating agent needs to design:
    - How 22 products across 3 categories (Inner Lab, Arcade, Studio Works) are presented
    - Per-product subscriptions vs category bundles vs all-access — what combinations are valid?
    - Which products are free forever (Arcade? Studio Works?) vs freemium (CWG, FlowState) vs paid-only?
    - How existing CWG Stripe subscriptions appear here — do they show as "Product Pass: CWG" or does the user need to re-subscribe through the platform?
    - Lightning as an equal payment method alongside Stripe (not hidden in a sub-page)
    - Transaction history showing both Stripe and BTCPay payments
    - The Stripe Price IDs that need to be created in the Stripe Dashboard to match whatever pricing structure is decided

14. **Stripe Price IDs not assigned to any phase.** The billing backend is ready to accept Price IDs and create checkout sessions, but no actual Stripe products/prices exist yet. This requires:
    - A pricing decision from the orchestrating agent (what costs what)
    - Creating products in Stripe Dashboard (prefixed `[MBS]` per global convention)
    - Wiring Price IDs into the billing checkout flow
    - The orchestrating agent should decide which phase owns this: Phase 1 addendum, Phase 2 (Inner Lab), or a dedicated billing phase.

---

## 11. Gotchas for Downstream Phases

### For ALL downstream products (CWG, FlowState, Arcade, Studio Works):

1. **JWT header format is `Authorization: Bearer <token>`** — not cookies, not custom headers.

2. **`req.user.userId` is the canonical field** — it's a string, not an ObjectId. If your backend needs an ObjectId, cast it: `new mongoose.Types.ObjectId(req.user.userId)`.

3. **Entitlement API returns `reason` field** — products should use this to determine behavior:
   - `free_tier` → enforce free tier limits (messages/day, minutes/day, etc.)
   - `product_pass` / `category_access` / `mbs_all_access` → full access
   - `no_subscription` → redirect to billing

4. **Cache entitlement responses for 5 minutes** — the platform spec says products SHOULD cache. Key: `userId:productSlug`, TTL: 5 min, in-memory.

5. **After billing success, redirect with `?refresh=true`** — tells the product to bust its entitlement cache for that user.

6. **CORS_ORIGINS must include your product's frontend domain** — or auth/entitlement API calls will fail silently.

7. **Open redirect protection is active** — the `?redirect=` URL in the login flow is validated against CORS_ORIGINS. If your domain isn't in the list, users will be redirected to `magicbusstudios.com` instead of your product.

### For Phase 2 — Inner Lab Middleware:

8. **`inner_lab` database is touched by GDPR deletion** — the MBS Platform connects to `inner_lab` and deletes all documents matching `user_id`. Make sure all Inner Lab collections use `user_id` consistently (not `userId` or `user`).

9. **ConsentAuditLog.user_id is a String** — not ObjectId. If Phase 2 writes to this collection, use `.toString()` on the user ID.

### For Phase 3 — CWG Migration:

10. **User model has `google_id` field** — when migrating CWG users, match on `google_id` or `email` to find/create mbs_platform users. The auth route already handles linking: if a Google user logs in and their email matches an existing account, the `google_id` is linked automatically.

11. **CRITICAL — Duplicate users will exist across databases.** Phase 1 creates users in `mbs_platform` on first Google SSO login. Users who already have accounts in `conversations_with_god` (or other product DBs) will get a **separate, new platform account** with a different `_id`. When CWG integrates with the platform, you MUST handle this:
    - **Option A (recommended) — Merge on first platform login**: When a CWG user logs into the platform for the first time, match by `email` or `google_id` against the CWG database. Link the existing CWG user data to the new platform `user_id`. CWG keeps its own DB but adds a `platform_user_id` field to its user records.
    - **Option B — Batch migration script**: Before CWG goes live on the platform, run a one-time script that creates platform accounts for all existing CWG users (matched by email), maps old CWG `_id` values to new platform `user_id` values, and updates CWG records with `platform_user_id`.
    - Either way, the CWG agent must decide how to handle entitlement history, conversation data, and subscription records that reference the old CWG user ID.
    - The platform owner (Abhinav) already has a platform account (email: `1984.abhinav@gmail.com`, platform ID: `69c53401fe8f1763b9046ae5`) that was created during Phase 1 testing. This account exists ONLY in `mbs_platform` — it has no link to any CWG data yet.

### For Phase 4 — FlowState Migration:

12. **Same duplicate-user issue as CWG** — match on `google_id` or `email` during migration. See item 11 above for approach options.

---

## 12. Testing Commands

### Health check (no auth needed):
```bash
curl https://api.magicbusstudios.com/api/health
# Expected: { "success": true, "service": "mbs-platform", "database": true, "timestamp": "..." }
```

### Form handler still works (no auth needed):
```bash
curl -X POST https://api.magicbusstudios.com/api/contact \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test","message":"Hello"}'
# Expected: { "success": true, "message": "Message sent successfully." }
```

### Google SSO (requires real Google credential):
```bash
# 1. Get a Google ID token from the login page UI
# 2. POST it:
curl -X POST https://api.magicbusstudios.com/api/auth/google \
  -H "Content-Type: application/json" \
  -d '{"credential":"<GOOGLE_ID_TOKEN>","redirect":"https://magicbusstudios.com"}'
# Expected: { "success": true, "token": "<JWT>", "redirect": "https://magicbusstudios.com", "user": {...} }
```

### Get current user:
```bash
curl https://api.magicbusstudios.com/api/auth/me \
  -H "Authorization: Bearer <JWT>"
# Expected: { "success": true, "user": {...}, "entitlements": [...] }
```

### Check entitlement:
```bash
curl https://api.magicbusstudios.com/api/entitlements/cwg \
  -H "Authorization: Bearer <JWT>"
# Expected: { "success": true, "hasAccess": true, "reason": "free_tier" }
```

### Check category:
```bash
curl https://api.magicbusstudios.com/api/entitlements/category/innerlab \
  -H "Authorization: Bearer <JWT>"
# Expected: { "success": true, "category": "innerlab", "products": [{slug, name, hasAccess, reason}, ...] }
```

### Logout:
```bash
curl -X POST https://api.magicbusstudios.com/api/auth/logout \
  -H "Authorization: Bearer <JWT>"
# Expected: { "success": true, "message": "Logged out successfully" }
```

### Create friend invite:
```bash
curl -X POST https://api.magicbusstudios.com/api/friends/invite \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{}'
# Expected: { "success": true, "invite": { "code": "...", "expires_at": "...", "link": "..." } }
```

### Billing portal (requires Stripe customer):
```bash
curl -X POST https://api.magicbusstudios.com/api/billing/portal \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{"returnUrl":"https://magicbusstudios.com/billing"}'
# Expected: { "success": true, "url": "https://billing.stripe.com/..." }
```

### Frontend pages (browser):
```
https://magicbusstudios.com/auth/login?brand=mbs
https://magicbusstudios.com/auth/login?brand=innerlab&redirect=https://conversationswithgod.ai
https://magicbusstudios.com/billing
https://magicbusstudios.com/account
```

### Verify marketing site unaffected:
```
https://magicbusstudios.com/             (homepage)
https://magicbusstudios.com/contact      (form works)
https://magicbusstudios.com/waitlist     (form works)
https://magicbusstudios.com/about        (static page)
```

---

## 13. Live Test Results (2026-03-26)

All tests performed against production deployment at `api.magicbusstudios.com`:

| # | Test | Method | Result | Notes |
|---|---|---|---|---|
| 1 | Health check | `GET /api/health` | ✅ PASS | `database: true` after MONGO_URL fix |
| 2 | Contact form | `POST /api/contact` | ✅ PASS | "Message sent successfully" |
| 3 | Auth middleware (no token) | `GET /api/auth/me` | ✅ PASS | Returns 401 "Authentication required" |
| 4 | Entitlements (no token) | `GET /api/entitlements` | ✅ PASS | Returns 401 "Authentication required" |
| 5 | Friends (no token) | `GET /api/friends` | ✅ PASS | Returns 401 "Authentication required" |
| 6 | Billing (no token) | `POST /api/billing/checkout` | ✅ PASS | Returns 401 "Authentication required" |
| 7 | Google SSO login | Browser flow | ✅ PASS | User created, JWT issued, redirected with token |
| 8 | Auth/me (with token) | `GET /api/auth/me` | ✅ PASS | Returns full profile + empty entitlements |
| 9 | Entitlements/cwg (with token) | `GET /api/entitlements/cwg` | ✅ PASS | `hasAccess: true, reason: "free_tier"` |
| 10 | Friends list (with token) | `GET /api/friends` | ✅ PASS | Returns empty friends list |
| 11 | Logout | `POST /api/auth/logout` | ✅ PASS | "Logged out successfully" |
| 12 | Account page (browser) | `/account` | ✅ PASS | Shows profile, subscriptions, friends, sign out |
| 13 | Homepage loads | `GET /` | ✅ PASS | HTML served |
| 14 | About page loads | `GET /about` | ✅ PASS | HTML served |
| 15 | Contact page loads | `GET /contact` | ✅ PASS | HTML served |

**First user created:** Abhinav Gupta (`1984.abhinav@gmail.com`), platform ID: `69c53401fe8f1763b9046ae5`

**Env var fix during testing:** Backend was using `MONGODB_URI` but Coolify had `MONGO_URL` (matching CWG convention). Code was updated to read `MONGO_URL` to stay consistent across all MBS apps.
