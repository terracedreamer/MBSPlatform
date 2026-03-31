# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 31, 2026

**RULE: If an item depends on a specific phase, it goes INTO that phase's platform-instructions document — NOT here. This file is ONLY for items that are either phase-independent or span multiple phases.**

---

## Phase Completion Tracker

| Phase | Status | Instructions Location | Items Tracked In |
|-------|--------|----------------------|-----------------|
| Phase 1: MBS Platform | ✅ DONE — all addendum items #1-15 deployed | `platform-instructions-for-mbs/CLAUDE.md` | That doc (addendum section) |
| Phase 2: IL Middleware + Auth | ✅ DONE — middleware + 4 auth pages live at innerlab.ai | `platform-instructions-for-innerlab/CLAUDE.md` | That doc |
| Phase 3: CWG Migration | ✅ DONE — Phase 3A (data) + Phase 3B (refactor) complete, running on `test` | `platform-instructions-for-cwg/PLATFORM_MIGRATION.md` | That doc |
| Phase 4: FlowState Migration | ✅ DONE — live on production (dev + main), entitlements wired | `platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md` | That doc |
| Phase 5: Standalone Products | ✅ DONE — all 11 deployed and verified live (2026-03-28) | `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` | That doc + 11 PHASE_5_REPORT files |

---

## Cross-Phase Dependencies (CRITICAL — review before starting each phase)

| Item | Depends On | Blocks | Notes |
|------|-----------|--------|-------|
| CWG migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 3 | Script writes to both mbs_platform AND inner_lab |
| FlowState migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 4 | Same pattern as CWG |
| Stripe product/price creation | Pricing decision (no phase dependency) | Real payments in any phase | Backend ready, needs Stripe Dashboard setup |
| GDPR cascade delete testing | Phase 1 (delete endpoint) + Phase 2 (il_* data) | None (works now, test after Phase 2) | Phase 1 built the cascade, Phase 2 creates the data it deletes |

---

## Pricing Decision — PARTIALLY RESOLVED (moved to Phase 1 Addendum #13)

**Status: CWG pricing decided. Bundles decided. Arcade/SW/FlowState deferred.**

- [x] CWG pricing: $9.99/mo, $79.99/yr (Stripe) + 21K sats/mo, 126K sats/yr (Lightning)
- [x] Inner Lab All Access: $19.99/mo, $159.99/yr
- [x] MBS All Access: $29.99/mo, $249.99/yr
- [x] Structured billing page design (category tabs → product → plans)
- [x] Lightning as equal payment option alongside Stripe
- [x] FlowState pricing — $5/mo, $45/yr (Session 9)
- [x] Arcade pricing — $10/mo bundle, $5/mo individual (Session 9)
- [x] Studio Works pricing — $10/mo bundle, $5/mo individual (Session 9)
- [ ] Create actual Stripe products in Dashboard (OWNER ACTION — see SESSION_9_PENDING_ITEMS.md for exact prices)

**CWG already has Stripe price IDs** in its Coolify env vars. The Phase 1 agent can either reuse those or create new platform-level ones.

---

## Phase-Independent Future Work

These items don't belong to any specific phase and can be done anytime:

### Marketing / Content
- [ ] Convert CWG marketing plan .docx in Desktop/Marketing/ to .md (old content, may delete)

### Moved to Phase 1 Addendum
12 items (login, rate limiting, Lightning, Nostr, LNURL, auth linking, email prefs, transactional emails, admin panel, promo codes, referrals, token refresh) — all completed and deployed. See `platform-instructions-for-mbs/CLAUDE.md` addendum section.

### Reminders (Owner Action Required)
- [ ] **Create Stripe products in Dashboard** — 6 products, 12 prices:

| Product | Monthly | Annual | Env Var (Monthly) | Env Var (Annual) |
|---------|---------|--------|-------------------|------------------|
| FlowState | $5 | $45 | `STRIPE_YOGA_MONTHLY_PRICE_ID` | `STRIPE_YOGA_ANNUAL_PRICE_ID` |
| CWG | $15 | $135 | `STRIPE_CWG_MONTHLY_PRICE_ID` | `STRIPE_CWG_ANNUAL_PRICE_ID` |
| Studio Works All Access | $10 | $90 | `STRIPE_SW_MONTHLY_PRICE_ID` | `STRIPE_SW_ANNUAL_PRICE_ID` |
| Arcade All Access | $10 | $90 | `STRIPE_ARCADE_MONTHLY_PRICE_ID` | `STRIPE_ARCADE_ANNUAL_PRICE_ID` |
| Inner Lab All Access | $20 | $180 | `STRIPE_IL_MONTHLY_PRICE_ID` | `STRIPE_IL_ANNUAL_PRICE_ID` |
| MBS All Access | $30 | $270 | `STRIPE_MBS_MONTHLY_PRICE_ID` | `STRIPE_MBS_ANNUAL_PRICE_ID` |

Steps: Create in Stripe Dashboard (test mode) → copy 12 price IDs → add to Coolify MBS B env vars → redeploy.

- [ ] **Regenerate BTCPay API key** — Current key has insufficient permissions (403). Lightning payments don't work until fixed.

### Completed — Session 10
12 enhancements (ADMIN_EMAILS, response helpers, activity logging, connected accounts, GDPR type-DELETE, onboarding, friends, AdminPage split, user segmentation, revenue analytics, referral emails, promo checkout) + 3 billing features (entitlement sync, premium gating, invoice PDFs). See CHANGELOG.md Session 10 entries.

### Deferred to Future Session
- [x] #16 JWT upgrade to RS256 — **COMPLETED Session 11**. All 15 repos upgraded, committed, pushed. Env vars pending in Coolify.
- [ ] #21 Test coverage expansion (frontend + billing + entitlements)


### Completed — Session 11 (RS256 JWT Upgrade)
- [x] **#16 RS256 JWT upgrade** — All 15 repos: MBS Platform signs RS256, all child apps verify RS256→HS256 dual-mode. RSA-2048 key pair generated. `GET /api/auth/public-key` endpoint added. Pending: Coolify env vars (`JWT_PRIVATE_KEY` on MBS B, `JWT_PUBLIC_KEY` on all 15).

### Session 12 Potential Items
- [ ] #21 Test coverage expansion (deferred from Sessions 10-11)
- [ ] RS256 Phase 2 cleanup — remove HS256 fallback after 7 days, remove `JWT_SECRET` from child apps
- [ ] LazyChef — remove `create_jwt_token` self-issued auth

### Post-Phase 5 Cleanup — All Complete
Dead code cleanup, GDPR endpoints (all 8 apps), MBS deletion UI + cascade, CWG entitlements, TaskTracker transactions. See CHANGELOG.md Session 8.

### Decided & Built — Session 8
Subscribe gating, product picker, CWG entitlements, friends consolidation, admin dashboard, free trial (7 days). All built and deployed. See CHANGELOG.md Session 8.

### Completed — Session 8 Enhancements
AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin. See CHANGELOG.md Session 8.

### Future Work (Decided but Deferred)
- [ ] Win-back offers — needs email + promo system working together
- [ ] User dashboard (My Products, billing history) — needs pricing + real subscriptions
- [ ] Email campaigns + announcements + newsletter — post-launch
- [ ] Multi-currency, family plan, teams — long-term

---

## Standards Compliance (Global CLAUDE.md Audit)

> Reference: Check global CLAUDE.md and ~/.claude/rules/ for full standards.
> Audited against MBS/ codebase (Layer 1 — magicbusstudios.com) on 2026-03-29.

### Architecture Docs & Rules Files

- [x] **Verify Architecture Docs**: Fixed Movie Picker (single container, not 2), updated GDPR limitation note for Session 7 cascade service, bumped version to 1.1 (2026-03-29)
- [x] **Verify GDPR Status Table**: Updated data-sovereignty.md and CLAUDE.md — 6 apps moved from "Missing" to "Built (pending deploy)", Whispering House still missing
- [x] **Verify Deployment Quirks**: Added Movie Picker (single container, TMDB API) and AI Tutor (Python/FastAPI, JWT fallback chain) entries. Fixed Lazy Chef DB name in env-standards.md (`lazychef` → `lazy_chef`)

### Backend Standards (MBS/ server/)

- [x] **CORS before helmet()**: Compliant. `server/index.js` lines 48-51: `app.use(cors(...))` then `app.use(helmet())`.
- [x] **Response format `{ success: true/false, ... }`**: Compliant. All 10 route files and index.js health endpoints use `{ success: true, ...data }` or `{ success: false, message: "..." }`. 217 `success:` occurrences across 232 `res.json`/`res.status` calls — no response missing the `success` field.
- [x] **Rate limiting on API routes**: Compliant. Three tiers: general (100/15min), auth (20/15min), billing (30/15min) in `server/index.js`.
- [x] **Input validation on routes**: Compliant. Auth routes validate required fields, password length, terms acceptance. Billing validates priceId. Promotions validates code. Friends validates invite code format.
- [x] **Route pattern (authenticate -> validate -> business logic -> respond)**: Compliant. All protected routes use `requireAuth` middleware, admin routes stack `requireAuth` + `requireAdmin`, then validate, then business logic.
- [x] **Env vars: `CORS_ORIGINS` (canonical)**: Compliant. Uses `process.env.CORS_ORIGINS` in `server/index.js`. No legacy `CORS_ORIGIN`/`CLIENT_URL`/`ALLOWED_ORIGINS`.
- [x] **Env vars: `JWT_SECRET` (canonical)**: Compliant. `server/middleware/auth.js` uses `process.env.JWT_SECRET`. No `JWT_SECRET_KEY` or `SECRET_KEY`.
- [x] **Env vars: `DB_NAME` (canonical)**: Compliant. `server/config/database.js` reads `process.env.DB_NAME` with default `mbs_platform`.
- [x] **Env vars: `PORT` (canonical)**: Compliant. `server/index.js` line 9: `process.env.PORT || 3001`.
- [x] **Env var: `MONGO_URL` (canonical with fallback)**: Compliant. `server/config/database.js` reads `MONGO_URL || MONGODB_URI` — canonical first, legacy fallback acceptable per env-standards.md.
- [x] **Health check endpoints**: Compliant. Both `/health` and `/api/health` return `{ success: true, service: "mbs-platform", database: bool, timestamp: ISO }`.
- [x] **Git branch**: Compliant. On `main` (correct for MBS — single-branch workflow per project CLAUDE.md).
- [x] **`req.user` shape `{ userId, email, name, avatar }`**: COMPLIANT (Session 8). JWT payload encodes `{ userId, email, name, avatar, isAdmin }` and `requireAuth` sets `req.user = decoded`. Auth route responses fixed from `id` to `userId` in Session 8.
- [x] **Winston logger (never `console.log`)**: COMPLIANT (Session 8). `server/utils/logger.js` created with Winston, all 155 `console.log` calls migrated to logger by audit agent. `SERVICE_NAME` env var supported.
- [x] **Centralized error handler middleware**: COMPLIANT (Session 8). `server/middleware/errorHandler.js` created and registered after all routes in `server/index.js`.
- [x] **Response helpers (`sendSuccess`, `sendError`, `sendNotFound`)**: COMPLIANT (Session 10). All 14 route files converted to use response helpers (~316 conversions). Created in Session 8, fully adopted in Session 10 (`0ba7114`).

### Frontend Standards (MBS/ src/)

- [x] **No `alert()`/`prompt()`/`confirm()`**: Compliant. Zero occurrences in `src/`.
- [x] **Sonner toasts (never react-hot-toast)**: Compliant. `sonner` imported in `main.jsx` (Toaster component) and across 6 page/component files. No `react-hot-toast` in codebase or `package.json`.
- [x] **Fonts: Space Grotesk (headings), DM Sans (body)**: Compliant. Both declared as CSS custom properties in `src/index.css` (`--font-heading`, `--font-body`). Font packages imported in `main.jsx` (`@fontsource-variable/space-grotesk`, `@fontsource-variable/dm-sans`). Also includes Instrument Serif (accent) and JetBrains Mono (mono) per standard.
- [x] **Safe array mapping `(array || []).map()`**: Mostly compliant. 7 instances of `(x || []).map()` in `AccountPage.jsx`, `BillingPage.jsx`, `Games.jsx`, `OtherWork.jsx`, `RoadmapTimeline.jsx`, `Home.jsx`. Other `.map()` calls use arrays initialized as `useState([])` or hardcoded arrays (safe). No unguarded dynamic data `.map()` found.
