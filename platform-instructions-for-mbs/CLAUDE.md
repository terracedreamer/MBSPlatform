# MBS Platform — Layer 1: SSO, Billing & Entitlements

## What This Project Becomes
This project is `MBS/` — the magicbusstudios.com website. It currently serves as a **marketing site with a SendGrid form handler**. It is being upgraded to ALSO serve as the **MBS Platform** — the centralized SSO, billing, and entitlements layer for ALL 22+ Magic Bus Studios products.

**After the upgrade:**
- `magicbusstudios.com` (public) — marketing pages (existing, unchanged)
- `magicbusstudios.com/auth/login` — branded login page (NEW)
- `magicbusstudios.com/billing` — checkout, subscription management (NEW)
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

## Auth: Three Methods (All Passwordless)
- **Google SSO** — primary, existing across MBS
- **Nostr** — challenge → signature verification (experimental, from CWG)
- **LNURL-Auth** — Lightning wallet login (experimental, from CWG)

No email/password auth. No "or" divider with email login.

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
  → Redirect to: magicbusstudios.com/auth/login?redirect={origin}&brand={innerlab|mbs}
  → User sees branded login page
  → Authenticates via Google SSO / Nostr / LNURL
  → Platform creates/finds user, issues JWT
  → Redirect back: {origin}?token={JWT}
  → Product frontend stores JWT, sends in Authorization header
  → Product backend calls GET /api/entitlements/{product} to verify access
  → 401 = redirect to login | 403 = redirect to billing
```

## Payments: Dual System
- **Stripe** — subscriptions (monthly/yearly) + one-time purchases. Stripe prefixes by category: `[IL]`, `[Arcade]`, `[SW]`, `[MBS]`
- **BTCPay / Lightning** — alternative payment in sats

Both handled here. No per-product payment integration.

## Database: `mbs_platform`

### Core Models
- **User**: email, name, avatar, google_id, nostr_npub?, lnurl_linking_key?, auth_provider (google|nostr|lnurl), preferred_language, preferences {}, consent_preferences {}, is_admin, referral_code?, referred_by?, referral_count, created_at, updated_at, last_login
- **Entitlement**: user_id, category (innerlab|arcade|studioworks), type (product_pass|category_access|mbs_all_access), product (slug|null), status (active|expired|cancelled|trial), stripe_subscription_id?, stripe_customer_id, purchased_at, expires_at?, trial_ends_at?
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

## API Endpoints

**Phase 1 (MVP)**
- POST /api/auth/google — Google SSO, create/find user, return JWT
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

### Inner Lab (11 modules)
`cwg`, `flowstate`, `breatharc`, `starmap`, `astrocompass`, `arcana`, `archetypes`, `dreamlens`, `rituals`, `innerquest`, `nexus`

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
│   │   ├── AuthLoginPage.jsx    # NEW — branded SSO login
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
- MONGODB_URI (may already exist for other purposes)
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

## What NOT to Do
- Do NOT remove existing marketing pages or form handler — they keep working
- Do NOT modify CWG or FlowState backends until platform is built and tested
- Do NOT build Stripe into individual products — all billing through this platform
- Do NOT hardcode product slugs
- Do NOT create a separate `platform.magicbusstudios.com` domain — everything lives on `magicbusstudios.com`
