# MBS Platform — Unified Identity, Billing & Entitlements

## Overview
A centralized middleware for Magic Bus Studios that provides unified identity, billing, entitlements, email, promotions, and analytics across ALL 22 MBS products (Inner Lab, The Arcade, Studio Works). Each product keeps its own backend for product-specific logic — the platform sits above them as a shared services layer. This is a NEW standalone project.

## Key Design Principle
**The platform answers one question: "Does user X have access to product Y?"** Everything else (billing, promos, referrals, email, analytics) is built on top of that simple access check.

## Stack Deviations from Global
- **Backend-only project** — no React frontend. This is a middleware/API service consumed by all 22 product frontends.
- **Auth**: Google SSO + Nostr + LNURL-Auth — NO email/password auth. Three auth methods, all passwordless.
- **Payments**: Stripe + BTCPay (Lightning) — dual payment system at platform level. No per-product payment handling.
- **Stripe prefixes**: By category (`[IL]`, `[Arcade]`, `[SW]`, `[MBS]`) instead of per-app (`[CWG]`, `[FlowState]`)
- **Port**: 3002 (non-default)
- **DB_NAME**: `mbs_platform`
- **Branded login page**: Login UI changes based on origin — Inner Lab modules show IL branding, Arcade/SW show MBS branding (via `?brand=innerlab` or `?brand=mbs` parameter)

## Project Structure
```
server/
  src/
    routes/        # auth, entitlements, billing, promo, referral, email, profile, admin, health
    models/        # User, Entitlement, Transaction, Promotion, Referral, EmailPreference, ActivityLog, Announcement
    middleware/     # requireAuth, requireAdmin, rateLimiter, validateInput
    services/      # stripe, sendgrid, entitlement-checker
    scripts/       # cwg-migration
    config/        # product-catalog (slugs, categories, URLs — NOT hardcoded)
    utils/
  index.js
  Dockerfile
```

## Product Catalog
### Inner Lab (11 modules)
`cwg` (Active — has users to migrate), `flowstate` (Active), `breatharc`, `starmap`, `astrocompass`, `arcana`, `archetypes`, `dreamlens`, `rituals`, `innerquest`, `nexus` (all Coming Soon)

### The Arcade (5 games)
`brokenchain`, `mindhacker`, `triviaroast`, `whisperinghouse`, `fakeartist` — all at `{slug}.magicbusstudios.com`

### Studio Works (6 tools)
`wildlens`, `lazychef` (lazy-chef.mbs.com), `tasktracker`, `tutor`, `smartcart`, `moviepicker` — all at `{slug}.magicbusstudios.com`

**Do NOT hardcode slugs** — store in config or database so new products can be added without code changes.

## Core Data Models
- **User**: email (unique, indexed), name, avatar, google_id (unique, indexed), nostr_npub?, lnurl_linking_key?, auth_provider (google|nostr|lnurl), preferred_language, preferences {}, consent_preferences {}, is_admin, referral_code?, referred_by?, referral_count, created_at, updated_at, last_login
- **Entitlement**: user_id (indexed), category (innerlab|arcade|studioworks), type (product_pass|category_access|mbs_all_access), product (slug|null), status (active|expired|cancelled|trial), stripe_subscription_id?, stripe_customer_id, purchased_at, expires_at?, trial_ends_at?
- **Transaction**: user_id, type (purchase|subscription|refund|gift), category, product?, amount (cents), currency, stripe_payment_id, description, promo_code?, created_at
- **Promotion**: code (unique), type (percentage|fixed|free_product|free_trial_extension), value, applies_to (all|category|specific_product), category?, product?, max_uses, current_uses, valid_from, valid_until, created_by
- **Referral**: referrer_id, referral_code (unique), referred_email, referred_user_id?, status (pending|completed|rewarded), reward_type, reward_value, created_at
- **EmailPreference**: user_id (unique), marketing_emails, product_updates, billing_alerts, promotional_offers, unsubscribe_token (unique)
- **ActivityLog**: user_id, product, category, action (session_start|session_end|purchase|login), metadata {}, created_at
- **Announcement**: title, message, type (info|promo|new_product|maintenance), target (all|category|specific_product), active, valid_from, valid_until
- **Friend**: users (sorted pair of user_ids), source_product (which product the friendship originated from), created_at
- **Invite**: code (unique), from_user_id, product (which product the invite is for), expires_at (TTL), created_at
- **ActiveSession**: user_id, session_token (JWT jti), ip_address (hashed), user_agent, created_at, last_active
- **PushSubscription**: user_id, subscription (endpoint, keys), user_agent, subscribed_at
- **FeatureFlag**: flag_name, enabled, category, description, target_plans, target_users, created_at
- **ConsentAuditLog**: user_id, change_type, old_value, new_value, timestamp — GDPR/CCPA required, cannot delete
- **DataRequest**: user_id, request_type (access|deletion|portability|rectification), status, created_at
- **NostrChallenge**: challenge, user_id?, expires_at — temporary auth challenges
- **LnurlChallenge**: k1, status, user_id?, expires_at — LNURL-Auth challenges
- **BtcpayInvoice**: user_id, invoice_id, amount_sats, status, product?, created_at — Lightning payment records

## Entitlement Types
- `product_pass` — access to one specific product (one-time or subscription)
- `category_access` — access to all products in a category (Inner Lab / Arcade / Studio Works)
- `mbs_all_access` — access to everything

## Login Flow (Cross-Product SSO)
For Google OAuth implementation patterns, see `~/.claude/skills/google-oauth-setup/SKILL.md`
```
User clicks "Login" on any product (e.g., conversationswithgod.ai)
  → Redirect to: platform.magicbusstudios.com/auth/login?redirect={origin}&brand={innerlab|mbs}
  → Login page shows branded UI (Inner Lab branding for IL modules, MBS branding for Arcade/SW)
  → User authenticates via Google SSO, Nostr, or LNURL-Auth
  → Platform creates/finds user, issues JWT
  → Redirect back: {origin}?token={JWT}
  → Product frontend stores JWT, sends in Authorization header
  → Product backend verifies JWT via platform's /entitlements/:product
  → 401 = redirect to platform login | 403 = redirect to platform billing
```

### Branded Login Mapping
| User comes from | Brand parameter | Login page shows |
|---|---|---|
| Any Inner Lab module (CWG, FlowState, etc.) | `brand=innerlab` | Inner Lab logo + branding |
| Any Arcade game | `brand=mbs` | MagicBusStudios logo + branding |
| Any Studio Works tool | `brand=mbs` | MagicBusStudios logo + branding |
| innerlab.ai | `brand=innerlab` | Inner Lab logo + branding |
| magicbusstudios.com | `brand=mbs` | MagicBusStudios logo + branding |

## API Endpoints
**Phase 1 (MVP)**
- POST /auth/google — Google SSO, create/find user, return JWT
- POST /auth/nostr — Nostr authentication (challenge → verify signature)
- POST /auth/lnurl — LNURL-Auth (k1 challenge → verify signature)
- GET /auth/me — current user + entitlements summary
- POST /auth/logout — invalidate session
- DELETE /auth/account — delete account + all data (GDPR). See `~/.claude/skills/compliance-gdpr-ccpa/SKILL.md`
- GET /entitlements — all entitlements for current user
- GET /entitlements/:product — check access → `{ hasAccess, reason }`
- GET /entitlements/category/:cat — all accessible products in category
- POST /billing/checkout — create Stripe checkout `{ product, category, promo_code? }`
- POST /billing/portal — Stripe customer portal URL
- POST /billing/webhook — Stripe webhook (payment events, sub changes) [Stripe sig auth]
- GET /billing/history — transaction history
- GET /health — health check
- For Stripe checkout, webhook, and portal patterns, see `~/.claude/skills/stripe-integration/SKILL.md`

**Phase 2**
- GET|PUT /email/preferences — email opt-in settings
- POST /email/unsubscribe — one-click unsubscribe [token auth]
- GET /admin/users — list users [admin]
- POST /admin/entitlements/grant|revoke — manual entitlement control [admin]
- For SendGrid transactional email patterns, see `~/.claude/skills/sendgrid-setup/SKILL.md`

**Phase 3**
- GET /promo/validate/:code — check promo validity [no auth]
- POST /promo/redeem — apply promo code
- POST /promo/create, GET /promo/list, DELETE /promo/:code — [admin]
- GET /referral/link — user's unique referral link
- POST /referral/invite — send referral email
- GET /referral/stats — referral count + rewards
- POST /referral/claim — claim referral on signup

**Phase 5+**
- GET|PUT /profile — full profile + update settings
- GET /profile/activity — cross-product activity history
- POST /profile/export — GDPR data export

## Data Migration
For cross-database migration patterns, see `~/.claude/skills/mongodb-shared-cluster/SKILL.md`

### CWG Migration (56 collections)
CWG has ~10 existing users in `conversations_with_god` database. Migration analysis: `CWG/MBS_DATABASE_MIGRATION_PLAN.md`
- **Bucket 1 → `mbs_platform`**: User identity (email, name, google_id, nostr_npub, lnurl_linking_key), Stripe fields, consent, active sessions, LNURL/Nostr challenges, GDPR data requests, consent audit log
- **Bucket 2 → `inner_lab` with `cwg_*` prefix**: All 39 CWG-specific collections (chats, journals, meditations, programs, badges, BTCPay invoices, community, sharing, etc.)
- **Bucket 3 → starts as `cwg_*`, promote to `il_*` later**: Consciousness profiles, personal history, check-ins, user memories, blockchain anchors, analytics events, notifications, push subscriptions, feature flags

### FlowState Migration (7 collections)
FlowState has 0 real users in `yogaghost` database. Migration analysis: `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md`
- **Bucket 1 → `mbs_platform`**: User identity (email, name, googleId), Stripe fields, email preferences
- **Bucket 2 → `inner_lab` with `yoga_*` prefix**: Sessions, achievements, community flows, user profiles (settings, streaks, favorites)
- **Bucket 3 → shared**: Friends → platform, invites → platform, activity → Inner Lab level, health conditions/injuries → potentially shared `il_*`

### Migration Rules
- Old databases (`conversations_with_god`, `yogaghost`) stay untouched as backups
- New data is COPIED, not moved
- Only delete old DBs after weeks/months of verified operation
- CWG's Stripe and BTCPay integrations move to MBS Platform — no per-product payment handling

## Deployment
For Coolify Docker deployment patterns, see `~/.claude/skills/coolify-deployment/SKILL.md`

- **Domain**: platform.magicbusstudios.com
- **Port**: 3002
- **CORS_ORIGINS**: must include ALL 22 product frontend domains + magicbusstudios.com + innerlab.ai

## Environment Variables
**Required (MVP)**
- MONGODB_URI, DB_NAME (`mbs_platform`)
- JWT_SECRET (64-char), JWT_EXPIRY (`7d`)
- GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- BTCPAY_URL, BTCPAY_API_KEY, BTCPAY_STORE_ID
- SENDGRID_API_KEY, FROM_EMAIL (`noreply@magicbusstudios.com`)
- PORT (`3002`), CORS_ORIGINS (all product URLs, comma-separated)

## Existing MBS Infrastructure (Do NOT Touch)
| Service | Domain | Notes |
|---|---|---|
| MBS Frontend | magicbusstudios.com | Studio marketing site |
| MBS Form Handler | api.magicbusstudios.com | Contact/subscribe/waitlist — SEPARATE service, do NOT merge |
| Inner Lab | innerlab.ai | Product marketing site |
| CWG | conversationswithgod.ai | Has existing users + Stripe WIP |
| 5 Arcade games | *.magicbusstudios.com | Individual frontends + backends |
| 6 Studio Works | *.magicbusstudios.com | Individual frontends + backends |

## What NOT to Do (Project-Specific)
- Do NOT modify any existing product backend until Phase 4
- Do NOT build Stripe into CWG or any product directly — all billing through this platform
- Do NOT merge with the form handler at api.magicbusstudios.com
- Do NOT hardcode product slugs — config or database
- Do NOT build play time / session tracking — removed from scope

## Three-Layer Architecture
This project is **Layer 1** in a three-layer system:
- **Layer 1 (this project)**: MBS Platform — SSO, billing, entitlements, friends, invites, push, feature flags, GDPR
- **Layer 2 (separate project, build alongside this)**: Inner Lab Middleware — shared consciousness, memories, check-ins, wellness profiles, activity feed, encryption/export. Backend-only API, no frontend. See `InnerLab-middleware/CLAUDE.md` for reference. **Build NOW** so CWG/FlowState migrations can write shared data directly to il_* tables instead of double-migrating later.
- **Layer 2 Frontend (build LATER)**: Inner Lab dashboard at innerlab.ai — daily briefing, cross-module insights, unified profile. Only when 2-3 modules have enough data to show.
- **Layer 3**: Standalone products (Arcade, Studio Works) — own databases, SSO only

Inner Lab modules (CWG, FlowState, etc.) are separate containers that all connect to `inner_lab` database. They call this platform for auth/billing and the Inner Lab Middleware for shared data.

## Current Status
Not started — spec only. All architecture decisions finalized (2026-03-25).

Reference documents:
- `MBS_Platform_Technical_Architecture.docx` — original spec
- `SESSION_HANDOFF.md` — all decisions and context
- `CWG/MBS_DATABASE_MIGRATION_PLAN.md` — CWG 56-collection bucket analysis
- `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md` — FlowState 7-collection bucket analysis

## Build Order (Revised 2026-03-25)
1. **MBS Platform (this project)** — SSO + billing + entitlements + friends + push + feature flags + GDPR
2. **Inner Lab Middleware (separate project)** — shared il_* collections, consciousness profile, user memories, check-ins, wellness profiles. Build NOW so migrations write shared data correctly from day one. See `InnerLab-middleware/CLAUDE.md`.
3. **CWG Migration** — identity → mbs_platform, CWG-specific → inner_lab cwg_*, shared data → inner_lab il_* via middleware
4. **FlowState Migration** — same pattern as CWG
5. **New Inner Lab modules** (BreathArc, StarMap, etc.) — born into the architecture
6. **Inner Lab Frontend** (innerlab.ai dashboard) — built LATER when enough modules/data exist

## Implementation Phases (MBS Platform)
**Phase 1 (MVP)** — Google SSO + Nostr/LNURL auth + user profiles + entitlements + Stripe billing + BTCPay Lightning + webhooks + branded login + friends/invites + push subscriptions + feature flags + GDPR consent
**Phase 2** — Transactional emails + basic admin (list users, grant/revoke) + email preferences
**Phase 3** — Promo codes + free trials + referral program + win-back offers
**Phase 4** — Connect each product backend (add checkAccess middleware to all 22 products)
**Phase 5** — User dashboard (My Products, billing history, profile settings, onboarding)
**Phase 6** — Admin dashboard (analytics, revenue, product stats, funnel tracking, audit log)
**Phase 7** — Email campaigns + announcements + surveys + newsletter
**Phase 8** — Advanced: multi-currency, family plan, teams, achievements, push notifications, enterprise SSO, API keys
