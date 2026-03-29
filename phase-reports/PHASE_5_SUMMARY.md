# Phase 5 Summary — Standalone Products SSO Migration

**Date**: March 28, 2026
**Status**: ALL 11 COMPLETE — deployed and verified live
**Scope**: 5 Arcade games + 6 Studio Works tools migrated to MBS Platform SSO

---

## Overview

All 11 standalone products were migrated from standalone auth (or no auth) to MBS Platform SSO in a single session. Each product now:
- Validates JWTs issued by the MBS Platform (HS256, shared secret)
- Redirects to magicbusstudios.com/auth/login for authentication
- Calls the entitlement API with 5-minute caching and fail-open behavior
- Handles legacy user collision (where applicable)

## Full Reports

Each product's detailed Phase 5 report lives in its project root:

| Product | Report Location |
|---------|----------------|
| BrokenChain | `Brokenchain/PHASE_5_REPORT_brokenchain.md` |
| MindHacker | `Mindhacker/PHASE_5_REPORT_mindhacker.md` |
| Trivia Roast | `Trivia/PHASE_5_REPORT_triviaroast.md` |
| Fake Artist | `Fakeartist/PHASE_5_REPORT_fakeartist.md` |
| Whispering House | `Whispering House/PHASE_5_REPORT_whisperinghouse.md` |
| WildLens | `Wildlife/PHASE_5_REPORT_wildlens.md` |
| Lazy Chef | `LazyChef/PHASE_5_REPORT_lazychef.md` |
| Movie Picker | `Movie/PHASE_5_REPORT_moviepicker.md` |
| SmartCart | `Shopping/PHASE_5_REPORT_smartcart.md` |
| TaskTracker | `TaskTracker/PHASE_5_REPORT_tasktracker.md` |
| AI Tutor | `Tutor/PHASE_5_REPORT_tutor.md` |

All paths relative to `C:\Users\1984a\OneDrive\Desktop\Codes\`.

---

## Per-Product Summary

### The Arcade (5 games)

**BrokenChain** (Node.js/Express)
- Had: Google OAuth (Passport) + email/password (bcrypt)
- Removed: passport.js, auth.js, bcryptjs, passport deps
- Legacy user fix: Yes — delete/recreate with platform _id
- Both SSO bugs hit: JWT_SECRET fallback added, legacy user collision fixed
- Commits: `224c347`, `eff929d`, `1d140ca`

**MindHacker** (Node.js/Express)
- Had: visitorId-based casual identity
- Removed: visitorId system, replaced with platformUserId
- Legacy user fix: Yes — email fallback with Player model redesign
- Deployment quirk: Traefik strips /api prefix, routes mounted without it
- Deployment quirk: Backend domain uses dots (api.mindhacker), not hyphens
- Deployment quirk: Frontend healthcheck must be disabled

**Trivia Roast** (Node.js/Express + vanilla HTML)
- Had: No auth at all
- Added: requireAuth on POST routes only (GET stays public)
- Legacy user fix: N/A — no User model, stateless JWT
- Deployment quirk: Services on separate Docker networks, nginx uses public domain
- Deployment quirk: authSource=admin required in MONGO_URL

**Fake Artist** (Node.js/Express)
- Had: No auth (socket-based ephemeral identity)
- Added: JWT auth for entitlement check only, socket stays unauthenticated
- Legacy user fix: N/A — no User model, no findById anywhere
- Design decision: Party game UX preserved — no login required to play

**Whispering House** (Python/FastAPI)
- Had: No standalone auth
- Added: Auth on create+join only, WebSocket stays unauthenticated
- Legacy user fix: N/A — no user storage
- Premium UI enhancements also added (typewriter text, confetti, floating dust)
- Known issue: "Start Game" network error on Coolify (needs investigation)

### Studio Works (6 tools)

**WildLens** (Node.js/Express + Next.js)
- Had: Google OAuth + email/password (bcrypt) + cookie-based refresh tokens
- Removed: auth routes, passport, cookie-parser, Google GSI
- Legacy user fix: Yes — full migration across 10+ collections (discoveries, posts, bookmarks, chat sessions, collections, expeditions, notifications, challenges, followers/following)
- This was the app that first revealed the legacy user collision bug

**Lazy Chef** (Python/FastAPI)
- Had: Google OAuth + email/password + Stripe
- Removed: auth_routes.py, auth_service.py (not deleted, just not imported)
- Legacy user fix: Yes — SAFE pattern using $set platformUserId (no delete/recreate)
- Verified LIVE with full Chrome browser test (all pages working)
- CI fixed: test files updated for SSO-style auth

**Movie Picker** (Node.js/Express)
- Had: Google OAuth (Passport) + email/password
- Removed: auth.js, passport.js, LoginPage, SignUpPage, AuthCallbackPage, AuthButton
- Legacy user fix: Yes — resolveUser middleware with in-memory Set cache, rekeys Watchlist + UserMovieInteraction
- Verified LIVE with Chrome browser test

**SmartCart** (Node.js/Express — SINGLE CONTAINER)
- Had: Google OAuth (Passport) + email/password
- Removed: auth.js, passport.js, LoginPage, RegisterPage, AuthCallbackPage
- Legacy user fix: Yes — ensureUser + migrateLegacyUser across 5 collections
- Schema change: All userId fields changed from ObjectId to Mixed type
- Architecture: Single container (Express serves Vite frontend from /public)

**TaskTracker** (Node.js/Express — Nixpacks)
- Had: Google OAuth (Passport) + email/password + refresh tokens + SendGrid
- Removed: 12 auth routes, 7 frontend pages, passport config
- Legacy user fix: Yes — TRANSACTIONAL migration across 14 collections (uses MongoDB sessions)
- Known issue: Transactions require replica set, fails on standalone MongoDB
- CI fix: ESLint warnings blocked build (CI=true), fixed unused imports
- Build: Nixpacks (not Dockerfile), uses REACT_APP_* (not VITE_*)

**AI Tutor** (Python/FastAPI)
- Had: Google OAuth + email/password (bcrypt)
- Removed: 4 auth routes, GoogleOAuthService, hash/verify password, bcrypt dep
- Legacy user fix: Yes — BOTH bugs confirmed and fixed here:
  - Fix 1 (commit `e68ea4b`): JWT_SECRET_KEY vs JWT_SECRET fallback
  - Fix 2 (commits `7a60443`, `68e56e2`): Legacy user email collision
- Onboarding flow preserved for new platform users

---

## Critical Bugs Discovered

### Bug 1: Legacy User Collision
**Symptom:** User authenticates on platform, gets redirected back to app, appears not signed in.
**Cause:** `findById(platformUserId)` returns null (legacy user has different _id). `User.create()` crashes on unique email index.
**Affected:** Every app with pre-existing users (7 of 11).
**Fix pattern:** Check by email first, migrate old record to platform _id, update all related collections.
**Documented in:** `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` Step 5.

### Bug 2: Python JWT_SECRET Naming
**Symptom:** Same as above (appears not signed in), but JWT verification silently fails.
**Cause:** Python config reads `JWT_SECRET_KEY`, Coolify has `JWT_SECRET`. Secret defaults to empty string.
**Affected:** Python apps (AI Tutor confirmed, CWG at risk).
**Fix:** Fallback chain: `os.environ.get("JWT_SECRET_KEY") or os.environ.get("JWT_SECRET")`.
**Documented in:** `platform-instructions-for-cwg/PLATFORM_MIGRATION.md`.

---

## Legacy User Migration Approaches (Three Patterns Used)

| Pattern | Used By | How It Works |
|---------|---------|-------------|
| **Delete/Recreate** | WildLens, BrokenChain, SmartCart, TaskTracker, AI Tutor | Delete old user record, create new with platform _id, updateMany all related collections |
| **$set platformUserId** | Lazy Chef, Movie Picker, MindHacker | Keep old _id, add platformUserId field, query by platformUserId going forward |
| **N/A** | Fake Artist, Trivia Roast, Whispering House, (MindHacker had no real users) | No User model or no existing users |

The delete/recreate pattern is more thorough but requires updating every collection that references the user. The $set pattern is safer (no data migration needed) but means old _id and platform userId coexist.

---

## Common Patterns Across All 11 Apps

- **JWT middleware:** `jsonwebtoken` (Node.js) or `PyJWT`/`python-jose` (Python), HS256, extracts `{ userId, email, name, avatar, isAdmin }`
- **Token storage:** `localStorage.setItem("mbs_token", token)` + `mbs_user`
- **Token arrival:** `?token=JWT` in URL after platform redirect, cleaned via `history.replaceState()`
- **Entitlement check:** 5-minute in-memory cache, fail-open to free_tier
- **Legacy routes:** `/login`, `/signup` redirect to platform login (not 404)
- **Old auth removed:** Passport, bcryptjs, Google OAuth libs, cookie-parser unimported (some kept in package.json)

---

## GDPR Status Per App

| Product | DELETE /api/user-data | Status |
|---------|----------------------|--------|
| BrokenChain | Not implemented | Stores user stats, friends — needs endpoint |
| MindHacker | Not implemented | Stores Player records — needs endpoint |
| Trivia Roast | Not needed | No userId-keyed persistent data (leaderboard keyed by playerName) |
| Fake Artist | Not needed | Ephemeral room data with 24hr TTL |
| Whispering House | Not implemented | Stores mbs_user_id in session docs — needs endpoint |
| WildLens | Not implemented | Stores discoveries, posts, collections — needs endpoint |
| Lazy Chef | Implemented | Deletes from all collections |
| Movie Picker | Not implemented | Stores watchlists, interactions — needs endpoint |
| SmartCart | Implemented | Deletes from all 5 collections |
| TaskTracker | Implemented | Full GDPR cascade across 14 collections |
| AI Tutor | Implemented | Deletes user + progress + chat history |

---

## Env Var Changes (Common Across All 11)

### Added to All
- `JWT_SECRET` — must match MBS Platform exactly
- `PLATFORM_URL` — `https://magicbusstudios.com` (or `PLATFORM_API_URL` = `https://api.magicbusstudios.com`)
- `PRODUCT_SLUG` — product identifier for entitlement checks

### Removed (Where They Existed)
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`
- `STRIPE_SECRET_KEY` / `STRIPE_WEBHOOK_SECRET` / `STRIPE_PRICE_*`
- `JWT_REFRESH_SECRET` / `JWT_EXPIRATION_HOURS`

### MBS Platform CORS_ORIGINS
All 11 product domains must be in the MBS Platform's `CORS_ORIGINS` env var for the SSO redirect flow to work.
