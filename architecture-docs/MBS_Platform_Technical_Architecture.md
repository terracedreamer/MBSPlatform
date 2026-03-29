# Magic Bus Studios — Platform Technical Architecture

**Version**: 1.0
**Date**: March 28, 2026
**Author**: Magic Bus Studios Engineering
**Status**: All 5 build phases complete. Platform live and operational.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Three-Layer Architecture](#2-three-layer-architecture)
3. [Authentication System](#3-authentication-system)
4. [Entitlement & Billing System](#4-entitlement--billing-system)
5. [Database Architecture](#5-database-architecture)
6. [Inner Lab Shared Data Layer](#6-inner-lab-shared-data-layer)
7. [API Reference](#7-api-reference)
8. [Product Catalog](#8-product-catalog)
9. [Data Deletion (GDPR)](#9-data-deletion-gdpr)
10. [Deployment Infrastructure](#10-deployment-infrastructure)
11. [Security Model](#11-security-model)
12. [Migration History](#12-migration-history)
13. [Known Limitations & Future Work](#13-known-limitations--future-work)

---

## 1. System Overview

Magic Bus Studios (MBS) operates a unified platform that provides single sign-on (SSO) authentication, centralized billing, and entitlement management across an ecosystem of 15+ web applications.

The platform answers one core question: **"Does user X have access to product Y?"**

Everything else — billing, promotions, referrals, analytics — is built on top of that simple access check.

### The Ecosystem

- **Magic Bus Studios** (magicbusstudios.com) — The platform hub. Handles all authentication, billing, and account management.
- **Inner Lab** (innerlab.ai) — A unified system for inner growth. 11 modules sharing a common data layer for cross-module intelligence.
- **The Arcade** — 5 AI-powered multiplayer games.
- **Studio Works** — 6 AI-powered productivity tools.

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React (Vite or Next.js), Tailwind CSS, shadcn/ui |
| Backend | Node.js (Express) or Python (FastAPI) |
| Database | MongoDB (single shared cluster, separate DB per concern) |
| Auth | JWT (HS256), Google OAuth 2.0, Nostr, LNURL-Auth |
| Payments | Stripe (subscriptions + one-time), BTCPay Server (Lightning/Bitcoin) |
| Email | SendGrid (transactional + marketing) |
| AI | OpenAI API (behind service layers) |
| Hosting | Coolify (self-hosted VPS), Nixpacks or Dockerfile builds |
| DNS/Proxy | Traefik (via Coolify) |

---

## 2. Three-Layer Architecture

```
+------------------------------------------------------------------+
|  LAYER 1: MBS Platform                                            |
|  Domain: magicbusstudios.com / api.magicbusstudios.com           |
|  Database: mbs_platform                                           |
|                                                                    |
|  Responsibilities:                                                 |
|  - SSO (4 auth methods + 2FA/TOTP)                               |
|  - JWT issuance and validation                                     |
|  - Stripe checkout + customer portal                              |
|  - BTCPay Lightning payments                                       |
|  - Entitlement checks (product, category, all-access)             |
|  - Friends, invites, referrals, promo codes                       |
|  - Email preferences + transactional emails                       |
|  - Admin panel                                                     |
|  - GDPR account deletion (cascades to all layers)                 |
|  - Marketing website + login/signup pages                         |
+------------------------------------------------------------------+
           |                              |
           | JWT validation               | JWT validation
           v                              v
+-------------------------+    +---------------------------+
| LAYER 2: Inner Lab      |    | LAYER 3: Standalone       |
| Domain: innerlab.ai     |    | Products                  |
| Database: inner_lab     |    |                           |
|                          |    | Each product has:         |
| Middleware:              |    | - Its own domain          |
|  il_* shared collections |    | - Its own database        |
|  consciousness profiles  |    | - JWT validation only     |
|  memories, check-ins     |    | - Entitlement check       |
|  wellness, activity feed |    | - No shared data layer    |
|                          |    |                           |
| Modules (same DB):       |    | Arcade (5 games):         |
|  CWG (cwg_*)            |    |  BrokenChain, MindHacker  |
|  FlowState (yoga_*)     |    |  TriviaRoast, FakeArtist  |
|  BreathArc (breath_*)   |    |  WhisperingHouse          |
|  StarMap (astro_*)      |    |                           |
|  + 7 more coming soon   |    | Studio Works (6 tools):   |
|                          |    |  WildLens, LazyChef       |
| Dashboard:               |    |  TaskTracker, AITutor     |
|  innerlab.ai/dashboard  |    |  SmartCart, MoviePicker    |
|  innerlab.ai/auth/*     |    |                           |
+-------------------------+    +---------------------------+
```

### How the Layers Interact

1. **User visits any product** (e.g., wildlens.magicbusstudios.com)
2. **Clicks "Sign In"** -> redirected to MBS Platform login (magicbusstudios.com/auth/login)
   - Inner Lab modules redirect to innerlab.ai/auth/login (Inner Lab branding)
   - Standalone products redirect to magicbusstudios.com/auth/login (MBS branding)
3. **User authenticates** (Google SSO, email/password, Nostr, or LNURL)
4. **Platform issues JWT** and redirects back to the product with `?token=JWT` in the URL
5. **Product stores JWT** in localStorage and sends it as `Authorization: Bearer` header on all API calls
6. **Product's backend verifies JWT** using the shared `JWT_SECRET` (HS256)
7. **Product checks entitlement** via `GET api.magicbusstudios.com/api/entitlements/{slug}`

---

## 3. Authentication System

### Four Authentication Methods

| Method | Flow | User Identity |
|--------|------|---------------|
| **Google SSO** | Google One Tap -> ID token -> platform verifies -> JWT issued | email + google_id |
| **Email/Password** | Signup with email verification, bcrypt hashed passwords | email + password_hash |
| **Nostr** | Challenge-response with Nostr keypair signature | nostr_npub (no email) |
| **LNURL-Auth** | Lightning wallet signs a challenge | lnurl_linking_key (no email) |

### Two-Factor Authentication (2FA)

- TOTP-based (Google Authenticator, Authy, etc.)
- 10 backup codes generated at setup (bcrypt hashed)
- Optional per user — enabled in account settings
- Enforced on email/password login when enabled

### JWT Details

| Field | Value |
|-------|-------|
| Algorithm | HS256 (symmetric, shared secret) |
| Expiry | 7 days |
| Payload | `{ userId, email, name, avatar, isAdmin, iat, exp }` |
| Header | `Authorization: Bearer <JWT>` |
| Storage | `localStorage.setItem("mbs_token", token)` |

**Important notes:**
- `userId` is a string (MongoDB ObjectId.toString())
- `email` can be null (Nostr/LNURL users are pseudonymous)
- All products use the same `JWT_SECRET` to verify tokens
- Token arrives via `?token=JWT` URL parameter after login redirect

### Login Pages

Two login pages exist with different branding but identical functionality:

| URL | Branding | Used By |
|-----|----------|---------|
| magicbusstudios.com/auth/login | MBS branding | Arcade games, Studio Works tools |
| innerlab.ai/auth/login | Inner Lab branding | CWG, FlowState, all Inner Lab modules |

Both call the same MBS Platform auth API endpoints. Both support all 4 auth methods + 2FA.

---

## 4. Entitlement & Billing System

### Entitlement Types

| Type | Description | Example |
|------|-------------|---------|
| `product_pass` | Access to one specific product | CWG monthly subscription |
| `category_access` | Access to all products in a category | Inner Lab All Access |
| `mbs_all_access` | Access to everything | MBS All Access bundle |
| `free_tier` | Default — limited access (product defines limits) | Arcade: 30 min/day |

### Entitlement Check

Every product calls this API to determine access level:

```
GET https://api.magicbusstudios.com/api/entitlements/{product_slug}
Authorization: Bearer <JWT>

Response: { success: true, hasAccess: true, reason: "free_tier" }
```

Reason values: `product_pass`, `category_access`, `mbs_all_access`, `free_tier`, `no_subscription`

**Caching:** All products cache the response for 5 minutes in-memory (keyed by userId + productSlug).

**Fail-open:** If the platform API is unreachable, products default to allowing access (free_tier). This prevents platform downtime from breaking every product.

### Payment Systems

**Stripe:**
- Subscriptions (monthly/annual) and one-time purchases
- Checkout sessions created by MBS Platform backend
- Customer portal for subscription management
- Webhooks handle payment events and create/update entitlements

**BTCPay Server (Lightning):**
- Bitcoin and Lightning Network payments
- 30-day non-recurring passes (no auto-renew)
- HMAC-SHA256 webhook verification

### Pricing (Current)

| Product | Monthly | Annual | Lightning |
|---------|---------|--------|-----------|
| CWG | $9.99 | $79.99 | 21K sats/mo |
| Inner Lab All Access | $19.99 | $159.99 | TBD |
| MBS All Access | $29.99 | $249.99 | TBD |
| Arcade / Studio Works | Free (premium TBD) | — | — |

---

## 5. Database Architecture

All applications share a single MongoDB cluster (self-hosted on the same VPS via Coolify). Each concern gets its own database.

### Database Map

| Database | Owner | Contents |
|----------|-------|----------|
| `mbs_platform` | MBS Platform (Layer 1) | Users, entitlements, transactions, promotions, referrals, friends, invites, sessions, feature flags, announcements, consent logs, data requests |
| `inner_lab` | Inner Lab Middleware (Layer 2) + all IL modules | Shared `il_*` collections + module-prefixed collections (`cwg_*`, `yoga_*`, `breath_*`, etc.) |
| `brokenchain` | BrokenChain | Game rooms, players, game results |
| `mindhacker` | MindHacker | Players, rooms, game results |
| `triviaroast` | Trivia Roast | Games, leaderboard |
| `fakeartist` | Fake Artist | Rooms (24hr TTL) |
| `whispering_house` | Whispering House | Sessions (ephemeral) |
| `wildlens` | WildLens | Discoveries, posts, collections, expeditions |
| `lazy_chef` | Lazy Chef | Recipes, ingredients, meal plans, shopping lists |
| `moviepicker` | Movie Picker | Watchlists, user interactions |
| `smartcart` | SmartCart | Shopping lists, pantry, purchase history |
| `tasktracker` | TaskTracker | Families, tasks, goals, approvals |
| `ai_tutor` | AI Tutor | Users, progress, chat history, flashcards |

### Core Data Models (mbs_platform)

**User:**
- `_id` (ObjectId), `email` (unique, sparse), `name`, `avatar`
- `password_hash` (bcrypt, optional), `google_id`, `nostr_npub`, `lnurl_linking_key`
- `auth_methods` [String] — e.g., `["email", "google"]`
- `email_verified`, `totp_enabled`, `totp_secret`, `totp_backup_codes`
- `preferred_language`, `preferences`, `consent_preferences`
- `is_admin`, `stripe_customer_id`, `referral_code`, `referred_by`
- `created_at`, `updated_at`, `last_login`

**Entitlement:**
- `user_id`, `category` (innerlab/arcade/studioworks), `type` (product_pass/category_access/mbs_all_access)
- `product` (slug), `status` (active/cancelled/expired)
- `stripe_subscription_id`, `purchased_at`, `expires_at`, `trial_ends_at`

**Transaction:**
- `user_id`, `type` (stripe_subscription/stripe_one_time/btcpay_lightning)
- `category`, `product`, `amount`, `currency`, `stripe_payment_id`

---

## 6. Inner Lab Shared Data Layer

Inner Lab modules share the `inner_lab` database. Collections are namespaced by prefix.

### Shared Collections (il_* — owned by Inner Lab Middleware)

| Collection | Purpose | Who Writes |
|-----------|---------|-----------|
| `il_consciousness_profiles` | Spiritual archetype, orientation, assessment | CWG, dashboard |
| `il_consciousness_snapshots` | Historical profile changes | CWG, dashboard |
| `il_personal_histories` | User's life story for AI personalization | CWG, dashboard |
| `il_check_ins` | Daily mood, energy, stress, intention | Any module, dashboard |
| `il_user_memories` | AI-extracted facts about the user | Any module (privacy-controlled) |
| `il_user_wellness_profiles` | Health conditions, injuries, goals | FlowState, BreathArc |
| `il_activity_feed` | Cross-module activity events | Any module |
| `il_blockchain_anchors` | OpenTimestamps data integrity proofs | Middleware |
| `il_sync_backups` | Local-first storage sync | Middleware |
| `il_analytics_events` | User event tracking (90-day TTL) | Any module |
| `il_notifications` | Cross-module notifications | Any module |

### Module Collections (owned by each module)

| Module | Prefix | Example Collections |
|--------|--------|-------------------|
| CWG | `cwg_` | cwg_chat_sessions, cwg_messages, cwg_journal_entries, cwg_practices, ... (39 collections) |
| FlowState | `yoga_` | yoga_user_profiles, yoga_sessions, yoga_achievements, yoga_community_flows, yoga_activity |

### User Memory Privacy Model

Memories are private by default. Cross-module sharing requires explicit user opt-in:
- Each memory has `source_module` and `shared` (boolean) fields
- `shared: false` — only the originating module can read it
- `shared: true` — all Inner Lab modules can read it
- User controls sharing via the Memories page on innerlab.ai/memories

### Database Rules

- Modules **write** to their own prefix and to `il_check_ins`, `il_user_memories`, `il_activity_feed`
- Modules **read** from `il_*` shared collections to personalize the experience
- Modules **never write** to another module's prefix
- The middleware **reads** from all prefixes (for cross-module insights) and **writes** only to `il_*`
- All documents use `user_id` (snake_case string) — the platform's `userId` from the JWT

---

## 7. API Reference

### MBS Platform (api.magicbusstudios.com)

**Authentication:**
| Method | Path | Description |
|--------|------|-------------|
| POST | /api/auth/google | Google SSO (ID token verification) |
| POST | /api/auth/signup | Email/password registration |
| POST | /api/auth/login | Email/password login |
| POST | /api/auth/nostr | Nostr challenge-response |
| POST | /api/auth/lnurl | LNURL-Auth |
| POST | /api/auth/2fa/verify | 2FA TOTP verification |
| POST | /api/auth/2fa/setup | Enable 2FA |
| POST | /api/auth/forgot-password | Password reset request |
| POST | /api/auth/reset-password | Password reset with token |
| GET | /api/auth/me | Current user + entitlements |
| POST | /api/auth/logout | Invalidate session |
| POST | /api/auth/refresh | Refresh JWT |
| DELETE | /api/auth/account | Full account deletion (GDPR) |

**Entitlements:**
| Method | Path | Description |
|--------|------|-------------|
| GET | /api/entitlements | All entitlements for user |
| GET | /api/entitlements/:product | Check access for specific product |
| GET | /api/entitlements/category/:cat | Products in category |

**Billing:**
| Method | Path | Description |
|--------|------|-------------|
| POST | /api/billing/checkout | Create Stripe checkout session |
| POST | /api/billing/portal | Stripe customer portal link |
| POST | /api/billing/webhook | Stripe webhook handler |
| POST | /api/billing/btcpay/checkout | BTCPay Lightning checkout |
| POST | /api/billing/btcpay/webhook | BTCPay webhook handler |
| GET | /api/billing/history | Transaction history |

**Social:**
| Method | Path | Description |
|--------|------|-------------|
| GET | /api/friends | List friends |
| POST | /api/friends/invite | Create friend invite |
| POST | /api/friends/accept | Accept invite |

### Inner Lab Middleware (api.innerlab.ai)

| Method | Path | Description |
|--------|------|-------------|
| POST | /api/check-in | Store mood/energy/stress |
| GET | /api/check-in/latest | Most recent check-in |
| GET | /api/check-in/history | Check-in history |
| GET | /api/consciousness | Consciousness profile |
| PUT | /api/consciousness | Update profile |
| GET | /api/memories | User memories (filtered) |
| POST | /api/memories | Create memory |
| PUT | /api/memories/:id/share | Share memory cross-module |
| POST | /api/activity | Log activity event |
| GET | /api/activity/feed | Cross-module feed |

---

## 8. Product Catalog

### Inner Lab Modules (11)

| Slug | Name | Status | Domain | Stack |
|------|------|--------|--------|-------|
| `cwg` | Conversations With God | Active | conversationswithgod.ai | Python (FastAPI) |
| `flowstate` | FlowState | Active | yoga.magicbusstudios.com | Node.js (Express) |
| `breatharc` | BreathArc | Coming Soon | TBD | TBD |
| `starmap` | StarMap | Coming Soon | TBD | TBD |
| `astrocompass` | AstroCompass | Coming Soon | TBD | TBD |
| `arcana` | Arcana | Coming Soon | TBD | TBD |
| `archetypes` | Archetypes | Coming Soon | TBD | TBD |
| `dreamlens` | DreamLens | Coming Soon | TBD | TBD |
| `rituals` | Rituals | Coming Soon | TBD | TBD |
| `innerquest` | InnerQuest | Coming Soon | TBD | TBD |
| `nexus` | Nexus | Coming Soon | TBD | TBD |

### The Arcade (5 games)

| Slug | Name | Domain |
|------|------|--------|
| `brokenchain` | Broken Chain | brokenchain.magicbusstudios.com |
| `mindhacker` | MindHacker | mindhacker.magicbusstudios.com |
| `triviaroast` | Trivia Roast | triviaroast.magicbusstudios.com |
| `fakeartist` | Fake Artist | fakeartist.magicbusstudios.com |
| `whisperinghouse` | Whispering House | whisperinghouse.magicbusstudios.com |

### Studio Works (6 tools)

| Slug | Name | Domain |
|------|------|--------|
| `wildlens` | WildLens | wildlens.magicbusstudios.com |
| `lazychef` | Lazy Chef | lazy-chef.magicbusstudios.com |
| `tasktracker` | Family Task Tracker | tasktracker.magicbusstudios.com |
| `tutor` | AI Tutor | tutor.magicbusstudios.com |
| `smartcart` | SmartCart | smartcart.magicbusstudios.com |
| `moviepicker` | Movie Picker | moviepicker.magicbusstudios.com |

---

## 9. Data Deletion (GDPR)

Data deletion is layered — not a single nuclear cascade:

| Level | Where Triggered | What Gets Deleted | What Stays |
|-------|----------------|-------------------|-----------|
| **App-level** | Settings within each app | Only that app's data in its own database | MBS account, entitlements, all other apps |
| **Category-level** | magicbusstudios.com/settings | All data from all apps in one category | MBS account, other categories |
| **Full account** | magicbusstudios.com/settings | Everything everywhere | Nothing |

**Key principle:** A user clicking "Delete my data" inside WildLens should NOT delete their MBS account or their SmartCart data. Only the MBS Platform (magicbusstudios.com) has authority for cross-product or full account deletion.

Each product implements `DELETE /api/user-data` which:
- Deletes all data for the authenticated user from that app's database only
- Is called by the user from within the app (app-level deletion)
- Is called by MBS Platform during category or full account deletion

---

## 10. Deployment Infrastructure

### Hosting

All applications run on Coolify (self-hosted VPS). Each application deploys as either:
- **Two containers**: frontend (nginx) + backend (Node.js/Python) — most apps
- **Single container**: Express serves both API and static frontend — some simpler apps (e.g., SmartCart)

### Container Pattern (Two-Container)

**Frontend:** Multi-stage Docker build — Node.js builds the React app, nginx serves the static files.
- `VITE_*` or `NEXT_PUBLIC_*` env vars are **build args** (baked in at build time)
- nginx handles SPA routing with `try_files $uri $uri/ /index.html`

**Backend:** Node.js or Python container running Express/FastAPI.
- Runtime env vars for secrets (JWT_SECRET, OPENAI_API_KEY, MONGO_URL, etc.)
- Health check endpoint at `/health` or `/api/health`

### Environment Variables (Common Across All Products)

| Variable | Purpose | Where |
|----------|---------|-------|
| `JWT_SECRET` | Must match MBS Platform exactly | Backend runtime |
| `MONGO_URL` | MongoDB connection string (also accepts MONGODB_URI, MONGO_URI) | Backend runtime |
| `DB_NAME` | Product's database name | Backend runtime |
| `PLATFORM_URL` | `https://magicbusstudios.com` | Backend runtime |
| `PRODUCT_SLUG` | Product identifier for entitlements | Backend runtime |
| `VITE_BACKEND_URL` / `NEXT_PUBLIC_API_URL` | Backend API URL | Frontend build arg |

### Domain Structure

```
magicbusstudios.com          — MBS Platform frontend
api.magicbusstudios.com      — MBS Platform backend
innerlab.ai                  — Inner Lab frontend
api.innerlab.ai              — Inner Lab backend
conversationswithgod.ai      — CWG frontend
api.conversationswithgod.ai  — CWG backend
*.magicbusstudios.com        — All other products (frontend)
api.*.magicbusstudios.com    — All other products (backend)
```

---

## 11. Security Model

### Authentication Security
- Passwords hashed with bcrypt (cost factor 10)
- JWT signed with HS256 shared secret (planned upgrade to RS256 asymmetric)
- 2FA/TOTP with bcrypt-hashed backup codes
- Rate limiting: 100 req/15min general, 20 req/15min auth, 30 req/15min billing
- Open redirect protection on login redirects (validated against CORS_ORIGINS)

### Data Security
- All API routes require JWT authentication (except health checks and public content)
- Input validation on all routes (express-validator or Pydantic)
- All user text input sanitized
- AI/LLM API keys never exposed to frontend — always behind backend service layer
- CORS configured per-product (each product's domain must be in the list)

### Privacy
- Nostr and LNURL users can be fully pseudonymous (no email required)
- Inner Lab memories are private by default — cross-module sharing requires explicit opt-in
- GDPR-compliant account deletion with three granularity levels
- Consent audit logging for preference changes

---

## 12. Migration History

### Phase 1: MBS Platform (March 25-26, 2026)
Built SSO + billing + entitlements at magicbusstudios.com. Google SSO, Nostr, LNURL, Stripe, BTCPay, friends/invites, GDPR delete, admin panel. Later added email/password auth + 2FA (Addendum #14-15).

### Phase 2: Inner Lab Middleware (March 26, 2026)
Built il_* APIs + dashboard at innerlab.ai. 11 Mongoose models, 22 API routes, 4 dashboard pages, 4 auth pages (login/signup/forgot-password/reset-password).

### Phase 3: CWG Migration (March 26-27, 2026)
- **3A (Data):** Migration script copied 20 users to mbs_platform, 14 collections to inner_lab (cwg_* + il_*)
- **3B (Refactor):** 41 backend + 30+ frontend files modified, 31 deleted. Python JWT middleware (PyJWT), collection references updated to cwg_* prefix, standalone auth/billing removed.

### Phase 4: FlowState Migration (March 27-28, 2026)
28 files changed. Express JWT middleware, collection references updated to yoga_* prefix, standalone auth removed. 0 users to migrate (no production data).

### Phase 5: Standalone Products (March 28, 2026)
All 11 standalone products (5 Arcade + 6 Studio Works) migrated to platform SSO. JWT middleware added, standalone auth removed, entitlement check wired, legacy user collision handled.

**Key bug discovered:** Legacy users with different `_id` than platform `userId` crash on unique email index during auto-provisioning. Fix: email-based lookup, delete old record, recreate with platform `_id`, update all related collections.

---

## 13. Known Limitations & Future Work

### Current Limitations
- JWT uses HS256 (symmetric) — all products share the same secret
- Entitlement check is wired but no product enforces premium gating yet
- BTCPay API key has insufficient permissions (Lightning payments non-functional)
- Stripe bundle price IDs (IL All Access, MBS All Access) not yet created in Dashboard
- GDPR category-level and cross-product deletion not yet wired (endpoints exist, orchestration doesn't)
- CWG running on `test` branch (intentional)

### Planned Improvements
- Premium feature gating per product (free tier limits)
- Post-signup module picker on innerlab.ai
- JWT upgrade to RS256 asymmetric signing
- Friends consolidation (product-level to platform-level)
- Admin dashboard with analytics and revenue tracking
- Free trial support with auto-conversion
- Multi-currency support, family plans, push notifications

---

## Appendix A: Per-Product Deployment Reference

Deployment details for every product in the ecosystem, sourced from Phase 5 reports and live verification (March 28, 2026).

### Platform (Layer 1 + 2)

| Product | Domain (Frontend) | Domain (Backend) | Containers | Stack | Database | Quirks |
|---------|------------------|-------------------|------------|-------|----------|--------|
| MBS Platform | magicbusstudios.com | api.magicbusstudios.com | 2 | Express + React (Vite) | mbs_platform | — |
| Inner Lab | innerlab.ai | api.innerlab.ai | 2 | Express + React (Vite) | inner_lab | Backend Dockerfile is `Dockerfile.server` (not `Dockerfile`) |
| CWG | conversationswithgod.ai | api.conversationswithgod.ai | 2 | FastAPI + React (Vite) | inner_lab (cwg_*) | Python app — JWT_SECRET_KEY vs JWT_SECRET fallback needed. Running on `test` branch. |
| FlowState | yoga.magicbusstudios.com | api.yoga.magicbusstudios.com | 2 | Express + React (Vite) | inner_lab (yoga_*) | — |

### The Arcade (Layer 3)

| Product | Domain (Frontend) | Domain (Backend) | Containers | Stack | Database | Quirks |
|---------|------------------|-------------------|------------|-------|----------|--------|
| BrokenChain | brokenchain.magicbusstudios.com | api.brokenchain.magicbusstudios.com | 2 | Express + React (Vite) | brokenchain | WebSocket (Socket.io) — must keep WS support enabled in Traefik |
| MindHacker | mindhacker.magicbusstudios.com | api.mindhacker.magicbusstudios.com | 2 | Express + React (Vite) | mindhacker | Traefik strips `/api` prefix — routes mounted without it. Backend domain uses dots not hyphens. Frontend healthcheck must be DISABLED in Coolify. |
| Trivia Roast | triviaroast.magicbusstudios.com | api.trivia.magicbusstudios.com | 2 | Express + vanilla HTML/JS | triviaroast | Coolify services on SEPARATE Docker networks — nginx proxy_pass uses public domain, not internal hostname. `authSource=admin` required in MONGO_URL. |
| Fake Artist | fakeartist.magicbusstudios.com | api-fakeartist.magicbusstudios.com | 2 | Express + React (Vite) | fakeartist | Socket connections remain UNAUTHENTICATED (party game UX). Backend domain uses HYPHEN (api-fakeartist), not dot. |
| Whispering House | whisperinghouse.magicbusstudios.com | api-whisperinghouse.magicbusstudios.com | 2 | FastAPI + React (Vite) | whispering_house | Python app. Auth only on create+join, not in-game. WebSocket stays unauthenticated. Backend domain uses HYPHEN. |

### Studio Works (Layer 3)

| Product | Domain (Frontend) | Domain (Backend) | Containers | Stack | Database | Quirks |
|---------|------------------|-------------------|------------|-------|----------|--------|
| WildLens | wildlens.magicbusstudios.com | api.wildlens.magicbusstudios.com | 2 | Express + Next.js | wildlens | Next.js (not Vite) — uses `NEXT_PUBLIC_*` build args, not `VITE_*`. Full legacy user migration across 10+ collections. |
| Lazy Chef | lazy-chef.magicbusstudios.com | lazy-chef-backend.magicbusstudios.com | 2 | FastAPI + React (Vite) | lazy_chef | Python app. Backend domain convention differs (lazy-chef-backend, not api-lazy-chef). CI uses GitHub Actions. |
| Movie Picker | moviepicker.magicbusstudios.com | (check Coolify) | 2 | Express + React (Vite) | moviepicker | Uses TMDB API for movie data. resolveUser middleware with in-memory Set cache for legacy migration. |
| SmartCart | smartcart.magicbusstudios.com | (same container) | **1** | Express serves frontend + API | smartcart | SINGLE CONTAINER — Express builds Vite frontend and serves from `/public`. No separate frontend service. Dockerfile has `ARG` for VITE_* vars. |
| TaskTracker | tasktracker.magicbusstudios.com | (same service) | 1 (Nixpacks) | Express + React (CRA) | tasktracker | Uses Nixpacks (not Dockerfile). `CI=true` treats ESLint warnings as errors. React app uses `REACT_APP_*` (not `VITE_*`). Legacy migration uses MongoDB TRANSACTIONS — requires replica set, fails on standalone. |
| AI Tutor | tutor.magicbusstudios.com | api.tutor.magicbusstudios.com | 2 | FastAPI + React (Vite) | ai_tutor | Python app. Both SSO bugs confirmed here (JWT_SECRET naming + legacy user collision). Onboarding flow kept for new users. |

### Deployment Learnings (Phase 5)

1. **Coolify env vars must be on separate lines** — concatenated vars (e.g., `JWT_SECRET=xxxLOG_LEVEL=INFO`) cause silent JWT validation failures.
2. **`authSource=admin` required** — when MongoDB uses `root` credentials, the connection string must include `authSource=admin` or auth fails silently.
3. **Coolify services are on separate Docker networks** — internal Docker hostnames (e.g., `backend:3001`) don't resolve between services. Use public domains for inter-service communication.
4. **Frontend healthcheck may need disabling** — Coolify's probe can't reach nginx in some configurations and kills the container.
5. **Backend domain conventions vary** — some use `api.X` (dots), some use `api-X` (hyphens), one uses `X-backend`. Check Coolify config per service.
6. **Traefik may strip `/api` prefix** — if routes return 404, check whether Traefik is stripping the prefix. Mount routes without `/api` if so.
7. **`VITE_BACKEND_URL` must NOT include `/api`** — if Traefik strips it, the URL should be just the domain.
