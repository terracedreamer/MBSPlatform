# CHANGELOG — MBS Platform

## April 10, 2026 — Session 31: Free Tier Entitlements, Subscribe Pages, GDPR Email Confirmation

### MBS Platform (1 commit on main + development)
- **Session 31 full build** (`0fc98c4`) — Free tier entitlement type added. subscribe-free endpoint rewritten (single product + category+modules mode). Category entitlements return registered:true/false. Stripe webhook premium→free auto-downgrade. freeTierLimits removed from all 23 products. BillingPage→SubscribePage rename with /billing→/subscribe redirect. New SubscribeInnerLabPage with 12 IL module picker. DataDeletionRequest model + GDPR email confirmation flow (no immediate deletion). Account settings confirmation-sent UI. Coming Soon seed script for 5 apps. 4 OneDrive -MSI conflict files cleaned up.

### New Files
- `server/models/DataDeletionRequest.js` — GDPR deletion request tracking
- `server/routes/userData.js` — Deletion confirmation + category deletion endpoints
- `server/seeds/setComingSoon.js` — Seed 5 apps to coming_soon status
- `src/pages/SubscribePage.jsx` — Renamed from BillingPage
- `src/pages/SubscribeInnerLabPage.jsx` — IL module picker page

### Key Changes
- `Entitlement.js` — `free_tier` added to type enum
- `entitlements.js` — `checkAccess()` returns isPremium, `subscribe-free` supports category+modules, `category/:cat` returns registered per product
- `billing.js` — Stripe webhook auto-creates free_tier on subscription cancellation
- `auth.js` — DELETE /api/auth/account now sends confirmation email (DataDeletionRequest + sendDeletionConfirmationEmail)
- `emailService.js` — New sendDeletionConfirmationEmail(), all /billing links → /subscribe
- `products.js` — freeTierLimits removed from all products
- `App.jsx` — /subscribe, /subscribe/innerlab, /subscribe/individual routes + /billing redirects
- `AccountPage.jsx` — Deletion confirmation-sent state

### Verification
- FlowState GDPR bug: NOT present — il_user_wellness_profiles correctly excluded from app-level deletion
- Frontend build passes clean (528ms)
- Pushed to both main and development branches

---

## April 10, 2026 — Session 30 (continued): Admin Fixes, Module Alignment Prompts, IL Agent Sync, Owner Decisions

### MBS Platform (1 commit on main)
- **Admin link fix + all categories on homepage** (`1b08acf`) — Fixed URL normalization bug (links without `https://` were relative). Homepage "What We're Building" now shows Inner Lab, Arcade, and Studio Works sections. Admin-created products appear on frontend. Per-category "Add to..." buttons in admin.

### MBSPlatform Architecture Repo (7 commits on main)
- **Module alignment prompts** (`51fb0fb`) — Created 3 alignment docs: INNERLAB_MODULE_ALIGNMENT.md (16-section prompt for 10 new modules), CWG_ALIGNMENT.md (visual + entitlement, feature removal first), FLOWSTATE_ALIGNMENT.md (CSS Modules decision, toast migration, GDPR singleton bug). Added PLATFORM_URL dual-use clarification to all three.
- **IL agent feedback incorporated** (`6f4161d`) — Fixed dashboardApi.js path, removed motion.js hedge, added CWG feature removal ordering, flagged FlowState il_user_wellness_profiles GDPR singleton bug.
- **Owner decisions applied** (`1699ac6`) — FlowState: keep CSS Modules, keep Inter font, push to dev. CWG: IL reference hex values sufficient.
- **Real free/premium gating** (`0ff58db`, `ea9481c`) — Removed effectivePremium from ALL three prompts. Agents must propose and implement real free/premium split. Owner adjusts later.

### Global Reference Files
- `~/.claude/reference/innerlab-module-alignment.md` — new, copied from MBSPlatform
- `~/.claude/CLAUDE.md` — 11→12 modules, registration includes admin dashboard, added module alignment reference
- `~/Desktop/CLAUDE.md` — same registration update
- `platform-instructions-for-new-modules/CLAUDE.md` — fixed entitlement response, redirect URL, PLATFORM_URL, added JWT_PUBLIC_KEY

### Cross-Agent Sync
- IL agent reviewed all 3 alignment prompts — confirmed in sync
- PLATFORM_URL dual-use pattern agreed: frontend=magicbusstudios.com (nginx proxy), backend=api.magicbusstudios.com (direct)
- FlowState GDPR bug identified: il_user_wellness_profiles (identity singleton) may be incorrectly deleted at app-level
- CWG ordering: feature removal must precede alignment

### Decisions
- **React Query removed from module prompt** — IL dashboard uses plain fetch via dashboardApi.js, modules should match
- **Module alignment prompt location** — lives in MBSPlatform repo + ~/.claude/reference/ (agents can access without user pasting)
- **CWG feature removal ordering** — must happen BEFORE visual alignment (7 features being stripped)

---

## April 8, 2026 — Session 31: Free Tier Architecture, Admin Product Management, Entitlement Instructions

### MBS Platform (1 commit on main)
- **Admin product management** (`8e4e516`) — Link editing, product creation/deletion, category grouping, unified status source (API overrides hardcoded modules.js)

### MBSPlatform Architecture Repo (1 commit on main)
- **IL entitlement instructions + architecture** (`4a543be`) — INNERLAB_ENTITLEMENT_INSTRUCTIONS.md, FREE_TIER_ARCHITECTURE.md major rewrite, dedicated /subscribe/innerlab spec, billing→subscribe rename, entitlement-integration.md global reference

### Decisions
- MBS passes NO limits (only hasAccess + isPremium) — each module defines its own
- Dedicated /subscribe/innerlab page with module picker (Option A)
- Premium cancellation → auto-downgrade to free_tier (not removal)
- GDPR email confirmation before any data deletion (DataDeletionRequest model)
- Billing → Subscribe rename (deferred)

---

## April 7-8, 2026 — Session 30: Frontend Tests, Feature Verification, Architecture Planning, Validation Audit

### MBS Platform (1 commit on main)
- **Frontend test expansion** (`8aa813c`) — 33 new tests: BillingPage (8), AdminPage (8), OnboardingModal (12), plus existing 19. Total: 52 tests across 6 suites. Cleaned up 3 Nitro conflict files.

### MBSPlatform Architecture Repo (3 commits on main)
- **Architecture plans + feature verification** (`7b42d1d`) — Centralized email digest 6-step plan, user dashboard spec, win-back/campaigns architecture, family/teams specs. Verified Daily Briefing, Weekly Review, Consciousness Full Mode all complete. DreamLens V3 resolved (Zod accepted). Backlog items removed.
- **Validation audit** (`4d3783e`) — Full 15-app audit: 14/14 rate limited, 12/14 validated. Gaps: Trivia Roast (3 POST routes), Tutor (2 raw JSON routes), Fake Artist (low risk).
- **CWG Dev verification** (`225c10a`) — devcwg.magicbusstudios.com healthy, SSO flow correct, MBS Platform API reachable.

### Verified
- **CWG Dev** — backend healthy (`mbs_platform` mode), 21 guides, SSO → innerlab.ai/auth/login → api.magicbusstudios.com
- **Inner Lab features** — Daily Briefing (complete), Weekly Consciousness Review (complete, tested), Consciousness Full Mode 16Q (complete)
- **MBS frontend tests** — 52/52 passing

### Decisions
- **DreamLens V3** — Zod accepted as valid for next-gen TypeScript builds. Express-validator remains standard for existing JS apps.
- **Validation gaps** — Trivia Roast and Tutor to be fixed in future session. Fake Artist low priority.

---

## April 7, 2026 — Session 29: Product Catalog Update, GDPR Cascade Test, Account Dropdown, Frontend Tests

### MBS Platform (3 commits on main)
- **Product catalog + SSO redirect improvements** (`ba72201`) — Bonds + LifeMap added to products.js, BreathArc removed (12 IL modules). Auth redirect auto-allows product catalog URLs + `*.magicbusstudios.com` subdomains. Onboarding modal suppressed during SSO redirect. Unified `performRedirect()`. MSI conflict files + billing-Nitro.js cleaned up.
- **Account dropdown** (`95c4ab6`) — "Account" nav button now shows dropdown with "My Account" + "Sign Out". No more scrolling to bottom of Account page.
- **Frontend test suite** (`12a260f`) — 19 tests (Vitest + React Testing Library + jsdom). AuthContext (8), AuthButton dropdown (5), ProtectedRoute (6). Setup with framer-motion mock, IntersectionObserver mock.

### MBSPlatform Architecture Repo (2 commits on main)
- **Module catalog docs** (`528daec`) — Updated CLAUDE.md, platform-instructions-for-mbs, platform-instructions-for-new-modules to 12 IL modules
- **BreathArc cleanup** (`0137cf0`) — Removed all BreathArc references from platform-instructions-for-innerlab (architecture diagram, collection prefixes, source_module examples, product table)

### Verified
- **GDPR cascade** — `DELETE /api/auth/account` → 200. `cascadeDeleteAll()` called all deployed apps. User deleted, JWT invalidated.
- **Account dropdown** — Chrome-verified live on magicbusstudios.com
- **MBS_PLATFORM_URL** — CWG prod correct (`api.magicbusstudios.com`), CWG dev fixed by owner

### Decisions
- **BreathArc removal confirmed** — owner decision, was provisional since Session 16
- **Duplicate user cleanup skipped** — `27chestnustst@gmail.com` left as-is per owner decision
- **Win-back offers, User dashboard, Email campaigns** — skipped, blocked on Stripe products + BTCPay key

---

## April 4, 2026 — Session 19: Migration Scripts, Birth Profile, Sharing Toggle, Bidirectional Sync, Standards

### CWG Migration Scripts Run
- Journal migration: 10 entries, all idempotent (already in il_reflections). Total: 11.
- Identity migration: 0 profiles/histories (no structured data exists yet).

### Inner Lab — New Features (6 commits on main)
- **Birth Profile dashboard UI** (`08ab184`) — `/birth-profile` page with Nominatim geocoding
- **Sharing toggle** (`2a4f20f`) — `il_sharing_preferences`, `/api/sharing`, `/sharing` page, GDPR updated to 14 collections
- **Dashboard nav** (`c31ebce`) — Birth Profile + Sharing Settings cards
- **Response helpers migration** (`295a962`) — all 14 route files use sendSuccess/sendError/sendNotFound/sendBadRequest
- **Test suites** (`0f3f896`) — 29 new tests (33 total). Auth, validation, response format, token edge cases.
- **FUTURE_WORK_TODO** (`6eb8f08`) — Session 19 items + rich Consciousness/My Story UI TODOs

### CWG — Bidirectional Sync (1 commit on test)
- **Read from il_* first** (`49803ba`) — profile GET, consciousness-type GET, and chat AI context now read identity data from il_* collections first, fall back to cwg_user_profiles

### FlowState — il_activity_feed Integration (1 commit on dev)
- **Activity feed writes** (`cc65a40`) — writes to il_activity_feed on session completion (yoga/breathwork/meditation). GDPR updated to delete il_activity_feed by source_module.
- **GDPR user_id verified** — NOT a bug. yoga_activity uses `userId` consistently in both writes and deletes.

### Codex Review
- Doc 28 Round 4: DreamLens DB architecture confirmed (DB_NAME=inner_lab correct for all IL modules)
- Architecture Summary for Codex added to doc 28
- CODEX_REVIEW_GUIDE.md created in MBSPlatform with ready-to-use prompt

### Documentation Updates
- `env-standards.md` — DB naming table restructured (IL modules vs standalone products)
- `innerlab-data.md` — il_* count updated to 14
- CWG Prod B DB_NAME discrepancy flagged: still has `conversations_with_god` (will fix during test→dev merge)

---

## April 2, 2026 — Session 15: Brand Docs + TaskTracker SSO Fix + Documentation Consolidation

### Brand Documentation Gap Analysis
- Created `FlowState_Product_Brief.md` (400+ lines) in `Desktop/Marketing/Brand Overview/`
- Added "Consciousness Brand Layer" to `MagicBusStudios_Brand_And_Company.md`
- Added "Inner Lab in the Consciousness Ecosystem" to `InnerLab_Product_Brief.md`
- Verified CWG 67-language count (marketing agent update was factually correct)

### Documentation Consolidation
- Marketing folder (`Desktop/Marketing/`) is now the **single source of truth** for all brand briefs and architecture docs
- Deleted `MBSPlatform/Architecture Docs/` (5 files) and `MBSPlatform/Brand Overview/` (8 files) — duplicates
- Updated `MBSPlatform/CLAUDE.md` to reference Marketing folder

### TaskTracker SSO Redirect Fix
- Root cause found: `VITE_PRODUCT_DOMAIN` Coolify build arg had `https://` prefix
- Code already prepends `https://` → redirect became `https://https://tasktracker.magicbusstudios.com`
- `validateRedirectUrl()` parsed hostname as `"https"` (not in CORS_ORIGINS) → returned safe default
- Fix: removed `https://` prefix from build arg. Also resolved 403 errors on all data routes.
- Chrome verified: all 6 API routes returning 200, dashboard fully functional, 8/8 standards PASS

---

## April 2, 2026 — Session 14: TaskTracker Vite + Architecture Docs

### TaskTracker CRA → Vite Migration
- Replaced react-scripts with vite@5. Renamed 70 `.js` → `.jsx`. Updated env vars to `VITE_*`.
- Committed (`0b75506`) and pushed to TaskTracker main.

### Architecture Documents for Codex Onboarding
- Updated Technical Architecture (v1.3), created Database Schema Reference, Module Building Guide, Dashboard Vision
- Committed (`9ee72cc`) and pushed to MBSPlatform main.

---

## March 31, 2026 — Session 13: Test Coverage + GDPR Doc Fix

### GDPR Documentation Fix
- CWG and FlowState GDPR endpoints were already implemented but docs said "Missing"
- Fixed GDPR status tables in: CLAUDE.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md, data-sovereignty.md
- All 15 apps now confirmed to have `DELETE /api/user-data` endpoints

### #21 Test Coverage Expansion — Billing + Entitlements
- Created `billing.test.js` — 19 tests covering:
  - POST `/api/billing/checkout` — direct priceId, slug+period resolution, invalid price, user not found, Stripe customer creation/reuse, invalid period, auth
  - POST `/api/billing/portal` — portal URL, no billing account, auth
  - GET `/api/billing/history` — transaction list, empty list, auth
  - POST `/api/billing/webhook` — checkout.session.completed (entitlement + transaction creation), subscription.deleted (cancellation), subscription.updated (past_due/active), invalid signature, unknown events
- Created `entitlements.test.js` — 21 tests covering:
  - GET `/api/entitlements` — empty list, active entitlements, auth
  - POST `/api/entitlements/subscribe-free` — 7-day trial creation, 409 duplicate, missing slug, unknown product, auth
  - GET `/api/entitlements/my-subscriptions` — enriched data with trial info, empty list
  - GET `/api/entitlements/category/:cat` — category product access, invalid category, auth
  - GET `/api/entitlements/:product` — not_subscribed, paid product_pass, free_tier, mbs_all_access priority, unknown product, trialing state, auth
- Updated `testApp.js` — added `createBillingApp()` and `createEntitlementsApp()` factory functions
- Total test count: 21 (auth) + 19 (billing) + 21 (entitlements) = 61 tests, all passing

### WildLens Toast Cleanup
- Confirmed all 21 files already using Sonner — custom ToastContext was orphaned dead code
- Deleted `context/ToastContext.js`, `components/shared/Toast.js`, removed `.toast-container` CSS
- Updated WildLens CLAUDE.md. Committed + pushed (`dfcb726`)

### LazyChef Sonner Migration — Already Complete
- Discovered migration was already done (commit `21881ba` on main, 30 files, 186 toast calls)
- No work needed — marked complete in FUTURE_WORK_TODO

### Standards Compliance Progress
- Updated remaining per-app items: LazyChef Sonner [x], WildLens Sonner [x]
- Remaining: TaskTracker CRA→Vite, per-app response helpers/validation/rate limiting

---

## March 31, 2026 — Session 12: RS256 Phase 2 + LazyChef + Repo Consolidation

### Marketing + Architecture Docs Updated
- All 4 marketing briefs updated with Session 9 pricing (CWG $15, FL $5, SW $10, Arcade $10, IL $20, MBS $30)
- FlowState status: "migration in progress" → "Live"
- Arcade brief: "SSO Migration Pending" → "SSO Migration Complete"
- Technical Architecture doc v1.1 → v1.2: all HS256 → RS256, pricing table, env vars, known limitations
- Historical disclaimer added to `audits/2026-03-30-platform-review.md` (pre-RS256 audit)
- All marketing docs synced to `Desktop/Marketing/Overview/`

### Repo Consolidation — Claude Setup / MBSPlatform Boundary
- Created `audits/` folder with 3 audit files absorbed from Claude Setup
- Merged per-app compliance items into FUTURE_WORK_TODO.md
- Fixed stale GDPR table in CLAUDE.md (added CWG + FlowState as missing)
- Fixed stale JWT reference in CLAUDE.md (removed HS256 fallback mention)
- Updated folder structure in CLAUDE.md to include audits/
- MBSPlatform is now self-contained for all MBS project documentation
- Claude Setup slimmed to Claude Code infrastructure only

### RS256 Phase 2 — Attempted and Reverted
- Removed HS256 fallback from all 15 repos (committed + pushed)
- Chrome verification: MBS Platform RS256 confirmed, SmartCart RS256 working
- Found: Lazy Chef Coolify hadn't redeployed (old HS256 code still running)
- Found: LazyChef frontend still calls local auth routes — removing `create_jwt_token` breaks login
- **Reverted all 15 repos** back to dual-mode (RS256 first, HS256 fallback)
- LazyChef self-issued auth restored
- Lesson: need to verify every Coolify redeploy + migrate LazyChef frontend before removing fallback

### RS256 Phase 2 (original, before revert) — Remove HS256 Fallback (all 15 repos)
- Removed HS256 backward-compatibility fallback from all 15 apps
- MBS Platform: `issueToken()` and `issueTempToken()` now require `JWT_PRIVATE_KEY` (no HS256 fallback)
- MBS Platform: `verifyToken()` now requires `JWT_PUBLIC_KEY` (RS256-only)
- 10 Node.js child apps: simplified verify to RS256-only, removed `JWT_SECRET` references
- 4 Python child apps: simplified verify to RS256-only, removed HS256 fallback code
- CWG: cleaned both `utils/auth.py` and `core/dependencies.py`

### LazyChef Self-Issued Auth Removal
- Removed `create_jwt_token()` function from `auth_service.py`
- Local signup, login, and Google OAuth callback routes now return 410 Gone
- LazyChef fully relies on MBS Platform SSO for all authentication

### Commits (per repo)
| Repo | Branch | Commit |
|------|--------|--------|
| MBS | main | `2b5dd8b` |
| Innerlab | main | `2e542cc` |
| YogaGhost | dev | `b463d86` |
| Brokenchain | main | `bb35491` |
| Wildlife | main | `e73b0aa` |
| Mindhacker | main | `d45d260` |
| Trivia | main | `87260b7` |
| Fakeartist | main | `429311d` |
| Shopping | main | `4260456` |
| Movie | main | `7b0ad64` |
| TaskTracker | main | `f7958d7` |
| CWG | test | `9baaad7` |
| LazyChef | main | `a00be57` |
| Tutor | main | `339c5d8` |
| Whispering House | main | `f49f79b` |

---

## March 31, 2026 — Session 11: RS256 JWT Upgrade (15 repos)

### RS256 Asymmetric JWT Signing
- Upgraded JWT signing from HS256 (symmetric) to RS256 (asymmetric) across all 15 apps
- MBS Platform: signs tokens with RS256 private key, dual-mode verify (RS256→HS256 fallback)
- 10 Node.js apps + 4 Python apps: dual-mode verify with `JWT_PUBLIC_KEY`
- New endpoint: `GET /api/auth/public-key` returns PEM public key
- RSA-2048 key pair generated, private key on MBS Backend only

### Bug Fixes During Deployment
- Coolify PEM format: must use "Is Multiline?" checkbox — single-line truncates PEM keys
- MBS `parsePemKey()`: handles Coolify's double-escaped `\\n` with `while` loop
- MBS graceful fallback: `jwt.sign()` wrapped in try/catch, falls back to HS256 on failure
- Tutor: fixed wrong import `from jose import jwt` → `import jwt` (PyJWT, not python-jose)
- Innerlab: regenerated stale `package-lock.json` (jest/supertest missing from lock file)

### Verified
- MBS Backend issuing RS256 tokens (confirmed `alg: RS256` via Chrome localStorage)
- All 15 repos committed and pushed
- Tutor and Innerlab deploy fixes pushed

### MBS commits: `d1173e6`, `49cf712`, `b33ff67`, `a0b4a74`

---

## March 30, 2026 — Session 10: 15 Enhancements + Entitlement Infrastructure

### 12 Enhancements (commit `0ba7114`)
- ADMIN_EMAILS env var, response helpers adoption (all 14 routes), activity logging
- Connected accounts UI, GDPR type-DELETE confirmation, 3-step onboarding flow
- Friends enhancement, AdminPage split (11 files), user segmentation, revenue analytics
- Referral email invites, promo code checkout flow

### 3 More Features (commit `2278e1d`)
- Real-time entitlement sync after Stripe checkout
- Premium feature gating (premiumFeatures for all 22 products, requireEntitlement/requirePremium middleware)
- Invoice PDF generation with PDFKit

### Bug fixes: `c374a94` (segment filter chips), `6150872` (admin tab visibility)

---

## March 30, 2026 — Session 9: Pricing Redesign + Premium UI Polish

### Pricing Overhaul
- **New 6-product pricing** with 25% annual discount: FlowState $5, CWG $15, SW $10, Arcade $10, IL $20, MBS $30
- **Modern BillingPage** — Monthly/Annual pill toggle (Annual default, "Save 25%"), hero MBS card, strikethrough pricing
- **New IndividualPlansPage** (`/billing/individual`) — 3-column layout with category-colored accent strips
- Updated backend billing.js price ID mapping for all 6 products

### Premium UI Polish (all 5 platform pages)
- Added ParticleField + AuroraBackground to all pages (AdminPage had NO background before)
- Card hover elevation with purple glow shadows
- Glow dividers between sections
- Color-coded section icons on AccountPage
- Category-colored left borders on ProductPickerPage
- Gradient stat card icons and purple glow active tab on AdminPage

### Critical Bug Fixes
- **SectionHeading incompatibility**: `whileInView`/`useInView` never fires under ProtectedRoute. Replaced with inline headings on all 4 protected pages
- **AnimatePresence initial mount**: Price toggles and admin tab content started invisible. Fixed with `initial={false}` on AnimatePresence
- **Key learning**: Never use scroll-triggered animations on ProtectedRoute pages

### Documentation
- SESSION_9_PENDING_ITEMS.md — 21 items organized by priority
- SESSION_10_PLAN.md — 12 enhancements planned for next session

### 9 MBS commits: `65d0612`, `d460c36`, `f7418d8`, `0e542c5`, `a67c631`, `ff2a4c9`, `f66205e`, `f9aa92d`, `6ba2e57`

---

## March 29, 2026 — Session 8: GDPR Deployed + 5 Roadmap Features + Standards Compliance

### Summary
Massive session covering 24 items: pushed all GDPR code, built Whispering House endpoint, fixed TaskTracker transactions, deployed everything to Coolify, built all 5 roadmap features (subscribe gating, product picker, free trial, CWG entitlement enforcement, admin dashboard), then did standards compliance (Winston logger, error handler, response helpers, userId shape fix, admin stats fix, logger crash fix). All verified live via Chrome.

### GDPR — Committed and Pushed

### Commits pushed
| Repo | Commit | What |
|------|--------|------|
| Fakeartist | `2775399` | Stub DELETE /api/user-data (no persistent data) |
| Trivia | `4dc33c8` | Stub DELETE /api/user-data (no persistent data) |
| Movie | `da991a9` | Deletes UserMovieInteraction, Watchlist, User |
| Mindhacker | `1e9d933` | Pulls from GameResult/Room, deletes Player |
| Brokenchain | `adb89ba` | Deletes Leaderboard, pulls from Room/Game/friends, deletes User |
| Wildlife | `65f6be7` | Deletes Discovery, CommunityPost, ChatSession, Collection, Bookmark, UserChallenge, Notification, uploads, User |
| MBS | `b0edb9b` | Cascade service, GDPR routes, deletion UI on AccountPage |

### Additional Fixes
| Item | Commit | What |
|------|--------|------|
| Whispering House GDPR | `117dbfd` | New Python/FastAPI DELETE /api/user-data endpoint |
| Wildlife dead code | `c9323d6` | Removed unused bcryptjs dependency |
| TaskTracker transactions | `8053c11` | Non-transactional fallback for standalone MongoDB |

### Coolify Deploys
All 9 services (8 backends + MBS frontend) redeployed via Coolify. All successful. MBS has webhook auto-deploy enabled.

### 5 Roadmap Features
| Feature | Repo | Commit | Details |
|---------|------|--------|---------|
| Subscribe gating + product picker + free trial | MBS | `ca7fac6` | Products API, subscribe-free with 7-day trial, checkAccess refactor (three-tier), ProductPickerPage, lazy trial downgrade |
| Admin dashboard | MBS | `ca7fac6` | AdminPage with overview/users/entitlements tabs, admin link on AccountPage |
| CWG entitlement enforcement | CWG | `f9c38ab` | Wired check_entitlement() to profile endpoint, syncs plan from MBS Platform |
| Friends consolidation | N/A | N/A | No work needed — MBS already has friends, CWG has none |
| Free trial (7 days) | MBS | `ca7fac6` | Bundled with subscribe-free, lazy downgrade after expiry |

### Standards Compliance (MBS `8b732fc`)
- Fixed admin stats response (nested → flat keys matching frontend)
- Fixed 7 auth routes: `id` → `userId` in response body
- Added Winston logger, centralized error handler, response helpers
- Fixed logger crash (`0ca9c42`) that broke all auth routes
- Set `is_admin: true` for both admin accounts via MongoDB

### 12 Enhancements (MBS `d52eaba` + `e2a0364`)
- AuthContext (React Context API) + ProtectedRoute component
- User profile editing (PUT /api/auth/profile + Edit button on AccountPage)
- Notification center (bell icon, announcements API, glass dropdown)
- Feature flags system (admin CRUD + public check + Flags tab)
- Activity feed (Activity tab, paginated ActivityLog)
- Product analytics (per-product subs, trial conversion, top products on Overview tab)
- Onboarding modal for users with 0 entitlements
- Push notifications (web-push, service worker, admin send, AccountPage toggle)
- Real-time admin dashboard (Socket.io /admin namespace, "Live" badge, toast events)
- Backfilled auth_provider on 19 legacy users (9 google + 10 email)
- Fixed ProtectedRoute blank pages (always render children, overlay for loading)

### All MBS Commits This Session
`b0edb9b` → `ca7fac6` → `0ca9c42` → `8b732fc` → `d52eaba` → `00e5b44` → `e2a0364`

---

## March 29, 2026 — Session 7: Architectural Decisions + GDPR Deletion Built

### Summary
Full doc review, prioritized all 16 remaining items, made major architectural decisions, then built GDPR deletion endpoints across 6 apps + cascade service + deletion UI in MBS Platform. Code is written but NOT committed/pushed to individual project repos yet.

### Architectural Decisions
- **Three-tier subscription model**: Not Subscribed → Free Subscriber → Premium Subscriber. Users must explicitly subscribe (even free) to access any product.
- **Subscribe gating for all apps**: Subscription click required. Premium gating only for CWG (only product with defined free/premium split).
- **Product picker for entire catalog**: All products (IL, Arcade, Studio Works). Users subscribe individually.
- **Friends consolidation: Option A (MBS Platform level)**: Remove from CWG/FlowState, use existing platform friends API.
- **Admin dashboard: hierarchical at MBS level**: Drill-down MBS → category → product. Absorbs CWG's existing admin data.
- **Admin accounts**: `terracedreamer@gmail.com` + `1984.abhinav@gmail.com` both `is_admin: true`. Future: `ADMIN_EMAILS` env var.
- **Free trial: 7 days premium per product** on subscription. Future: credit card upfront.
- **RS256 JWT upgrade**: Deferred to pre-launch.
- **Enterprise SSO**: Removed from roadmap.
- **CWG `test` branch**: Stays indefinitely.

### Code Built (NOT yet committed to individual repos)
| Project | Files Changed |
|---------|--------------|
| Fakeartist/ | new `server/routes/userData.js`, modified `server/server.js` |
| Trivia/ | new `server/routes/userData.js`, modified `server/index.js` |
| Movie/ | new `server/routes/userData.js`, modified `server/index.js` |
| Mindhacker/ | new `server/src/routes/userData.js`, modified `server/src/index.js` |
| Brokenchain/ | new `server/src/routes/userData.js`, modified `server/src/app.js` |
| Wildlife/ | new `server/routes/userData.js`, modified `server/index.js` |
| MBS/ | modified `server/config/products.js` (apiUrl), new `server/services/gdprCascade.js`, modified `server/routes/auth.js` (2 new routes + cascade on account delete), modified `src/pages/AccountPage.jsx` (deletion UI) |

### Files Updated (MBSPlatform repo)
- SESSION_HANDOFF.md — Full session 7 summary + uncommitted code reminder
- CHANGELOG.md — This entry
- CURRENT_STATUS.md — GDPR status updated
- FUTURE_WORK_TODO.md — Restructured with decided + deferred items

---

## March 28, 2026 — Session 6: Phase 5 Complete, All Products Verified, Architecture Docs Created

### Summary
All 5 build phases complete. Reviewed WildLens SSO login loop — identified two root causes (legacy user collision + Python JWT_SECRET naming) and cascaded fixes to all instructions. Collected and reviewed all 11 Phase 5 reports. Verified all 13 products live via Chrome browser automation. Established three-level GDPR deletion architecture. Created architecture reference documents.

### Key Decisions
- **GDPR three-level deletion**: App-level (within app only), category-level, and full account (both from magicbusstudios.com only). A user deleting data from WildLens does NOT delete their MBS account or other apps' data.
- **CWG stays on `test` branch**: Owner decision — not promoted to `main`.

### Documentation Created
- `architecture-docs/MBS_Platform_Technical_Architecture.md` — comprehensive technical reference
- `architecture-docs/MBS_Platform_Overview.md` — non-technical overview for marketing/executive use

### Learnings Cascaded
- `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` — Added Step 5 (getOrProvisionUser pattern for legacy user collision), Phase 5 Learnings section (4 lessons), updated GDPR section to three-level model
- `platform-instructions-for-cwg/PLATFORM_MIGRATION.md` — Added Python JWT_SECRET_KEY vs JWT_SECRET warning
- `platform-instructions-for-new-modules/CLAUDE.md` — Added cross-reference to legacy user collision fix

### Phase 5 Reports Collected (11 total)
| Product | Key Finding |
|---------|------------|
| BrokenChain | Both SSO bugs hit and fixed (JWT_SECRET fallback + legacy user) |
| Fake Artist | N/A — stateless JWT, no User model |
| Lazy Chef | Python app, safe pattern ($set platformUserId, not delete/recreate) |
| MindHacker | visitorId → platformUserId redesign, route prefix issue found |
| Movie Picker | resolveUser middleware pattern, rekeys Watchlist + UserMovieInteraction |
| SmartCart | ensureUser + migrateLegacyUser across 5 collections, Mixed _id type |
| TaskTracker | Transactional migration across 14 collections (needs replica set) |
| Trivia Roast | N/A — no User model, auth on POST only |
| AI Tutor | Confirmed BOTH bugs (JWT_SECRET naming + legacy user), both fixed |
| Whispering House | N/A — no standalone auth, auth on create+join only |
| WildLens | Full migration across 10+ collections including followers/following |

### Live Verification (Chrome browser — all 13 products)
- BrokenChain, SmartCart, TaskTracker, AI Tutor, WildLens, Whispering House, Lazy Chef, Movie Picker, Trivia Roast: **Authenticated** (showing user name/avatar)
- MindHacker, Fake Artist: **SSO Ready** (Sign In button — correct per-subdomain behavior)
- FlowState (yoga): **Authenticated** (full dashboard with user profile)
- CWG: **Authenticated** (guides page with Unlock Better Guidance modal)

### Memory Saved
- SSO login loop pattern (two root causes + fix)
- GDPR deletion architecture (three levels)

---

## March 27, 2026 — Session 5: Phase 3B + Phase 4 Reports Reviewed

### Summary
Reviewed Phase 3B (CWG refactor) completion report. Filed to phase-reports/. Cascaded Phase 3B learnings to FlowState instructions (User ID Resolution, Cross-Origin Login Redirect). Confirmed CWG running on `test` branch at cwg.magicbusstudios.com intentionally.

### Learnings Cascaded to FlowState
- User ID Resolution section (platform ObjectId vs module-specific IDs)
- Cross-Origin Login Redirect confirmation (already works via Inner Lab commit `efebe36`)

### Known Issues Added
- CWG Settings page crash ("Illegal constructor" — pre-existing)
- CWG admin stats show 0 (old data under UUID, queries use platform ID)
- CWG entitlements not wired (no free/premium enforcement)

---

## March 27, 2026 — Session 4: Phase 3A Complete, Docs Hardened, Marketing Updated

### Summary
Reviewed Phase 3A migration report (CWG data migration — 20 users, 28 collections). Filed report. Corrected collection count across all docs (28 actual, not 56). Added report generation instructions to ALL paste-ready prompts. Updated all 4 marketing docs with current platform state (four auth methods, live pages, correct login redirects). Added addendum workflow documentation. Phase 3B (CWG refactor) in progress by user.

### Documentation Changes
- **ORCHESTRATION_GUIDE.md**: Added addendum workflow section, orchestrator direct changes section, Phase 3A results, report instructions to all phase prompts
- **phase-reports/README.md**: Full rewrite — addendum pattern, orchestrator changes, review checklist
- **CLAUDE.md**: Collection count corrected (28 not 56)
- **CURRENT_STATUS.md**: Phase 3A done, Phase 3B not started
- **SESSION_HANDOFF.md**: Session 4 summary, Phase 3B in progress
- **platform-instructions-for-cwg/PLATFORM_MIGRATION.md**: Collection count corrected
- **Marketing docs (all 4)**: "3 auth methods" → 4 + 2FA, "Pre-Platform" → "Platform Built", CWG login redirect fixed to innerlab.ai, unified account concept added

### Phase Reports Filed
- `phase-reports/PHASE_3A_MIGRATION_REPORT.md` — CWG data migration results

### Key Findings from Phase 3A
- CWG had 28 collections (not 56 as documented) — corrected everywhere
- 20 users migrated (18 new + 2 merged by email)
- `consciousness_profile_structured` and `personal_history_structured` don't exist — dual-format normalization was unnecessary
- UUID → ObjectId remapping worked correctly for all foreign keys
- Script is idempotent (safe to re-run)

---

## March 26, 2026 — Session 3 (continued): Phase 1+2 Addendum Review, Live Fixes, Phase 3 Prep

### Summary
Reviewed both Phase 1 Addendum (email/password + 2FA) and Phase 2 Addendum (Inner Lab auth pages) reports. Made direct code fixes to MBS/ and Innerlab/ from the orchestrator. Full audit of Phase 3-5 instructions. All phases ready for execution.

### Code Changes (from orchestrator — not from project agents)
- **MBS/**: Added "Sign Up" button alongside "Login" in nav when logged out (Layout.jsx)
- **Innerlab/**: Added "Sign In" + "Sign Up" buttons to nav when not authenticated (Layout.jsx — desktop + mobile)
- **Innerlab/**: Branded auth page headings — "Sign in to Inner Lab" / "Join Inner Lab" with font-display italic gradient, tagline pill badge (AuthLoginPage.jsx, AuthSignupPage.jsx)
- **Innerlab/**: ProtectedRoute redirect confirmed fixed (was magicbusstudios.com/login → now /auth/login)
- **Both projects**: Created ORCHESTRATOR_CHANGES.md documenting all changes for future agents

### Phase 3-5 Audit Results
- CWG migration doc: CLEAN ✓
- YogaGhost migration doc: CLEAN ✓
- Standalone products doc: CLEAN ✓
- New-modules template: FIXED (module domain → innerlab.ai, billing redirect cleaned)
- Orchestration Guide: FIXED (FlowState redirect → innerlab.ai)

### Live Verification (via Chrome extension)
- innerlab.ai/auth/login: ✅ Live, branded heading visible
- innerlab.ai nav: ✅ Sign In + Sign Up buttons showing
- innerlab.ai/dashboard (unauthenticated): ✅ Redirects to /auth/login?redirect=%2Fdashboard
- api.innerlab.ai/health: ✅ {"status":"ok","service":"innerlab-middleware"}
- magicbusstudios.com nav: ✅ Login + Sign Up buttons showing
- magicbusstudios.com/auth/login: ✅ API reachable, email/password + Google SSO functional

### Future Work Added
- Post-signup module picker (show all modules, let users pick subscriptions)

---

## March 26, 2026 — Session 3: Phase 2 Review, Email/Password Auth, 2FA, Login Pages

### Summary
Reviewed Phase 2 report (Inner Lab Middleware built and deployed). Added email/password authentication and 2FA/TOTP as 4th auth method across the entire platform. Added login/signup page specs to both MBS and Inner Lab. Updated all downstream instructions.

### Architecture Decisions
- **Email/Password auth** added as 4th method (alongside Google SSO, Nostr, LNURL)
- **2FA/TOTP** with backup codes — optional, user-enabled from account settings
- **Login/signup pages on TWO sites**: magicbusstudios.com (MBS branded) and innerlab.ai (Inner Lab branded). Both call MBS Platform auth APIs.
- **Inner Lab modules** redirect to `innerlab.ai/auth/login` (not magicbusstudios.com)
- **CWG users auto-become platform users** during migration (password hashes + 2FA data copied)
- **Login = Signup for SSO methods** (auto-create on first auth). Email/password has a separate signup form.

### Phase 2 Report Review
- Phase 2 fully built and deployed at api.innerlab.ai + innerlab.ai
- 11 Mongoose models, 22 API routes, 4 dashboard pages
- Cross-project blocker found: Inner Lab ProtectedRoute redirects to `magicbusstudios.com/login` (should be `/auth/login`) — will be fixed when Inner Lab gets its own login page
- Schema additions not in original spec: snapshot_reason, full schemas for blockchain_anchors, sync_backups, analytics_events, notifications
- Phase 2 report copied to `phase-reports/PHASE_2_REPORT.md`

### Files Updated
- `platform-instructions-for-mbs/CLAUDE.md` — auth section rewritten (4 methods), User model updated (password_hash, email_verified, totp fields, auth_methods array), 11 new API routes, addendum #14 (email/password) + #15 (2FA/TOTP), signup page added
- `platform-instructions-for-innerlab/CLAUDE.md` — login page, signup page, reset-password page specs added, VITE_PLATFORM_URL + VITE_GOOGLE_CLIENT_ID env vars, auth-gated routing updated, nav bar auth state
- `platform-instructions-for-cwg/PLATFORM_MIGRATION.md` — CWG users copied with password_hash + 2FA + auth_methods, login redirects to innerlab.ai
- `platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md` — login redirects to innerlab.ai
- `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` — auth method list updated
- `platform-instructions-for-new-modules/CLAUDE.md` — auth method list updated, login points to Inner Lab
- `CLAUDE.md` (main) — auth section, user model, login pages table, deployment table
- `SESSION_HANDOFF.md`, `CURRENT_STATUS.md`, `CHANGELOG.md`

### Synced To
- MBS/platform-instructions/ ✅
- Innerlab/platform-instructions/ ✅
- CWG/platform-instructions/ ✅
- YogaGhost/platform-instructions/ ✅

---

## March 26, 2026 — Session 2: Full Audit, Marketing Briefs, Orchestration Guide, Build Prep

### Summary
6 audit passes, 50+ issues found and fixed. Architecture is implementation-ready. Orchestration workflow established with auto-generated phase reports.

### Marketing Brief Enhancements
- Absorbed Arcade detail into MBS master doc (URLs, taglines, pricing, Time Banks, target audience)
- Expanded Studio Works from one-liners to full feature descriptions
- Added Inner Lab Dashboard section to IL brief (module launcher, activity feed, cross-module intelligence, memory privacy)
- Enhanced FlowState description (breathwork, streaks, achievements, programs)
- Added design language to IL brief (exact colors, typography from live innerlab.ai)
- Added website state documentation to MBS brief (page listing, known issues)
- Synced all briefs to Desktop/Marketing/Overview/

### Architecture Fixes (across 6 audit passes)
- Fixed stale folder names in CLAUDE.md, CURRENT_STATUS.md (copy-to-* → platform-instructions-for-*)
- Fixed wrong MBS_PLATFORM_URL in IL middleware (platform.magicbusstudios.com → magicbusstudios.com)
- Fixed contradictory build-later/build-now in IL middleware
- Removed redundant il_shared_memories and il_daily_check_ins collections
- Synced memory document schema across IL middleware and new-modules kit
- Added friends API endpoints to main CLAUDE.md
- Fixed entitlement check URLs (added https://)
- Clarified IL middleware CORS (dashboard frontend only, modules use shared DB)
- Assigned prefixes: AstroCompass → astrocart_*, Archetypes → archetype_*
- Standardized status to "Coming Soon" across module tables
- Added il_user_wellness_profiles and il_blockchain_anchors to collection lists
- Fixed CWG collection count (39 → 42)
- Added user dedup/merge logic (upsert on email) to both migration docs
- Fixed avatar/picture, google_id/googleId, stripeCustomerId field renames in migrations
- Added all 12 missing FlowState User model fields with defaults

### New Architectural Specs Added
- JWT payload specification (userId, email, name, avatar, isAdmin, iat, exp)
- il_* schema contracts for 5 collections (check-ins, consciousness, histories, wellness, activity)
- MongoDB JSON Schema validation recommendations
- Open redirect protection (ALLOWED_REDIRECT_DOMAINS from CORS_ORIGINS)
- Token-in-URL handling (extract → store → replaceState)
- Deep link preservation through login redirect
- Free tier entitlement logic (product catalog freeTier flag, 5 reason values)
- BTCPay → Entitlement flow (webhook → create entitlement with expiry)
- Entitlement priority order (mbs_all_access > category > product > free)
- Entitlement cache spec (5min TTL, ?refresh=true invalidation)
- Entitlement category validation (innerlab|arcade|studioworks, no underscores)
- Admin endpoints: verify is_admin from DB not JWT
- JWT security model (shared secret risk, RS256 upgrade path)
- Token refresh strategy (Phase 1: simple expiry, Phase 2: refresh tokens)
- Entitlement API resilience (cache, graceful degradation)
- GDPR deletion cascade (cross-database: mbs_platform + inner_lab + all module prefixes)
- CORS complete domain enumeration (18 domains)
- Deployment checklist (env vars first, health check, defensive route loading)
- Removed stripe_customer_id from Entitlement model (lives on User only)
- stripe_customer_id placement clarification
- Dual write path for il_* clarified (modules via DB, dashboard via HTTP)
- userId → user_id mapping documented (JWT camelCase, DB snake_case)
- CWG cutover sequence (simplified: run script, deploy, users re-login)
- source_module field name fixed (was inconsistently "source" in one place)
- FlowState friends/invites collection naming fixed (no mbs_ prefix needed)

### New Files Created
- `platform-instructions-for-standalone-products/` — SSO migration for 11 Arcade + Studio Works apps
- `ORCHESTRATION_GUIDE.md` — Step-by-step with exact copy-paste prompts for each Claude Code session

### Files Updated
- CLAUDE.md (main), CURRENT_STATUS.md, FUTURE_WORK_TODO.md, SESSION_HANDOFF.md, CHANGELOG.md
- platform-instructions-for-mbs/CLAUDE.md
- platform-instructions-for-innerlab/CLAUDE.md
- platform-instructions-for-cwg/PLATFORM_MIGRATION.md
- platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md
- platform-instructions-for-new-modules/CLAUDE.md
- Brand Overview/MagicBusStudios_Brand_And_Company.md
- Brand Overview/InnerLab_Product_Brief.md
- Brand Overview/TheArcade_Marketing_Brief.md

### Completion Report System (added late Session 2)
- Added `Completion Report (REQUIRED)` section to all 5 platform-instructions docs
- Phase 1 → PHASE_1_REPORT.md, Phase 2 → PHASE_2_REPORT.md, etc.
- Each report tailored to phase-specific concerns (migration results, JWT integration, etc.)
- Reports feed back to orchestrator session for review before starting next phase

### Marketing Brief Final Sync
- Updated all 4 briefs to March 26 date
- Arcade brief game descriptions synced with live website copy (Broken Chain, MindHacker, Whispering House, Fake Artist)
- Trivia Roast URL typo note added to Arcade brief

### Build Prep
- JWT_SECRET generated
- Orchestration workflow documented (build → report → review → next phase)
- Pre-build checklist added to CURRENT_STATUS.md

---

## March 25, 2026 — Session 1 (final): Merged Architecture + Think Tank Model

### Major decision: Merge platforms into existing website projects
- MBS Platform code gets built inside `MBS/` folder (magicbusstudios.com) — NOT a separate project
- Inner Lab Middleware + Dashboard gets built inside `Innerlab/` folder (innerlab.ai) — NOT a separate project
- `ILPlatform/` folder no longer needed
- This repo (`MBSPlatform/`) becomes the **architecture think tank** — no code, just docs + reference CLAUDE.md files
- Created `copy-to-mbs/CLAUDE.md` — copy into `MBS/` to build Layer 1
- Updated `copy-to-innerlab/CLAUDE.md` — copy into `Innerlab/` to build Layer 2

### Result: 4 containers instead of 8
- magicbusstudios.com: frontend (marketing + login + billing) + backend (forms + SSO + entitlements)
- innerlab.ai: frontend (marketing + dashboard) + backend (il_* APIs)
- Same pattern as every other MBS product — no new containers needed

### Previous revision (also this date)
- Inner Lab Middleware backend must be built NOW, not later
- Inner Lab frontend/dashboard also built NOW (not deferred)
- Reason: migrations need il_* shared tables, dashboard has value even with just CWG data

### Files updated
- CLAUDE.md: revised build order, three-layer architecture clarification
- copy-to-innerlab/CLAUDE.md: changed from "build later" to "build now"
- SESSION_HANDOFF.md: revised build order and decision #4
- CURRENT_STATUS.md: updated next steps
- FUTURE_WORK_TODO.md: moved middleware to Priority 2B (build now), frontend stays later
- CHANGELOG.md: this entry

---

## March 25, 2026 — Session 1 (continued): Architecture Finalized

### What happened
- Inspected CWG database schema (56 collections documented in DATABASE_SCHEMA.md)
- Inspected FlowState database schema (7 collections documented in DATA_SCHEMA.md)
- Both product agents created MBS_DATABASE_MIGRATION_PLAN.md with bucket categorizations
- Discovered CWG has extensive sovereignty features: Nostr, LNURL-Auth, BTCPay Lightning, client encryption, blockchain anchors, GDPR consent/data requests
- Discovered FlowState has MongoDB backend (7 collections), not frontend-only as initially assumed
- Reviewed live sites: magicbusstudios.com, innerlab.ai, conversationswithgod.ai
- Confirmed branded login model (Inner Lab branding vs MBS branding based on origin)

### All architecture decisions finalized (17 total)
1. Fresh `inner_lab` database — old DBs stay as backups
2. User identity ONLY in `mbs_platform`
3. Shared IL data starts as module-prefixed, promote to `il_*` when needed
4. Inner Lab Core container — build later when 2-3 modules exist
5. Frontend: B+C hybrid (standalone modules + future unified dashboard)
6. Branded login (IL modules → IL branding, Arcade/SW → MBS branding)
7. Arcade/Studio Works — SSO only, no shared data
8. Admin — simple `is_admin` flag
9. Nostr/LNURL auth — MBS Platform level (all products)
10. Payments — Stripe + BTCPay both at MBS Platform
11. Friends/Invites — MBS Platform level with product context
12. Activity feed — Inner Lab level (built when IL frontend exists)
13. Push subscriptions — MBS Platform level
14. Feature flags — MBS Platform level
15. User memories (AI) — Inner Lab level with user opt-in (Option C)
16. Encryption & data export — Inner Lab level
17. Modules as separate containers sharing `inner_lab` database

### Files created/updated
- Updated `CLAUDE.md` — added sovereignty features, branded login, three-layer architecture, migration details
- Updated `SESSION_HANDOFF.md` — complete context with all 17 decisions
- Updated `CHANGELOG.md`, `CURRENT_STATUS.md`, `FUTURE_WORK_TODO.md`
- Created `copy-to-innerlab/CLAUDE.md` — full reference for building Layer 2

### No code written
- Project remains spec-only. Architecture is complete. Ready for Phase 1 implementation.

---

## March 24, 2026 — Session 1: Architecture Planning (Started)

### What happened
- Full architecture discussion across MBS Platform + Inner Lab ecosystem
- Read and analyzed all reference documents (Technical Architecture, Product Briefs, ChatGPT architecture docs, Inner Lab Blueprint)
- Reviewed live innerlab.ai website (marketing site + modules page)

### Initial decisions made
- Three-layer architecture established
- B+C hybrid frontend model agreed
- Module isolation (separate containers, shared DB) confirmed
