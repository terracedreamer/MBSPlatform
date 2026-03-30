# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 30, 2026

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

### Moved to Phase 1 Addendum (no longer future work)
All of these are now in `platform-instructions-for-mbs/CLAUDE.md` under "Phase 1 Addendum":
- ~~Login button~~ → Phase 1 Addendum #1
- ~~Rate limiting~~ → Phase 1 Addendum #2
- ~~Lightning billing UI~~ → Phase 1 Addendum #3
- ~~Nostr auth~~ → Phase 1 Addendum #4
- ~~LNURL-Auth~~ → Phase 1 Addendum #5
- ~~Auth method linking~~ → Phase 1 Addendum #6
- ~~Email preferences~~ → Phase 1 Addendum #7
- ~~Transactional emails~~ → Phase 1 Addendum #8
- ~~Admin panel~~ → Phase 1 Addendum #9
- ~~Promo codes~~ → Phase 1 Addendum #10
- ~~Referrals~~ → Phase 1 Addendum #11
- ~~Token refresh~~ → Phase 1 Addendum #12

### Reminders (Owner Action Required)
- [ ] **Create Stripe products in Dashboard** — 6 products, 12 prices. Full table in SESSION_9_PENDING_ITEMS.md.
- [ ] **Regenerate BTCPay API key** — Current key has insufficient permissions (403). Lightning payments don't work until fixed.

### Completed — Session 10 (12 Enhancements — commit `0ba7114`)
- [x] #17 ADMIN_EMAILS env var — `isEnvAdmin()` checks env before DB in requireAdmin + issueToken
- [x] #19 Response helpers adoption — all 14 route files (~316 conversions)
- [x] #13 Activity log for subscription events (subscribe-free, checkout, cancel, expire, upgrade, trial expire)
- [x] #20 AdminPage splitting (2067 -> 767 lines + 6 tab components + 8 reusable admin components, 11 new files)
- [x] #14 User segmentation — GET /api/admin/users/segments + filter chips (new/trial/paid/free/churned/inactive)
- [x] #15 Revenue analytics — GET /api/admin/analytics/revenue + Analytics tab (MRR, ARR, ARPU, churn, growth, trend)
- [x] #11 Connected Accounts UI — Google, Email, Nostr, LNURL status + link Nostr form
- [x] #18 GDPR delete confirmation — "Type DELETE" on all 3 delete modals
- [x] #10 Onboarding flow — 3-step wizard (welcome -> pick product -> success)
- [x] #12 Friends system enhancement — avatar grid, remove friend, shared products endpoint
- [x] #4 Referral email invites — SendGrid HTML template, 10/day rate limit
- [x] #5 Promo code checkout flow — validate -> Stripe coupon (percentage/fixed/trial_extension) -> apply to session + UI

### Deferred to Future Session
- [ ] #16 JWT upgrade to RS256 (multi-repo, 2-3 days)
- [ ] #21 Test coverage expansion (frontend + billing + entitlements)

### Completed — Session 10 continued (commits `c374a94`, `6150872`, `2278e1d`)
- [x] #3 Real-time entitlement sync after Stripe checkout — `refreshUser()` on BillingPage `?success=true` redirect
- [x] Premium feature gating — `premiumFeatures` array on all 22 products, `requireEntitlement(slug)` + `requirePremium` middleware, enhanced entitlements response
- [x] Invoice PDF generation — PDFKit, `server/services/invoiceService.js`, `GET /api/billing/invoice/:transactionId`, download button on BillingPage
- [x] Fix: AdminUsers segment filter chips — segments object converted to array (`c374a94`)
- [x] Fix: admin tab content initial={false} for ProtectedRoute compat (`6150872`)

### Session 11 Potential Items
- [ ] **#16 RS256 JWT upgrade** (deferred from Session 10) — plan written and approved at `.claude/plans/humble-sparking-wigderson.md`, ready to execute
- [ ] #21 Test coverage expansion (deferred from Session 10)

### Post-Phase 5 Cleanup
- [x] **Dead code cleanup across Phase 5 apps** — Audited all 7 apps. Only bcryptjs in Wildlife was dead (removed). All others clean.
- [x] **GDPR `DELETE /api/user-data` endpoint** — All 8 apps (including Whispering House). Committed, pushed, deployed to Coolify.
- [x] **MBS Platform deletion UI + cascade service** — Deployed. AccountPage has Data Management section.
- [x] **CWG entitlements** — check_entitlement() wired to profile endpoint (Session 8, `f9c38ab` on test branch).
- [x] **TaskTracker MongoDB transactions** — Non-transactional fallback added (Session 8, `8053c11`).

### Decided — Ready to Build (Session 7 Decisions, 2026-03-29)
- [x] **Three-tier subscription model** — Built in Session 8 (`ca7fac6`). checkAccess() refactored to require explicit subscription.
- [x] **Subscribe gating for all apps** — ProductPickerPage at /products with subscribe-free endpoint.
- [x] **CWG entitlement enforcement** — Wired in Session 8 (`f9c38ab` on test branch).
- [x] **Product/module picker (all products)** — ProductPickerPage covers all 22 products with category tabs.
- [x] **Consolidate friends to MBS Platform level (Option A)** — No work needed. MBS already has friends API, CWG has no friends code.
- [x] **Admin dashboard (hierarchical)** — AdminPage at /admin with overview, users, entitlements tabs.
- [x] **Free trial (7 days premium)** — Bundled with subscribe-free. Lazy downgrade after trial expires.

### Completed — 12 Enhancements (Session 8, MBS `d52eaba` + `e2a0364`)

**Quick wins (1-5):**
- [x] **Backfill auth_provider on legacy users** — 9 google + 10 email via MongoDB terminal
- [x] **ProtectedRoute component** — wraps all protected routes, loading overlay, admin guard
- [x] **Auth Context (React Context API)** — centralized user/token/admin state, cross-tab sync
- [x] **User profile editing** — PUT /api/auth/profile + Edit button on AccountPage
- [x] **Email verification** — already existed, verified working

**Medium features (6-10):**
- [x] **Notification center** — bell icon, announcements API, glass dropdown, dismiss tracking
- [x] **Feature flags system** — admin CRUD + public check-flag + Flags tab on AdminPage
- [x] **Activity feed on admin dashboard** — Activity tab, paginated ActivityLog, color-coded badges
- [x] **Product analytics** — per-product subs, trial conversion, top products on Overview tab
- [x] **Onboarding flow** — "Welcome to MagicBusStudios!" modal for users with 0 entitlements

**Larger features (14-15):**
- [x] **Push notifications** — web-push + service worker + AccountPage toggle + admin send modal. Needs VAPID keys in env vars.
- [x] **Real-time admin dashboard** — Socket.io /admin namespace, "Live" badge, toast events for signups/entitlements

### Future Work (Decided but Deferred)
- [ ] **JWT upgrade to RS256 asymmetric signing** — All products share same HS256 secret; leaked key = forge tokens for all apps. Important before real users/launch. Move to RS256: MBS Platform holds private key (signing), all products hold public key (verification only).
- [x] **Admin accounts via ADMIN_EMAILS env var** — Implemented in Session 10 (`0ba7114`). `isEnvAdmin()` checks `ADMIN_EMAILS` env var before DB `is_admin` field in both `requireAdmin` and `issueToken`.
- [x] **Premium feature gating (per-product)** — Middleware created (`requireEntitlement`, `requirePremium` in `server/middleware/entitlement.js`), `premiumFeatures` defined for all 22 products in `products.js`. Infrastructure ready; per-product route enforcement can be applied as needed.
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
- [ ] **`req.user` shape `{ userId, email, name, avatar }`**: PARTIAL. The JWT payload (via `issueToken` in `middleware/auth.js`) correctly encodes `{ userId, email, name, avatar, isAdmin }` and `requireAuth` sets `req.user = decoded` which is correct. However, **7 auth route responses** return `user: { id: existingUser._id, ... }` instead of `user: { userId: ... }` — lines 98, 144, 187, 436, 598, 733, 826 in `server/routes/auth.js`. The `id` key in the response body does not match the `userId` convention.
- [ ] **Winston logger (never `console.log`)**: NOT COMPLIANT. 155 `console.log/warn/error` calls across all 15 server files. Only `server/services/gdprCascade.js` imports a logger (`require("../utils/logger")`), but `server/utils/logger.js` does not exist — this import will crash at runtime if the GDPR cascade is invoked. No Winston setup anywhere.
- [ ] **Centralized error handler middleware**: NOT COMPLIANT. No `app.use((err, req, res, next) => ...)` in `server/index.js`. Each route does its own try/catch with inline `res.status(500).json(...)`. No centralized error-handling middleware registered.
- [x] **Response helpers (`sendSuccess`, `sendError`, `sendNotFound`)**: COMPLIANT (Session 10). All 14 route files converted to use response helpers (~316 conversions). Created in Session 8, fully adopted in Session 10 (`0ba7114`).

### Frontend Standards (MBS/ src/)

- [x] **No `alert()`/`prompt()`/`confirm()`**: Compliant. Zero occurrences in `src/`.
- [x] **Sonner toasts (never react-hot-toast)**: Compliant. `sonner` imported in `main.jsx` (Toaster component) and across 6 page/component files. No `react-hot-toast` in codebase or `package.json`.
- [x] **Fonts: Space Grotesk (headings), DM Sans (body)**: Compliant. Both declared as CSS custom properties in `src/index.css` (`--font-heading`, `--font-body`). Font packages imported in `main.jsx` (`@fontsource-variable/space-grotesk`, `@fontsource-variable/dm-sans`). Also includes Instrument Serif (accent) and JetBrains Mono (mono) per standard.
- [x] **Safe array mapping `(array || []).map()`**: Mostly compliant. 7 instances of `(x || []).map()` in `AccountPage.jsx`, `BillingPage.jsx`, `Games.jsx`, `OtherWork.jsx`, `RoadmapTimeline.jsx`, `Home.jsx`. Other `.map()` calls use arrays initialized as `useState([])` or hardcoded arrays (safe). No unguarded dynamic data `.map()` found.
