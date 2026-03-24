# MBS Platform — Unified Identity, Billing & Entitlements

## Overview
A centralized middleware for Magic Bus Studios that provides unified identity, billing, entitlements, email, promotions, and analytics across ALL 22 MBS products (Inner Lab, The Arcade, Studio Works). Each product keeps its own backend for product-specific logic — the platform sits above them as a shared services layer. This is a NEW standalone project.

## Key Design Principle
**The platform answers one question: "Does user X have access to product Y?"** Everything else (billing, promos, referrals, email, analytics) is built on top of that simple access check.

## Stack Deviations from Global
- **Backend-only project** — no React frontend. This is a middleware/API service consumed by all 22 product frontends.
- **Auth**: Google SSO only — NO email/password auth. Unlike other MBS apps, there is no "or" divider with email login.
- **Stripe prefixes**: By category (`[IL]`, `[Arcade]`, `[SW]`, `[MBS]`) instead of per-app (`[CWG]`, `[FlowState]`)
- **Port**: 3002 (non-default)
- **DB_NAME**: `mbs_platform`

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
- **User**: email (unique, indexed), name, avatar, google_id (unique, indexed), preferred_language, preferences {}, created_at, updated_at, last_login
- **Entitlement**: user_id (indexed), category (innerlab|arcade|studioworks), type (product_pass|category_access|mbs_all_access), product (slug|null), status (active|expired|cancelled|trial), stripe_subscription_id?, stripe_customer_id, purchased_at, expires_at?, trial_ends_at?
- **Transaction**: user_id, type (purchase|subscription|refund|gift), category, product?, amount (cents), currency, stripe_payment_id, description, promo_code?, created_at
- **Promotion**: code (unique), type (percentage|fixed|free_product|free_trial_extension), value, applies_to (all|category|specific_product), category?, product?, max_uses, current_uses, valid_from, valid_until, created_by
- **Referral**: referrer_id, referral_code (unique), referred_email, referred_user_id?, status (pending|completed|rewarded), reward_type, reward_value, created_at
- **EmailPreference**: user_id (unique), marketing_emails, product_updates, billing_alerts, promotional_offers, unsubscribe_token (unique)
- **ActivityLog**: user_id, product, category, action (session_start|session_end|purchase|login), metadata {}, created_at
- **Announcement**: title, message, type (info|promo|new_product|maintenance), target (all|category|specific_product), active, valid_from, valid_until

## Entitlement Types
- `product_pass` — access to one specific product (one-time or subscription)
- `category_access` — access to all products in a category (Inner Lab / Arcade / Studio Works)
- `mbs_all_access` — access to everything

## Login Flow (Cross-Product SSO)
For Google OAuth implementation patterns, see `~/.claude/skills/google-oauth-setup/SKILL.md`
```
User clicks "Login" on any product (e.g., conversationswithgod.ai)
  → Redirect to: platform.magicbusstudios.com/auth/login?redirect={origin}
  → Google SSO → Platform creates/finds user, issues JWT
  → Redirect back: {origin}?token={JWT}
  → Product frontend stores JWT, sends in Authorization header
  → Product backend verifies JWT via platform's /entitlements/:product
  → 401 = redirect to platform login | 403 = redirect to platform billing
```

## API Endpoints
**Phase 1 (MVP)**
- POST /auth/google — Google SSO, create/find user, return JWT
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

## CWG User Migration
For cross-database migration patterns, see `~/.claude/skills/mongodb-shared-cluster/SKILL.md`

CWG has 5–10 existing users in a separate MongoDB database. Migration script must:
1. Read users from CWG database
2. Create matching records in `mbs_platform.users` (email, name, avatar, google_id, preferred_language, created_at)
3. Create entitlements based on current plan (free or premium)
4. Product-specific data (chat sessions, consciousness profiles, message counts) stays in CWG database

**⚠ Inspect the actual CWG database/codebase for the latest schema before writing the migration script** — the architecture document's field mapping may not be current.

**⚠ CWG currently has its own Stripe integration being built. All Stripe billing moves to this platform — do NOT build Stripe per-product.**

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

## Current Status
Not started — spec only. Reference document: `MBS_Platform_Technical_Architecture.docx`

## Implementation Phases
**Phase 1 (MVP)** — Google SSO + user profiles + entitlements + Stripe billing + webhooks + CWG migration
**Phase 2** — Transactional emails + basic admin (list users, grant/revoke) + email preferences
**Phase 3** — Promo codes + free trials + referral program + win-back offers
**Phase 4** — Connect each product backend (add checkAccess middleware to all 22 products)
**Phase 5** — User dashboard (My Products, billing history, profile settings, onboarding)
**Phase 6** — Admin dashboard (analytics, revenue, product stats, funnel tracking, audit log)
**Phase 7** — Email campaigns + announcements + surveys + newsletter
**Phase 8** — Advanced: multi-currency, family plan, teams, achievements, push notifications, SSO, API keys
