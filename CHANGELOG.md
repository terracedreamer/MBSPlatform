# CHANGELOG — MBS Platform

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
- marketing-docs/MagicBusStudios_Brand_And_Company.md
- marketing-docs/InnerLab_Product_Brief.md
- marketing-docs/TheArcade_Marketing_Brief.md

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
