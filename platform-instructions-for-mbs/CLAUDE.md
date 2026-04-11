# MBS Platform — Layer 1: SSO, Billing & Entitlements

## What This Project Becomes
This project is `MBS/` — the magicbusstudios.com website. It currently serves as a **marketing site with a SendGrid form handler**. It is being upgraded to ALSO serve as the **MBS Platform** — the centralized SSO, billing, and entitlements layer for ALL 22+ Magic Bus Studios products.

**After the upgrade:**
- `magicbusstudios.com` (public) — marketing pages (existing, unchanged)
- `magicbusstudios.com/auth/login` — branded login page (NEW)
- `magicbusstudios.com/auth/signup` — branded signup page (NEW)
- `magicbusstudios.com/subscribe` — checkout, subscription management (NEW, renamed from /billing)
- `magicbusstudios.com/account` — profile, settings, connected auth methods (NEW)
- `magicbusstudios.com/admin` — user management, analytics (NEW, Phase 2+)
- `magicbusstudios.com/api/...` — all platform API endpoints (NEW)

The existing form handler endpoints continue to work alongside the new platform APIs.

## What This Project Does NOT Do
- Does NOT handle Inner Lab shared data (consciousness, memories, check-ins) — that's the Inner Lab Middleware at innerlab.ai
- Does NOT handle product-specific logic — each product (CWG, FlowState, etc.) keeps its own backend
- Does NOT replace the existing marketing site — it adds to it

## Stack
- **Frontend**: React (Vite) — existing marketing pages + new auth/billing/account pages
- **Backend**: Express — existing form handler + new platform API routes
- **Database**: MongoDB — `DB_NAME: mbs_platform` (NEW database, separate from form handler)
- **Auth**: Google SSO + Nostr + LNURL-Auth (all passwordless)
- **Payments**: Stripe + BTCPay (Lightning)
- **Email**: SendGrid (existing setup, extended for platform transactional emails)
- **Deployment**: Coolify — same 2 containers (frontend + backend) already deployed

## Architecture Context

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 1: THIS PROJECT (magicbusstudios.com)                  │
│  Database: mbs_platform                                       │
│                                                               │
│  Frontend: marketing + login + billing + account + admin      │
│  Backend:  form handler + SSO + entitlements + Stripe/BTCPay  │
│            + friends + invites + push + feature flags + GDPR  │
│                                                               │
│  ALL 22 products redirect here for login.                     │
│  ALL 22 products call this API to check entitlements.         │
└──────────┬──────────────────────────┬────────────────────────┘
           │                          │
┌──────────▼──────────┐       ┌───────▼──────────────────────┐
│  Layer 2: Inner Lab │       │  Layer 3: Standalone         │
│  (innerlab.ai)      │       │  Products                    │
│  DB: inner_lab      │       │  Arcade, Studio Works        │
│                      │       │  Each has own DB             │
│  Shared il_* data   │       │  SSO only via Layer 1        │
│  + dashboard        │       └──────────────────────────────┘
│  + module backends  │
│  (CWG, FlowState)  │
└──────────────────────┘
```

## Auth: Four Methods
- **Google SSO** — primary, fastest path (existing across MBS)
- **Email/Password** — traditional signup + login with email verification, password reset, and optional 2FA/TOTP
- **Nostr** — challenge → signature verification (experimental, from CWG)
- **LNURL-Auth** — Lightning wallet login (experimental, from CWG)

Login and signup are the same page. Google SSO button on top, then "or" divider, then Nostr + LNURL buttons, then "or use email" divider with email/password form. "Don't have an account? Create one" link goes to the signup form (name + email + password + age confirmation + terms).

## JWT Payload Specification

When the platform issues a JWT, it MUST contain exactly these fields:

```javascript
{
  userId: String,         // MongoDB ObjectId as string — the _id from mbs_platform.users
  email: String|null,     // user's email address (null for Nostr/LNURL-only users)
  name: String,           // user's display name
  avatar: String|null,    // user's avatar URL (from Google profile picture, etc.)
  isAdmin: Boolean,       // whether user has admin privileges
  iat: Number,            // issued-at timestamp (auto-set by JWT library)
  exp: Number             // expiration timestamp (controlled by JWT_EXPIRY env var, default 7d)
}
```

**All downstream services** (Inner Lab Middleware, CWG, FlowState, Arcade, Studio Works, new modules) decode this token and access `req.user.userId` to identify the user. This is the canonical user ID across the entire ecosystem.

**Do NOT include** entitlements, subscription status, or other data that changes frequently — those are checked via API calls, not baked into the token.

## Branded Login
The login page at `magicbusstudios.com/auth/login` changes branding based on the `?brand=` parameter:

| Origin | Brand | Login page shows |
|---|---|---|
| Any Inner Lab module | `brand=innerlab` | Inner Lab logo, IL colors |
| Any Arcade game | `brand=mbs` | MBS logo, MBS colors |
| Any Studio Works tool | `brand=mbs` | MBS logo, MBS colors |
| innerlab.ai | `brand=innerlab` | Inner Lab logo |
| magicbusstudios.com | `brand=mbs` | MBS logo |

## Login Flow
```
User clicks "Login" on any product (e.g., conversationswithgod.ai)
  → Redirect to: https://magicbusstudios.com/auth/login?redirect=https://conversationswithgod.ai&brand=innerlab
  → User sees branded login page
  → Authenticates via Google SSO / Email-Password / Nostr / LNURL
  → Platform creates/finds user, issues JWT
  → Platform VALIDATES redirect domain against ALLOWED_REDIRECT_DOMAINS (see below)
  → Redirect back: https://{origin}?token={JWT}
  → Product frontend extracts token from URL, stores it, removes token from URL (history.replaceState)
  → Product frontend sends JWT in Authorization header on API calls
  → Product backend calls GET /api/entitlements/{product} to verify access
  → 401 = redirect to login | 403 = redirect to billing
```

### Open Redirect Protection (SECURITY — REQUIRED)
The `?redirect=` parameter on the login page MUST be validated against an allowlist to prevent token theft:
- Maintain an `ALLOWED_REDIRECT_DOMAINS` list (can be derived from CORS_ORIGINS or a separate env var)
- Before redirecting back with `?token={JWT}`, check that the redirect domain is in the allowlist
- If the domain is NOT in the allowlist, redirect to `magicbusstudios.com` instead (safe default)
- The `?redirect=` value MUST include `https://` protocol — reject bare domains

### Token-in-URL Handling
The JWT is passed as a URL query parameter during the redirect. The product frontend MUST:
1. Extract the token from the URL immediately on page load
2. Store it (localStorage or cookie)
3. Remove the token from the URL using `history.replaceState(null, '', window.location.pathname)`
4. This prevents the token from appearing in browser history, server logs, or referrer headers

## Payments: Dual System
- **Stripe** — subscriptions (monthly/yearly) + one-time purchases. Stripe prefixes by category: `[IL]`, `[Arcade]`, `[SW]`, `[MBS]`
- **BTCPay / Lightning** — alternative payment in sats

Both handled here. No per-product payment integration.

## Database: `mbs_platform`

### Core Models
- **User**: email, name, avatar, password_hash?, google_id?, nostr_npub?, lnurl_linking_key?, auth_methods [String] (e.g. ["email","google","nostr","lnurl"]), email_verified (Boolean, default: true for Google, false for email signup), email_verification_token?, email_verification_expires?, password_reset_token?, password_reset_expires?, totp_enabled (Boolean, default: false), totp_secret?, totp_backup_codes [String]?, preferred_language, preferences {}, consent_preferences {}, is_admin, stripe_customer_id?, referral_code?, referred_by?, referral_count, created_at, updated_at, last_login
- **Entitlement**: user_id, category (innerlab|arcade|studioworks), type (product_pass|category_access|mbs_all_access), product (slug|null), status (active|expired|cancelled|trial), stripe_subscription_id?, purchased_at, expires_at?, trial_ends_at?
- **Transaction**: user_id, type, category, product?, amount (cents), currency, stripe_payment_id, description, promo_code?, created_at
- **Promotion**: code (unique), type, value, applies_to, category?, product?, max_uses, current_uses, valid_from, valid_until, created_by
- **Referral**: referrer_id, referral_code, referred_email, referred_user_id?, status, reward_type, reward_value, created_at
- **EmailPreference**: user_id, marketing_emails, product_updates, billing_alerts, promotional_offers, unsubscribe_token
- **Friend**: users (sorted pair), source_product, created_at
- **Invite**: code, from_user_id, product, expires_at, created_at
- **ActiveSession**: user_id, session_token, ip_address (hashed), user_agent, created_at, last_active
- **PushSubscription**: user_id, subscription (endpoint, keys), user_agent, subscribed_at
- **FeatureFlag**: flag_name, enabled, category, description, target_plans, target_users
- **ConsentAuditLog**: user_id, change_type, old_value, new_value, timestamp (GDPR — cannot delete)
- **DataRequest**: user_id, request_type (access|deletion|portability|rectification), status, created_at
- **NostrChallenge**: challenge, user_id?, expires_at
- **LnurlChallenge**: k1, status, user_id?, expires_at
- **BtcpayInvoice**: user_id, invoice_id, amount_sats, status, product?, created_at
- **ActivityLog**: user_id, product, category, action, metadata {}, created_at
- **Announcement**: title, message, type, target, active, valid_from, valid_until

## Entitlement Types
- `product_pass` — one specific product
- `category_access` — all products in a category (Inner Lab / Arcade / Studio Works)
- `mbs_all_access` — everything

## Entitlement Check Response Specification

`GET /api/entitlements/:product` returns `{ success: true, hasAccess: Boolean, reason: String }`.

Valid `reason` values:

| reason | Meaning | When returned |
|--------|---------|---------------|
| `free_tier` | User has no paid entitlement but product allows free access | Product has `freeTier: true` in catalog config |
| `product_pass` | User has a paid pass for this specific product | Active Entitlement with type=product_pass for this slug |
| `category_access` | User has access to the entire category (e.g., all Inner Lab) | Active Entitlement with type=category_access for matching category |
| `mbs_all_access` | User has access to everything | Active Entitlement with type=mbs_all_access |
| `no_subscription` | User has no access and product has no free tier | No matching Entitlement and product has `freeTier: false` |

### Free Tier Logic
The product catalog config determines which products have free tiers. Example:
```javascript
// config/products.js — freeTierLimits REMOVED (Session 31)
{ slug: "cwg", freeTier: true }
{ slug: "brokenchain", freeTier: true }
{ slug: "wildlens", freeTier: false }
```

**Three-state entitlement model (Session 31):**
- Users with `type: "free_tier"` entitlement → `{ hasAccess: true, isPremium: false, reason: "free_tier" }`
- Users with paid entitlement → `{ hasAccess: true, isPremium: true, reason: "product_pass" | "category_access" | "mbs_all_access" }`
- Users with no entitlement and `freeTier: false` → `{ hasAccess: false, reason: "no_subscription" }`

Each product's OWN backend enforces free tier limits (message caps, time caps) based on the `isPremium` field. MBS Platform passes NO limits — only `hasAccess` + `isPremium`.

## API Endpoints

**Phase 1 (MVP)**
- POST /api/auth/google — Google SSO, create/find user, return JWT
- POST /api/auth/signup — email/password registration (name, email, password, age_confirmed, terms_accepted). Hash password with bcrypt. Send verification email. Return JWT.
- POST /api/auth/login — email/password login. Verify bcrypt hash. If totp_enabled, return { requires2FA: true } instead of JWT — frontend must then call /api/auth/2fa/verify.
- POST /api/auth/forgot-password — send password reset email via SendGrid (token with 1hr expiry)
- POST /api/auth/reset-password — verify token, set new password_hash
- GET /api/auth/verify-email/:token — verify email address, set email_verified: true
- POST /api/auth/resend-verification — resend verification email (rate limited: 3/hr)
- POST /api/auth/2fa/setup — (requires JWT) generate TOTP secret + QR code + 10 backup codes. Store secret but don't enable yet.
- POST /api/auth/2fa/verify-setup — (requires JWT) verify TOTP code to confirm setup, then set totp_enabled: true
- POST /api/auth/2fa/verify — verify TOTP code or backup code during login. On success, issue JWT. Backup codes are one-time-use (delete after use).
- POST /api/auth/2fa/disable — (requires JWT + password) disable 2FA, clear totp_secret and backup codes
- GET /api/auth/2fa/status — (requires JWT) returns { enabled: Boolean }
- POST /api/auth/nostr — Nostr authentication
- POST /api/auth/lnurl — LNURL-Auth
- GET /api/auth/me — current user + entitlements summary
- POST /api/auth/logout — invalidate session
- DELETE /api/auth/account — delete account + all data (GDPR)
- GET /api/entitlements — all entitlements for current user
- GET /api/entitlements/:product — check access → `{ hasAccess, reason }`
- GET /api/entitlements/category/:cat — all accessible products in category
- POST /api/billing/checkout — create Stripe checkout
- POST /api/billing/portal — Stripe customer portal URL
- POST /api/billing/webhook — Stripe webhook
- POST /api/billing/btcpay/checkout — BTCPay Lightning checkout
- POST /api/billing/btcpay/webhook — BTCPay webhook
- GET /api/billing/history — transaction history
- GET /api/friends — list friends
- POST /api/friends/invite — create invite
- POST /api/friends/accept — accept invite
- GET /api/health — health check

**Existing endpoints (keep working):**
- POST /api/contact — contact form (SendGrid)
- POST /api/subscribe — newsletter subscribe (SendGrid)
- POST /api/waitlist — waitlist signup (SendGrid)

**Phase 2+** — email preferences, admin panel, promos, referrals

## Product Catalog
Store in config, NOT hardcoded:

### Inner Lab (12 modules)
`cwg`, `flowstate`, `bonds`, `lifemap`, `starmap`, `astrocompass`, `arcana`, `archetypes`, `dreamlens`, `rituals`, `innerquest`, `nexus`

### The Arcade (5 games)
`brokenchain`, `mindhacker`, `triviaroast`, `whisperinghouse`, `fakeartist`

### Studio Works (6 tools)
`wildlens`, `lazychef`, `tasktracker`, `tutor`, `smartcart`, `moviepicker`

## Project Structure (After Upgrade)
```
MBS/
├── src/                    # React frontend
│   ├── pages/              # Existing marketing pages + NEW auth/billing/account pages
│   │   ├── HomePage.jsx         # existing
│   │   ├── AboutPage.jsx        # existing
│   │   ├── AuthLoginPage.jsx    # NEW — branded login (Google SSO + email/password + Nostr + LNURL + 2FA)
│   │   ├── AuthSignupPage.jsx   # NEW — branded signup (Google SSO + email/password form)
│   │   ├── BillingPage.jsx      # NEW — checkout, subscription management
│   │   ├── AccountPage.jsx      # NEW — profile, settings, auth methods
│   │   └── AdminPage.jsx        # NEW (Phase 2) — user management
│   ├── components/
│   └── ...
├── server/
│   ├── index.js            # Express entry — existing form handler + NEW platform routes
│   ├── routes/
│   │   ├── auth.js         # NEW — Google SSO, Nostr, LNURL, JWT
│   │   ├── entitlements.js # NEW — check access, grant, revoke
│   │   ├── billing.js      # NEW — Stripe checkout, portal, webhooks
│   │   ├── btcpay.js       # NEW — BTCPay Lightning
│   │   ├── friends.js      # NEW — friends, invites
│   │   ├── forms.js        # EXISTING — contact, subscribe, waitlist (SendGrid)
│   │   └── admin.js        # NEW (Phase 2)
│   ├── models/             # NEW — Mongoose models
│   ├── middleware/          # NEW — requireAuth, requireAdmin, rateLimiter
│   ├── services/           # NEW — stripe, btcpay, sendgrid, entitlement-checker
│   ├── config/             # NEW — product catalog
│   └── scripts/            # NEW — migration scripts
├── Dockerfile
├── nginx.conf
└── package.json
```

## Environment Variables (Add to Existing)
**NEW (add to Coolify):**
- MONGO_URL (may already exist for other purposes)
- DB_NAME (`mbs_platform`)
- JWT_SECRET (64-char)
- JWT_EXPIRY (`7d`)
- GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- BTCPAY_URL, BTCPAY_API_KEY, BTCPAY_STORE_ID
- CORS_ORIGINS (all 22+ product frontend URLs, comma-separated)

**EXISTING (keep):**
- SENDGRID_API_KEY, FROM_EMAIL, TO_EMAIL
- PORT

## Migration Scripts
Two migration scripts to build (run once, from this backend):

### CWG Migration (56 collections)
- Read from `conversations_with_god` database
- Write user identity to `mbs_platform.users`
- Write CWG product data to `inner_lab` DB with `cwg_*` prefix
- Write shared data to `inner_lab` DB with `il_*` prefix
- Old database stays as backup
- See `CWG/MBS_DATABASE_MIGRATION_PLAN.md` in the CWG project (or `MBSPlatform/platform-instructions-for-cwg/PLATFORM_MIGRATION.md` for the full migration doc)

### FlowState Migration (7 collections)
- Read from `yogaghost` database
- Same split pattern: identity → mbs_platform, product → yoga_*, shared → il_*
- See `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md` in the FlowState project (or `MBSPlatform/platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md` for the full migration doc)

## Architectural Notes for Downstream Products

### Deep Link Preservation
The `?redirect=` parameter in the login flow MUST accept full URLs including paths:
- Example: `?redirect=https://conversationswithgod.ai/chat/123`
- The platform redirects back to the FULL URL: `https://conversationswithgod.ai/chat/123?token={JWT}`
- Product frontends must handle the token extraction without losing the current path

### Token Refresh Strategy (Phase 1 — Simple)
Phase 1 uses short-lived JWTs with no refresh token. When the JWT expires (7 days):
- Product frontends detect 401 responses
- Redirect to MBS Platform login (same branded flow)
- Google SSO typically auto-approves (no re-consent needed) so the user experience is a brief redirect, not a full re-login
- **Phase 2+**: Add a `/api/auth/refresh` endpoint with refresh tokens for silent re-authentication

### Entitlement API Resilience
The entitlement check endpoint (`GET /api/entitlements/:product`) is called by all 22 products. To prevent it becoming a single point of failure:
- Products SHOULD cache entitlement responses on the backend for 5-15 minutes
- If the platform is unreachable, products SHOULD allow access based on the last cached response (graceful degradation)
- The platform MUST respond fast (< 100ms) — this is a simple DB lookup with an index on `user_id`
- Rate limiting: per-product, not per-user (each product backend makes one call per user request)

## GDPR Account Deletion Cascade

`DELETE /api/auth/account` triggers a full cascade across databases:

1. **Cancel Stripe** — Cancel all active subscriptions via Stripe API
2. **Delete from `mbs_platform`** — User, Entitlements, Transactions, ActiveSessions, PushSubscriptions, Referrals, Friends, Invites, EmailPreferences, DataRequests, NostrChallenges, LnurlChallenges, BtcpayInvoices, ActivityLog
3. **Keep ConsentAuditLog** — Legal requirement. Anonymize `user_id` to a SHA-256 hash
4. **Delete from `inner_lab`** — All il_* documents where `user_id` matches. The platform connects to the `inner_lab` database directly for this (same MONGO_URL, different DB_NAME)
5. **Delete module data** — All cwg_*, yoga_*, and other module-prefixed documents where `user_id` matches. Same direct DB connection approach.
6. **Confirmation** — Return success. GDPR requires completion within 30 days; this should complete in seconds since it's all database deletes.

The platform handles the ENTIRE cascade — individual products do NOT need deletion endpoints.

## BTCPay → Entitlement Flow

When BTCPay webhook reports invoice paid (`status: "settled"`):
1. Update BtcpayInvoice `status` to `"settled"`
2. Create Entitlement: `type: "product_pass"`, `status: "active"`, `product: invoice.product`, `expires_at: now + product catalog duration` (e.g., 30 days for monthly equivalent)
3. Create Transaction record
4. BTCPay does NOT support automatic recurring billing — user must manually repurchase when entitlement expires. Surface expiry warnings via email (Phase 2).

## Entitlement Priority (Overlapping Access)

When multiple entitlements grant access, return the BROADEST reason:
`mbs_all_access > category_access > product_pass > free_tier`

The product only cares about `hasAccess` — the `reason` is for UI display and analytics.

## Entitlement Category Values

Category values are lowercase slugs with no separators: `innerlab`, `arcade`, `studioworks`. NOT `inner_lab`, `Inner Lab`, or `inner-lab`. Validate on write.

## Entitlement Cache (Products)

Products SHOULD cache entitlement responses on their backend:
- **Cache key**: `userId:productSlug`
- **TTL**: 5 minutes (balances freshness vs load)
- **Storage**: In-memory (Map or node-cache for Node, dict or cachetools for Python)
- **Invalidation**: After billing success, redirect the user with `?refresh=true` to tell the product to bust its cache for that user
- **Graceful degradation**: If the platform is unreachable, allow access based on last cached response

## Admin Endpoint Security

Admin endpoints (`/api/admin/*`) MUST verify `is_admin` from the **database**, not the JWT. The JWT's `isAdmin` field is for UI hints only (showing admin menu). Any actual admin action must re-check `mbs_platform.users.is_admin` on every request. Reason: if a user is demoted, their JWT still has `isAdmin: true` for up to 7 days.

## JWT Security Model

All services share a single JWT_SECRET. This is a deliberate tradeoff for simplicity at MVP scale.

**Risk**: If any service's JWT_SECRET is exposed, all services are compromised.

**Mitigations**:
1. Rotate JWT_SECRET periodically (requires coordinated redeployment of all services)
2. Monitor for suspicious admin access patterns
3. **RS256 asymmetric signing is NOW ACTIVE** — the platform signs JWTs with `JWT_PRIVATE_KEY` (RS256), products verify with `JWT_PUBLIC_KEY`. Products can VERIFY but not FORGE tokens. HS256 fallback remains for backward compatibility.
   - **Node.js apps**: `jsonwebtoken` supports RS256 natively — no extra packages needed.
   - **Python apps**: MUST use `PyJWT[crypto]` (not plain `PyJWT`) — the `[crypto]` extra installs the `cryptography` package required for RSA operations. Without it, RS256 fails with `InvalidAlgorithmError: Algorithm not supported`.

## ALLOWED_REDIRECT_DOMAINS

Derive programmatically from CORS_ORIGINS at startup. Parse the comma-separated list, extract hostnames, use as the redirect allowlist. Do NOT maintain a separate list — they must stay in sync.

## Deployment Checklist

1. Set ALL new env vars in Coolify FIRST (JWT_SECRET, STRIPE keys, BTCPAY, CORS_ORIGINS, DB_NAME, etc.)
2. Verify env vars in Coolify UI (names only, not values)
3. Ensure VITE_BACKEND_URL is set as a BUILD ARG for the frontend container
4. Deploy backend container first — verify `/api/health` returns 200
5. Verify existing form endpoints still work (POST /api/contact)
6. Deploy frontend container
7. Test: marketing pages load, login page loads, full login flow works
8. **Defensive route loading**: Wrap new route imports in try/catch so a bug in auth.js doesn't crash the entire server and take down the existing marketing site.

## CORS_ORIGINS — Complete Domain List

Must include all product frontend domains:
```
https://magicbusstudios.com,https://www.magicbusstudios.com,https://innerlab.ai,https://www.innerlab.ai,https://conversationswithgod.ai,https://www.conversationswithgod.ai,https://yoga.magicbusstudios.com,https://brokenchain.magicbusstudios.com,https://mindhacker.magicbusstudios.com,https://triviaroast.magicbusstudios.com,https://whisperinghouse.magicbusstudios.com,https://fakeartist.magicbusstudios.com,https://wildlens.magicbusstudios.com,https://lazy-chef.magicbusstudios.com,https://tasktracker.magicbusstudios.com,https://tutor.magicbusstudios.com,https://smartcart.magicbusstudios.com,https://moviepicker.magicbusstudios.com
```
Add new module/product domains as they launch. Subdomains of magicbusstudios.com do not need www variants.

## stripe_customer_id Placement

`stripe_customer_id` lives on the **User** model only. It represents the Stripe customer (one per user). The Entitlement model has `stripe_subscription_id` (one per subscription). Do not duplicate the customer ID on Entitlement — join via `user_id` if needed.

## What NOT to Do
- Do NOT remove existing marketing pages or form handler — they keep working
- Do NOT modify CWG or FlowState backends until platform is built and tested
- Do NOT build Stripe into individual products — all billing through this platform
- Do NOT hardcode product slugs
- Do NOT create a separate `platform.magicbusstudios.com` domain — everything lives on `magicbusstudios.com`

---

## Completion Report (REQUIRED)

When you finish building, generate a file called `PHASE_1_REPORT.md` in the project root. The orchestrator will fetch it from here. The report must contain:

1. **What was built** — Every file created or modified, grouped by backend/frontend
2. **What changed from the plan** — Any deviations from this document. Did you rename anything? Skip anything? Add anything not in the spec? Why?
3. **Env vars required** — Complete list of every env var the code expects, with example values
4. **Database collections** — Exact name of every Mongoose model and its MongoDB collection name
5. **API routes** — Full table (method, path, auth required?, description)
6. **Frontend routes** — Every React route added (path, component, auth-gated?)
7. **JWT payload** — Exact fields your code puts in the token and reads from the token
8. **Entitlement response format** — Exact JSON shape returned by GET /api/entitlements/:product
9. **Assumptions made** — Anything you had to decide that wasn't explicitly in the spec
10. **Known gaps** — Anything intentionally deferred or couldn't complete
11. **Gotchas for downstream phases** — Anything Phase 2 (Inner Lab Middleware), Phase 3 (CWG migration), Phase 4 (FlowState migration), or Phase 5 (standalone products) agents need to know that differs from or adds to their platform-instructions docs
12. **Testing commands** — Exact curl commands or steps to verify each feature works

This report is critical — the orchestrator session (MBSPlatform repo) uses it to update instructions for all downstream phases before they start.

---

## Phase 1 Addendum (Added by Orchestrator after Phase 1 Report Review — 2026-03-26)

The core Phase 1 build is complete and deployed. Complete ALL items below before moving to Phase 2. These maximize what the platform can do before we start building the Inner Lab middleware.

### 1. Login Button in MBS Nav
- Add a "Login" button to the marketing site navigation bar, next to the existing items (Inner Lab, Studio Works, The Arcade, About)
- Link to `/auth/login?brand=mbs`
- Simple nav button — match existing nav styling

### 2. Rate Limiting on Platform Routes
- The existing form routes already have rate limiting (10/15min)
- Platform routes do NOT have rate limiting yet — add it now:
  - Auth routes (`/api/auth/*`): 10 requests per 15 minutes
  - Entitlement checks (`/api/entitlements/*`): 60 requests per 15 minutes
  - Billing routes (`/api/billing/*`): 5 requests per 15 minutes
  - Friends routes (`/api/friends/*`): 20 requests per 15 minutes

### 3. Lightning Payment Option in Billing UI
- Backend routes exist (`POST /api/billing/btcpay/checkout`, `POST /api/billing/btcpay/webhook`)
- BillingPage.jsx currently only shows Stripe checkout buttons
- Add a "Pay with Lightning" option alongside Stripe on the billing page
- BTCPay entitlements are 30-day non-recurring passes — make this clear in the UI ("30-day access, no auto-renewal")

### 4. Nostr Authentication
- Model already exists (`NostrChallenge`)
- Build the route: `POST /api/auth/nostr`
- Flow: client sends `{ npub, signature, challenge }` → server verifies signature against the challenge → creates/finds user → issues JWT
- Challenge endpoint: `GET /api/auth/nostr/challenge` → returns `{ challenge }` (random string, stored in NostrChallenge with TTL)
- On user creation, add `"nostr"` to `auth_methods` array, set `nostr_npub` field
- If a user with the same email already exists (from Google SSO), link the Nostr identity to the existing account

### 5. LNURL-Auth
- Model already exists (`LnurlChallenge`)
- Build the route: `POST /api/auth/lnurl`
- Flow: server generates LNURL-auth URL with k1 challenge → user's Lightning wallet signs it → callback verifies → creates/finds user → issues JWT
- `GET /api/auth/lnurl/challenge` → returns `{ lnurl }` (encoded LNURL-auth string)
- `GET /api/auth/lnurl/callback?k1=...&sig=...&key=...` → verify signature, create/find user by `lnurl_linking_key`, issue JWT
- On user creation, add `"lnurl"` to `auth_methods` array, set `lnurl_linking_key` field
- If a user with the same linking key already exists, log them in

### 6. Auth Method Linking
- Users who logged in via Google should be able to link a Nostr identity and/or LNURL identity to their existing account
- `POST /api/auth/link-nostr` (requires JWT) — verify Nostr signature, add `nostr_npub` to existing user
- `POST /api/auth/link-lnurl` (requires JWT) — verify LNURL signature, add `lnurl_linking_key` to existing user
- If the Nostr npub or LNURL key is already linked to a DIFFERENT account, return error

### 7. Email Preferences
- Model already exists (`EmailPreference`)
- Build routes:
  - `GET /api/email-preferences` (requires JWT) — get current preferences
  - `PUT /api/email-preferences` (requires JWT) — update preferences
  - `GET /api/email-preferences/unsubscribe?token=...` (no auth) — one-click unsubscribe via token
- Fields: marketing_emails, product_updates, billing_alerts, promotional_offers
- Generate `unsubscribe_token` on user creation, include in all outgoing emails
- Create default EmailPreference record when a new user signs up (all opted in)

### 8. Transactional Emails (SendGrid)
SendGrid is already integrated for form handling. Add transactional emails:
- **Welcome email** — sent on first signup (any auth method). Subject: "Welcome to Magic Bus Studios"
- **Purchase confirmation** — sent after successful Stripe or BTCPay payment. Include product name, amount, receipt link
- **Subscription cancelled** — sent when Stripe subscription is cancelled
- **Subscription expiring** — sent 3 days before a subscription renewal date (Stripe) or BTCPay 30-day pass expiry
- **Payment failed** — sent when Stripe payment fails (retry instructions)
- **Account deleted** — sent after GDPR account deletion confirming data was removed
- All emails: include unsubscribe link, respect EmailPreference settings, use `noreply@magicbusstudios.com` as sender
- HTML email templates — clean, simple, branded. No complex layouts.

### 9. Basic Admin Panel
- `requireAdmin` middleware already exists
- Build routes:
  - `GET /api/admin/users` — paginated user list with search (by email, name)
  - `GET /api/admin/users/:id` — full user detail with entitlements and transactions
  - `POST /api/admin/entitlements/grant` — manually grant an entitlement `{ userId, type, product?, category? }`
  - `DELETE /api/admin/entitlements/:id` — revoke an entitlement
  - `GET /api/admin/stats` — basic counts (total users, active entitlements, revenue)
- Frontend: add `/admin` route (auth-gated + admin check). Simple table UI — nothing fancy.
- Admin check: verify `is_admin: true` from database, NOT from JWT (JWT isAdmin is a UI hint only)

### 10. Promo Code Routes
- Model already exists (`Promotion`)
- Build routes:
  - `POST /api/promos/validate` (requires JWT) — check if a code is valid `{ code }` → `{ valid, discount, applies_to }`
  - `POST /api/admin/promos` (admin only) — create a promo code
  - `GET /api/admin/promos` (admin only) — list all promo codes
  - `DELETE /api/admin/promos/:id` (admin only) — deactivate a promo code
- Integrate with billing checkout: if promo code is provided, apply discount before creating Stripe session
- Track `current_uses` against `max_uses`

### 11. Referral Routes
- Model already exists (`Referral`)
- Build routes:
  - `GET /api/referrals` (requires JWT) — get user's referral code and stats
  - `POST /api/referrals/apply` (requires JWT) — apply a referral code `{ code }` (only on first purchase)
- On user creation, auto-generate a unique `referral_code` on the User model
- When a referred user makes their first purchase, increment `referral_count` on the referrer and create a Referral record
- Reward logic: deferred (just track referrals for now, rewards come later)

### 12. Token Refresh
- Currently tokens expire in 7 days with no refresh — user must re-authenticate via Google/Nostr/LNURL
- Add `POST /api/auth/refresh` (requires valid JWT that hasn't expired) — issues a new JWT with fresh expiry
- Frontend should call this silently when token is within 1 day of expiry
- If token is already expired, redirect to login

### 13. Structured Billing Page with Real CWG Pricing

Replace the placeholder billing page with a structured layout organized by category.

**Page Layout:**
- Top level: three category tabs/sections — **Inner Lab**, **The Arcade**, **Studio Works**
- Plus an **MBS All Access** section at the top (access to everything)

**When user clicks "Inner Lab":**
- Shows available modules: **Conversations With God** (Live), **FlowState** (Live), plus Coming Soon modules
- Shows **Inner Lab All Access** bundle option
- When user clicks on **Conversations With God**, shows:

**CWG Pricing (from existing CWG Stripe integration):**

| Plan | Stripe Price | Lightning (sats) |
|------|-------------|-----------------|
| CWG Premium Monthly | $9.99/mo | 21,000 sats/mo |
| CWG Premium Yearly | $79.99/yr | 126,000 sats/yr |

CWG also has pay-per-use Lightning options (these are product-specific, not platform subscriptions):
- Single Chat: 1,000 sats (~$0.50)
- Extended Session (24 hours): 5,000 sats (~$2.50)

**Bundle Pricing:**

| Plan | Price | Scope |
|------|-------|-------|
| Inner Lab All Access Monthly | $19.99/mo | All Inner Lab modules |
| Inner Lab All Access Yearly | $159.99/yr | All Inner Lab modules |
| MBS All Access Monthly | $29.99/mo | Everything (Inner Lab + Arcade + Studio Works) |
| MBS All Access Yearly | $249.99/yr | Everything |

**Stripe Products to Create:**
The CWG Stripe products already exist (env vars `STRIPE_MONTHLY_PRICE_ID` and `STRIPE_ANNUAL_PRICE_ID` in CWG's Coolify config). For the platform billing page, create these NEW Stripe products (prefixed `[MBS]` per convention):
- `[MBS] CWG Premium Monthly` — $9.99/mo (or reuse existing CWG price IDs)
- `[MBS] CWG Premium Yearly` — $79.99/yr (or reuse existing CWG price IDs)
- `[MBS] Inner Lab All Access Monthly` — $19.99/mo
- `[MBS] Inner Lab All Access Yearly` — $159.99/yr
- `[MBS] All Access Monthly` — $29.99/mo
- `[MBS] All Access Yearly` — $249.99/yr

**NOTE:** The actual Stripe Price IDs will need to be created in Stripe Dashboard manually and set as env vars. The billing page should read price IDs from env vars or the product catalog config, NOT hardcode them.

**For Arcade and Studio Works tabs:**
- Show the products as "Free" or "Coming Soon" for now
- These don't have paid tiers yet — just list them so the structure is in place

**Lightning payment** should be shown as an equal option alongside Stripe for every paid plan (not hidden in a sub-menu). For Lightning plans, BTCPay creates a 30-day pass (no auto-renewal) — make this clear in the UI.

### 14. Email/Password Authentication + Signup Page

Add traditional email/password auth as a 4th method alongside Google SSO, Nostr, and LNURL.

**Signup Page (`AuthSignupPage.jsx` at `/auth/signup`):**
- Google SSO button on top (fastest path) — "Sign up with Google"
- "or" divider
- Signup form: Name, Email, Password (8+ chars), Confirm Password
- Age confirmation checkbox (13+, 16+ in EU/EEA/UK)
- Terms of Service + Privacy Policy acceptance checkbox (required)
- "Create Account" button
- "Already have an account? Sign in" link → `/auth/login`
- Branded same as login page (`?brand=` param applies)
- `?redirect=` param preserved — after signup, redirect to the product that sent them

**Login Page (`AuthLoginPage.jsx` — update existing):**
- Google SSO button on top — "Sign in with Google"
- "or" divider
- Nostr + LNURL buttons
- "or use email" divider
- Email + Password form
- "Sign In" button
- "Don't have an account? Create one" link → `/auth/signup` (preserve brand + redirect params)
- "Forgot password?" link → inline or modal reset flow
- If backend returns `{ requires2FA: true }` after password verify → show TOTP code input (6-digit field + "Use backup code" link)

**Backend Routes (add to `routes/auth.js`):**
- `POST /api/auth/signup` — `{ name, email, password, age_confirmed, terms_accepted }` → validate email format, check password 8+ chars, check email not already taken, hash with bcrypt (12 rounds), create user with `auth_methods: ["email"]`, `email_verified: false`, generate verification token, send verification email via SendGrid, return JWT
- `POST /api/auth/login` — `{ email, password }` → find user by email, verify bcrypt hash, check `email_verified` (if false, return error "Please verify your email"), if `totp_enabled` return `{ success: true, requires2FA: true, tempToken: "..." }` (short-lived token for 2FA step), otherwise return JWT
- `POST /api/auth/forgot-password` — `{ email }` → generate reset token (crypto.randomBytes), set `password_reset_token` + `password_reset_expires` (1 hour), send reset email. Always return success (even if email not found — prevents email enumeration).
- `POST /api/auth/reset-password` — `{ token, password }` → find user by valid non-expired token, hash new password, clear reset token fields, return success
- `GET /api/auth/verify-email/:token` — find user by `email_verification_token` where not expired, set `email_verified: true`, clear token fields. Redirect to login page with `?verified=true` message.
- `POST /api/auth/resend-verification` — `{ email }` → generate new token, send new email. Rate limit: 3 per hour.

**Auth Method Linking (cross-method):**
When a user signs up with email/password and later clicks "Sign in with Google" using the SAME email:
- Find existing user by email → add `google_id` to existing account, add "google" to `auth_methods` array
- Do NOT create a duplicate user
- Same logic applies in reverse: Google SSO user who later sets a password via account settings

**Email templates (SendGrid — add to existing transactional email system):**
- Verification email: "Verify your email address" — link to `GET /api/auth/verify-email/:token`
- Password reset email: "Reset your password" — link to `magicbusstudios.com/auth/reset-password?token=:token`
- Both branded (respect `?brand=` if available, default to MBS branding)

**Frontend pages to add/update:**
- `AuthSignupPage.jsx` — NEW signup page
- `AuthLoginPage.jsx` — UPDATE to add email/password form + 2FA input
- `AuthResetPasswordPage.jsx` — NEW page for setting new password (reads `?token=` from URL)

**Env vars (no new ones needed — uses existing SENDGRID_API_KEY and JWT_SECRET)**

### 15. Two-Factor Authentication (2FA/TOTP) with Backup Codes

Add optional TOTP-based 2FA that users enable from their Account page.

**Backend Routes (add to `routes/auth.js`):**
- `POST /api/auth/2fa/setup` — (requires JWT) Generate TOTP secret using `otplib` or `speakeasy` npm package. Generate QR code URL (otpauth:// URI). Generate 10 backup codes (crypto.randomBytes, 8 chars each, hashed with bcrypt before storing). Return `{ secret, qrCodeUrl, backupCodes }`. Store `totp_secret` on user but do NOT set `totp_enabled: true` yet.
- `POST /api/auth/2fa/verify-setup` — (requires JWT) `{ code }` → verify TOTP code against stored secret. If valid, set `totp_enabled: true`. This confirms the user has their authenticator app set up correctly.
- `POST /api/auth/2fa/verify` — `{ tempToken, code }` → verify the short-lived temp token from login, then verify TOTP code OR backup code. If backup code, delete it from the array (one-time use). Return JWT on success.
- `POST /api/auth/2fa/disable` — (requires JWT) `{ password }` → verify password, then set `totp_enabled: false`, clear `totp_secret` and `totp_backup_codes`. Password required to prevent unauthorized disable.
- `GET /api/auth/2fa/status` — (requires JWT) → `{ enabled: Boolean }`
- `POST /api/auth/2fa/regenerate-backup-codes` — (requires JWT + password) → generate new set of 10 backup codes, replace old ones

**Frontend (Account page addition):**
- Add "Two-Factor Authentication" section to AccountPage.jsx
- If not enabled: "Enable 2FA" button → shows QR code + secret + backup codes → verify setup
- If enabled: "Disable 2FA" button (requires password confirmation) + "Regenerate Backup Codes" button
- Display backup codes ONCE during setup (user must save them — cannot be shown again)

**Login flow with 2FA:**
1. User enters email + password → `POST /api/auth/login`
2. Backend verifies password, sees `totp_enabled: true`
3. Returns `{ success: true, requires2FA: true, tempToken: "..." }` (tempToken is a short-lived JWT, 5 min expiry, with just `{ userId, purpose: "2fa" }`)
4. Frontend shows 6-digit code input field + "Use backup code" toggle
5. User enters code → `POST /api/auth/2fa/verify` with `{ tempToken, code }`
6. Backend verifies tempToken + TOTP code → returns full JWT
7. Note: Google SSO, Nostr, and LNURL bypass 2FA (those methods are inherently strong auth)

**npm packages needed:** `otplib` (or `speakeasy`) for TOTP, `qrcode` for QR code generation

### Deferred (genuinely cannot build now)
- FlowState pricing (no paid tier defined yet)
- Arcade/Studio Works pricing (free for now)
- Pay-per-use Lightning for CWG (this is CWG-specific, handled in Phase 3 migration)

### When done
Generate an updated report as `PHASE_1_ADDENDUM_REPORT.md` in the project root. Include:
1. What was built (every file, grouped by backend/frontend)
2. New API routes added (full table)
3. New frontend routes added
4. Env vars (any new ones needed?)
5. Gotchas for downstream phases (anything Phase 2+ agents need to know)
6. Testing commands for every new feature
