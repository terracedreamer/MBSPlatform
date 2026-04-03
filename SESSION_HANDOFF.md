# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: April 3, 2026 (Session 17 — end of session)
**Git Branch**: main
**Last Commit**: See per-repo commits below
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## SESSION 17 SUMMARY (April 3, 2026) — Architecture Review, No Code Changes

### What was done this session:

**Full Architecture Review & Deep Dive**
- Comprehensive architecture review across all codebases (CWG, Inner Lab, FlowState, MBS Platform)
- 14 architecture decisions confirmed for shared data, GDPR, sharing toggle, dashboard access

**Critical Bugs Found (Not Yet Fixed)**
- CWG GDPR endpoint missing il_reflections + il_activity_feed from deletion scope
- CWG GDPR endpoint not filtering il_* collections by source_module (would delete other modules' data)
- FlowState writes zero il_* data (no check-ins, no memories, no reflections to shared collections)
- FlowState GDPR endpoint has inconsistent user_id fields (mix of userId vs user_id)
- il_reflections Mongoose model has visibility default "private" — should be "shared" per architecture decision

**Documentation Updated (10+ files)**
- Updated docs across Marketing/Architecture Docs, Innerlab, global rules, Incubator doc 28
- Created new `Inner_Lab_GDPR_Deletion_Scoping.md` architecture doc

**14 Architecture Decisions Confirmed**
- Identity data (consciousness profile, personal history) = always shared, no toggle needed
- Module activity data = per-module sharing toggle (default ON, retroactive Option B)
- GDPR app-level: filter il_* by source_module, don't delete singleton identity docs
- Dashboard requires any IL module subscription (not a separate subscription)
- Nexus = marketplace for spiritual practitioners
- BreathArc: keep in products.js for now (not yet removed from code)
- il_reflections visibility default = "shared" (code currently says "private" — fix needed)
- FlowState il_* integration deferred (not blocking CWG refactor)

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403
3. **CWG merge test → main** — requires passphrase (do AFTER CWG refactor is complete)
4. **Confirm BreathArc removal** — ~mid-April 2026

### For next session (Session 18):
1. **CWG refactor** — finish remaining 11 files that reference cwg_journal_entries
2. **Fix CWG GDPR endpoint** — add il_reflections + il_activity_feed to deletion scope, add source_module filtering on all il_* deletes
3. **Fix il_reflections Mongoose model** — change visibility default from "private" to "shared"
4. **CWG consciousness profile + personal history migration** — move from cwg_user_profiles to il_consciousness_profiles and il_personal_histories
5. **Run CWG journal migration script** — copy existing cwg_journal_entries to il_reflections
6. **Verify Inner Lab** — confirm Journal page shows CWG data, consciousness profile shows, personal history shows
7. **Chrome verification** — test all changes live after Coolify redeploy
8. Per-app standards improvements
9. LazyChef SSO migration

### Key architecture decisions confirmed in Session 17:
- **Identity data is always shared** — consciousness profile and personal history have no per-module toggle. They are singleton documents at IL level.
- **Module activity data has per-module sharing toggle** — default ON, retroactive (Option B: when toggled ON, past data becomes visible to other modules)
- **GDPR app-level deletion filters by source_module** — CWG delete only removes `source_module: "cwg"` entries from il_reflections, il_check_ins, il_user_memories, il_activity_feed. Does NOT delete singleton identity docs (consciousness profile, personal history).
- **il_reflections visibility default = "shared"** — the Mongoose model currently defaults to "private", which contradicts the decision. Must be fixed.
- **Dashboard access requires any IL module subscription** — not a separate product. `category_access: "innerlab"` or any `product_pass` for an IL module.
- **Nexus = marketplace** for spiritual practitioners (not a social network)
- **FlowState il_* integration deferred** — FlowState currently writes zero shared data. Will be addressed separately from CWG refactor.

---

## SESSION 16 SUMMARY (April 2, 2026) — End of session

### What was done this session:

**Innerlab Incubator Cross-Reference Review (Full)**
- Reviewed `docs/27-claude-cross-reference-pack.md` against live code in MBS/, Innerlab/, CWG/, YogaGhost/
- Answered all 30 original question IDs + 7 follow-up questions from Codex
- 3 critical findings: FlowState has breathwork+meditation (not movement-only), CWG auto-extracts memories silently, Nexus premiumFeatures conflict
- Full responses appended to `Innerlab Incubator/docs/28-claude-response-log.md`

**BreathArc Provisional Removal Decision**
- FlowState confirmed as three-pillar module (yoga + breathwork + meditation)
- BreathArc provisionally removed from module catalog (pending final confirmation ~mid-April 2026)
- Wave 1 build order changed: dreamlens → rituals (was: breatharc → dreamlens → rituals)
- All cleanup deferred — no live repo changes until confirmed

**Layer 2 Contract Reference for Codex**
- Added complete il_* schema documentation to doc 28 (all 11 live collections with exact fields, types, indexes)
- Added module build guardrails (what modules must/must not do)
- Added build ownership split (Claude Code builds platform, Codex builds modules)
- Added module graduation checklist

**MBS Platform Test Verification**
- All 107 tests pass across 7 test suites (admin, friends, promos, referrals tests were already written — verified working)
- npm install needed locally (otplib wasn't installed) — now resolved

**Arcade SSO Verification (Chrome)**
- BrokenChain: SSO redirect confirmed → magicbusstudios.com/auth/login?redirect=brokenchain
- MindHacker: SSO redirect confirmed → magicbusstudios.com/auth/login?redirect=mindhacker
- FakeArtist: SSO redirect confirmed → magicbusstudios.com/auth/login?redirect=fakeartist
- All three working correctly

**Inner Lab Platform Build (8 dashboard pages, all pushed to main)**
- il_reflections model + CRUD API (6 entry types, first-class module fields)
- Journal page, Check-In History page, Wellness page, Weekly Review page
- Notification routes (6 endpoints) + NotificationBell component in header
- Wellness profile routes (get + upsert)
- Dashboard enhancements: state-aware greeting, continue-where-left-off, sparkline, memory resurfacing
- GDPR cascade updated (12 collections)

**CWG → Inner Lab Data Architecture Decision**
- Journals: il_reflections is the ONLY journal store. CWG (and all modules) write directly there.
- Module-specific fields (mood, symbols, emotions, etc.) are first-class schema fields, NOT metadata.
- Consciousness profile + Personal history ("My Story") will also move to Inner Lab level.
- CWG becomes a thin module that reads identity data from il_* collections.

**CWG Partial Refactor (pushed to test, WIP)**
- journal_routes.py fully refactored to read/write il_reflections
- journal_insights_routes.py and chat_routes.py refactored
- Migration script written (not yet run)
- 11 CWG files still reference cwg_journal_entries
- Consciousness profile sync added
- Memory schema fix (memory→content, added source_module/memory_type/shared)
- Check-in schema fix (added source_module, mapped spirit_level→mood)

**Incubator Cross-Reference + BreathArc Decision**
- Full 30-question review + 7 follow-ups answered in doc 28
- BreathArc provisionally removed (FlowState keeps yoga+breathwork+meditation)
- Doc 28 updated with: Layer 2 contract reference, module guardrails, build ownership split, journaling architecture

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403
3. **CWG merge test → main** — requires passphrase (do AFTER CWG refactor is complete)
4. **Confirm BreathArc removal** — ~mid-April 2026

### For next session (Session 17) — COMPLETED: architecture review done, items 2-8 deferred to Session 18:
1. ~~**Re-discuss shared data architecture**~~ — **DONE** (14 architecture decisions confirmed)
2. **CWG refactor** — finish remaining 11 files (deferred to Session 18)
3. **CWG consciousness profile + personal history migration** — deferred to Session 18
4. **Run CWG journal migration script** — deferred to Session 18
5. **Verify Inner Lab** — deferred to Session 18
6. **Chrome verification** — deferred to Session 18
7. Per-app standards improvements — deferred
8. LazyChef SSO migration — deferred

### Key architecture decisions confirmed in Session 17:
- **il_reflections is the ONLY journal store** — no module-local journal collections (CONFIRMED)
- **Module-specific fields are first-class schema fields** on il_reflections (mood, symbols, emotions, etc.), NOT metadata (CONFIRMED)
- **Same data, different views** — CWG shows entries with mood words + guide context; Inner Lab shows them plain. Both read from il_reflections. (CONFIRMED)
- **Data lives at Inner Lab level, UI can exist in multiple places** — Inner Lab, CWG, DreamLens all can show My Story / Consciousness Profile / Journal. First write wins. All read from the same il_* collections. (CONFIRMED)
- **Mobile-ready** — standalone apps collect data locally if no Inner Lab, sync to il_* when connected. Not built yet but architecture supports it. (CONFIRMED)
- **CWG keeps its frontend** for My Story and Consciousness Profile — just rewires backend to read/write from il_* instead of cwg_user_profiles (CONFIRMED)
- **NEW: Identity data always shared** — consciousness profile + personal history have no per-module toggle
- **NEW: Module activity data has sharing toggle** — default ON, retroactive Option B
- **NEW: GDPR app-level filters by source_module** — don't delete singleton identity docs
- **NEW: Dashboard access = any IL module subscription** — not a separate product
- **NEW: Nexus = marketplace** for spiritual practitioners
- **NEW: il_reflections visibility default = "shared"** — code says "private", fix needed

---

## SESSION 15 SUMMARY (April 2, 2026)

### What was done this session:

**Brand Documentation Gap Analysis & Fixes**
- Created `FlowState_Product_Brief.md` in `Desktop/Marketing/Brand Overview/` (400+ lines from codebase exploration — three-pillar wellness, pose detection, mastery system, AI coaching, breathwork, meditation)
- Updated `MagicBusStudios_Brand_And_Company.md` — added "The Consciousness Brand Layer" subsection (consciousness.dev, myconsciousness.io, consciousness.tools ecosystem diagram)
- Updated `InnerLab_Product_Brief.md` — added Section 7C: "Inner Lab in the Consciousness Ecosystem" (relationship table, flywheel model, marketing positioning)
- Verified CWG 67-language count was factually correct (marketing agent's update confirmed against actual locale directories)

**Documentation Consolidation — Single Source of Truth**
- Marketing folder (`Desktop/Marketing/`) is now the single source of truth for all brand briefs and architecture docs
- Deleted `MBSPlatform/Architecture Docs/` (5 files — identical to Marketing copies)
- Deleted `MBSPlatform/Brand Overview/` (8 files — Marketing copies were more up-to-date)
- Updated `MBSPlatform/CLAUDE.md` to reference Marketing folder instead of local copies

**TaskTracker SSO Redirect Fix (ROOT CAUSE FOUND)**
- Issue: After Google SSO login, user ended up at `magicbusstudios.com` instead of being redirected back to TaskTracker
- Root cause: Coolify build arg `VITE_PRODUCT_DOMAIN=https://tasktracker.magicbusstudios.com` had `https://` prefix, but the code already prepends `https://` in the redirect URL construction
- Result: redirect URL became `https://https://tasktracker.magicbusstudios.com` — `validateRedirectUrl()` parsed hostname as `"https"` which isn't in CORS_ORIGINS → returned safe default `magicbusstudios.com`
- Fix: User removed `https://` prefix from Coolify build arg → `VITE_PRODUCT_DOMAIN=tasktracker.magicbusstudios.com`
- This also resolved the 403 errors on all data routes (AuthCallback flow now executes properly)

**TaskTracker Chrome Verification — ALL PASSING**
- `/api/auth/me` → 200, `/api/tasks` → 200, `/api/family` → 200, `/api/approvals` → 200, `/api/rewards` → 200, `/api/goals` → 200
- No console errors. Sentry initialized. Vite hash-pattern assets confirmed (`index-DB661mMO.js`)
- Dashboard: "Abi's Family" loaded, tasks visible with assignments, all 8 navigation tabs working
- Standards compliance: 8/8 PASS (Chrome verification now complete)

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403
3. **CWG merge test → main** — requires passphrase

### For next session (Session 16):
- More MBS Platform tests — admin, friends, promos, referrals
- CWG: merge `test` → `main` when ready
- Per-app standards improvements
- LazyChef SSO migration
- BrokenChain/MindHacker/FakeArtist SSO activation in Coolify
- Consciousness DNS completion

---

## SESSION 14 SUMMARY (April 2, 2026)

### What was done this session:

**TaskTracker CRA → Vite Migration (COMPLETE)**
- Replaced react-scripts with vite@5 + @vitejs/plugin-react-swc
- Renamed all 70 `.js` files to `.jsx` (required by Vite's esbuild pipeline)
- Replaced all `process.env.REACT_APP_*` with `import.meta.env.VITE_*`
- Replaced all `process.env.NODE_ENV` checks with `import.meta.env.MODE/DEV/PROD`
- Moved `index.html` from `client/public/` to `client/` root
- Updated nixpacks.toml and build scripts for `client/dist` output
- Build succeeds: 848 modules transformed in 2.01s
- Committed (`0b75506`) and pushed to TaskTracker main

**Architecture Documents for Codex Onboarding (COMPLETE)**
- Updated `MBS_Platform_Technical_Architecture.md` (v1.3): Fixed TaskTracker CRA→Vite ref, GDPR status, added Session 13-14 completed items
- Created `MBS_Database_Schema_Reference.md`: Complete MongoDB schemas for mbs_platform + inner_lab databases (all 29+ collections documented with field types, constraints, indexes)
- Created `Inner_Lab_Module_Building_Guide.md`: Full guardrails for building new IL modules — auth patterns, database rules, deployment setup, coding standards, 13 critical guardrails, 15+ lessons learned
- Created `Inner_Lab_Dashboard_Vision.md`: Product vision for making Inner Lab the star — dashboard enhancement priorities, API specs for daily briefing/trends, data flow diagrams
- Updated `deployment-quirks.md`: TaskTracker now uses Vite (not CRA)
- Committed (`9ee72cc`) and pushed to MBSPlatform main

### Commits this session:
| Repo | Commit | Message |
|------|--------|---------|
| TaskTracker | `0b75506` | feat: migrate frontend from CRA to Vite |
| MBSPlatform | `9ee72cc` | docs: add comprehensive architecture docs for Codex onboarding |

### Pending — Owner Action:
1. ~~Update Coolify build args for TaskTracker~~ — **DONE** (Session 15). Also fixed `VITE_PRODUCT_DOMAIN` prefix bug.
2. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
3. **Regenerate BTCPay API key** — current returns 403
4. ~~Verify TaskTracker live~~ — **DONE** (Session 15). All 6 API routes returning 200, Chrome verified.

---

## SESSION 13 SUMMARY (March 31, 2026)

### What was done this session:

**GDPR Documentation Fix — CWG + FlowState Already Implemented**
- Session 12 docs said CWG and FlowState GDPR endpoints were "Missing" — they were already implemented
- CWG: `DELETE /api/user-data` in `server.py` (38 cwg_* + 7 il_* collections)
- FlowState: `DELETE /api/user-data` in `index.js` (7 yoga_* collections)
- Fixed GDPR status tables in: CLAUDE.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md, `~/.claude/rules/data-sovereignty.md`
- All 15 apps now confirmed to have GDPR endpoints

**#21 Test Coverage Expansion — Billing + Entitlements (MBS Platform)**

Added 40 new tests across 2 test files. Total MBS Platform test count: 61 (up from 21).

| File | Tests | What's Covered |
|------|-------|---------------|
| `billing.test.js` | 19 | Checkout (priceId, slug+period, validation, Stripe customer), Portal, History, Webhook (all 4 event types + signature + unknown events) |
| `entitlements.test.js` | 21 | List, Subscribe-free (trial, duplicate, validation), My-subscriptions (enriched+trial), Category check, Product access (not_subscribed, paid, free_tier, mbs_all_access priority, trialing) |
| `testApp.js` | — | Added `createBillingApp()` and `createEntitlementsApp()` factory functions |

All 61 tests pass. Mocked Stripe + Mongoose models — no real DB or Stripe needed.

**WildLens Toast Cleanup**
- Confirmed all 21 files already import from Sonner — custom ToastContext was orphaned
- Deleted `context/ToastContext.js`, `components/shared/Toast.js`, removed `.toast-container` CSS
- Updated WildLens CLAUDE.md. Committed + pushed (`dfcb726`)

**LazyChef Sonner Migration — Already Done**
- Discovered migration was already completed (commit `21881ba` on main)
- 30 files, 186 toast calls, all using Sonner. No work needed.

**Exploration for next session:**
- MBS admin/friends/promos/referrals routes fully analyzed — ready for test writing
- TaskTracker CRA→Vite scope: 71 files, 4 REACT_APP_ vars, 3 deploy targets (Coolify/Vercel/VPS)

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `bae89d9` | feat: add billing + entitlements test coverage (40 new tests) |

### Other repo commits this session:
| Repo | Commit | Message |
|------|--------|---------|
| MBSPlatform | `69f2f7a` | docs: Session 13 — test coverage + GDPR doc fix |
| MBSPlatform | `9bb2507` | docs: end-session update — WildLens/LazyChef toast complete, stale refs fixed |
| Wildlife | `dfcb726` | chore: remove orphaned custom toast files (Sonner migration complete) |

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403

### For next session (Session 14):
- More MBS Platform tests — admin, friends, promos, referrals (routes already analyzed)
- TaskTracker CRA → Vite migration (71 files, 4 env vars, 3 deploy targets)
- CWG: merge `test` → `main` when ready
- Per-app standards improvements (response helpers, input validation, rate limiting)
- LazyChef SSO migration (future — frontend must redirect to MBS Platform before RS256 Phase 2)

---

## SESSION 12 SUMMARY (March 31, 2026)

### What was done this session:

**RS256 Phase 2 — Attempted and Reverted**

Attempted to remove HS256 fallback from all 15 repos. Chrome verification revealed:
- MBS Platform issues RS256 tokens correctly (`alg: RS256` confirmed)
- SmartCart (Node.js) accepted RS256 token and loaded data
- Lazy Chef (Python) still ran old code (Coolify hadn't redeployed) — HS256 token still accepted
- LazyChef frontend still calls local auth routes (`/api/auth/login`, `/api/auth/google/callback`) — removing `create_jwt_token` would break login

**Decision: Reverted all 15 repos back to dual-mode (RS256 first, HS256 fallback).** The fallback is a safety net that costs nothing. Removing it requires:
1. Verifying every Coolify service has redeployed with `JWT_PUBLIC_KEY` working
2. Migrating LazyChef frontend to MBS Platform SSO before removing local auth
3. Verifying all 15 apps end-to-end via Chrome after each Coolify redeploy

**What was changed (then reverted):**
- **MBS Platform**: `verifyToken()`, `issueToken()`, `issueTempToken()` — made RS256-only. **Reverted** to dual-mode.
- **10 Node.js child apps**: Simplified to RS256-only. **Reverted** to dual-mode with HS256 fallback.
- **4 Python child apps**: Simplified to RS256-only. **Reverted** to dual-mode.
- **CWG**: Both `utils/auth.py` and `core/dependencies.py` cleaned. **Reverted.**
- **All 15 repos**: `git revert` commits pushed. Code is identical to end of Session 11.

**LazyChef — Self-Issued Auth Removal Attempted and Reverted**
- Removed `create_jwt_token()` and set local auth routes to 410 Gone
- Chrome verification found LazyChef frontend still calls local `/api/auth/login` and `/api/auth/google/callback` — removing these breaks login
- **Reverted**: `create_jwt_token` and all local auth routes restored

### Net effect after revert:
- All 15 repos back to **dual-mode**: RS256 first, HS256 fallback (same as Session 11)
- LazyChef self-issued auth restored (same as Session 11)
- RS256 is confirmed working on MBS Platform (token shows `alg: RS256`)
- HS256 fallback ensures no app breaks regardless of Coolify redeploy timing

**Repo consolidation (same session):**
- Absorbed 3 audit files from Claude Setup into `MBSPlatform/audits/` (compliance audit, Arcade brand audit, platform review)
- Merged per-app standards compliance items into FUTURE_WORK_TODO.md
- Fixed stale GDPR table in CLAUDE.md (CWG and FlowState still missing)
- Fixed stale JWT comment in CLAUDE.md (HS256 fallback reference removed)
- Added `audits/` folder to CLAUDE.md folder structure
- Claude Setup slimmed to Claude infrastructure only (skills, rules, tasks, backups)

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403

### For next session (Session 13):
- CWG GDPR endpoint (`DELETE /api/user-data`) — pure addition, on `test` branch
- FlowState GDPR endpoint — pure addition, push to `dev`
- #21 Test coverage expansion (frontend + billing + entitlements)
- CWG: merge `test` → `main` when ready

---

## SESSION 11 SUMMARY (March 31, 2026)

### What was done this session:

**#16 RS256 JWT Upgrade — All 15 repos upgraded, committed, pushed, verified live**

Upgraded JWT signing from HS256 (symmetric shared secret) to RS256 (asymmetric) across the entire 15-app ecosystem. This is a security-critical change: previously, if any child app's `JWT_SECRET` leaked, an attacker could forge tokens for ALL apps. Now only MBS Platform holds the private signing key; child apps only have the public verification key.

**Changes by repo:**

| Repo | Branch | File(s) Changed | What Changed |
|------|--------|-----------------|-------------|
| MBS | main | `server/middleware/auth.js`, `server/routes/auth.js`, `server/routes/admin.js`, `server/services/realtimeService.js` | Sign with RS256 (private key), dual-mode verify (RS256→HS256), new `GET /api/auth/public-key` endpoint, centralized `verifyToken()` function |
| Innerlab | main | `server/middleware/requireAuth.js` | Dual-mode verify |
| YogaGhost | dev | `server/auth.js` | Dual-mode verify |
| Brokenchain | main | `server/src/middleware/auth.js` | Dual-mode verify |
| Wildlife | main | `server/middleware/auth.js` | Dual-mode verify + helper function |
| Mindhacker | main | `server/src/middleware/auth.js` | Dual-mode verify |
| Trivia | main | `server/middleware/auth.js` | Dual-mode verify |
| Fakeartist | main | `server/middleware/requireAuth.js` | Dual-mode verify |
| Shopping | main | `server/middleware/auth.js` | Dual-mode verify |
| Movie | main | `server/middleware/auth.js` | Dual-mode verify |
| TaskTracker | main | `server/middleware/auth.js` | Dual-mode verify |
| CWG | test | `backend/utils/auth.py`, `backend/core/dependencies.py` | Dual-mode verify (PyJWT) |
| LazyChef | main | `backend/auth_service.py` | Dual-mode verify (PyJWT) |
| Tutor | main | `backend/auth_service.py` | Dual-mode verify (PyJWT — NOT python-jose) |
| Whispering House | main | `backend/app/core/auth.py` | Dual-mode verify (python-jose) |

**Key design decisions:**
- **Backward compatible**: All existing HS256 tokens continue to work via fallback
- **Dual-mode verify pattern**: Try RS256 (public key) first → if `TokenExpiredError`, re-throw (genuinely expired) → otherwise fall back to HS256 (old token)
- **RSA-2048 key pair generated**: Private key for MBS Backend only, public key for all 15 services
- **New endpoint**: `GET /api/auth/public-key` on MBS Platform returns PEM public key (for future dynamic key fetching)
- **LazyChef**: Still has legacy `create_jwt_token` (local auth). `verify_jwt_token` now supports RS256. Full removal of self-issued tokens is a separate cleanup task.

### Coolify deployment progress:
- ✅ **MBS Backend**: `JWT_PRIVATE_KEY` + `JWT_PUBLIC_KEY` added (multiline format), redeployed, **RS256 verified working** (token shows `alg: RS256`)
- ✅ **All 14 child apps**: `JWT_PUBLIC_KEY` added (multiline format in Coolify, "Is Multiline?" checkbox checked)

### Bug fixes during deployment:
- **Coolify PEM format**: Single-line env vars truncate PEM keys. Must use "Is Multiline?" checkbox in Coolify UI.
- **MBS `parsePemKey()`**: Added `while` loop to handle Coolify's double-escaped `\\n`, plus graceful fallback if RS256 signing fails (falls back to HS256 with error logging).
- **Tutor import bug**: Initial commit used `from jose import jwt` (python-jose) but Tutor only has `pyjwt`. Fixed to `import jwt` (PyJWT). Caused Coolify deploy failure (`ModuleNotFoundError: No module named 'jose'`).
- **Innerlab lock file**: `jest` + `supertest` added to `package.json` but `package-lock.json` not regenerated. Fixed with `npm install`. Caused ~6 failed Coolify deploys.

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `d1173e6` | feat: RS256 JWT signing — MBS Platform issuer upgrade |
| `49cf712` | fix: RS256 signing graceful fallback + robust PEM parsing |
| `b33ff67` | fix: improve RS256 error logging with key format diagnostics |
| `a0b4a74` | fix: handle double-escaped \n in PEM keys from Coolify |

### Pending — Owner Action:
1. ~~Add `JWT_PUBLIC_KEY` to child apps~~ — **DONE** (all 15 services configured)
2. **Keep `JWT_SECRET`** on all services — HS256 fallback still active (Phase 2 was attempted and reverted in Session 12)
3. `JWT_SECRET` is still needed — do NOT remove from Coolify env vars
4. **Stripe Dashboard** — Create 6 products with 12 prices (still pending from Session 9)
5. **BTCPay** — Regenerate API key with full store permissions (still pending)

### For next session (Session 12):
- #21 Test coverage expansion (frontend + billing + entitlements)
- RS256 Phase 2 cleanup: remove HS256 fallback from all apps (after 7 days)
- LazyChef: remove `create_jwt_token` self-issued auth (fully rely on MBS Platform)
- Verify RS256 is working end-to-end via Chrome on live site

---

## SESSION 10 SUMMARY (March 30, 2026)

### What was done this session:

**Batch 1 — Backend Hardening**
1. **#17 ADMIN_EMAILS env var** — `isEnvAdmin()` checks env before DB in `requireAdmin` + `issueToken` (auth.js)
2. **#19 Response helpers adoption** — all 14 route files converted (~316 conversions). No more inline `res.json({success})`.
3. **#13 Activity logging for subscription events** — subscribe-free, checkout completed, cancelled, expired, upgraded, trial expired

**Batch 2 — AccountPage Enhancements**
4. **#11 Connected Accounts UI** — Google, Email, Nostr, LNURL connection status + link Nostr form on AccountPage
5. **#18 GDPR "Type DELETE" confirmation** — all 3 delete modals (app, category, account) now require typing DELETE
6. **#10 3-step onboarding flow** — Welcome categories -> Pick product (subscribe-free) -> Success

**Batch 3 — Friends Enhancement**
7. **#12 Friends system enhancement** — avatar grid UI, remove friend button, `DELETE /api/friends/:friendId`, `GET /api/friends/:friendId/shared-products`

**Batch 4 — Admin Dashboard Overhaul**
8. **#20 AdminPage split** — 2067 -> 767 lines + 6 tab components + 8 reusable admin components (11 new files)
9. **#14 User segmentation** — `GET /api/admin/users/segments` endpoint + filter chips on Users tab (new/trial/paid/free/churned/inactive)
10. **#15 Revenue analytics** — `GET /api/admin/analytics/revenue` + new Analytics tab (MRR, ARR, ARPU, churn rate, growth rate, monthly trend, revenue by product)

**Batch 5 — Billing Features**
11. **#4 Referral email invites** — SendGrid HTML template, 10/day rate limit, preference check
12. **#5 Promo code checkout flow** — validate promo -> create Stripe coupon (percentage/fixed/trial_extension) -> apply to session + UI input on BillingPage

**Bug Fixes (post-batch)**
13. **Fix AdminUsers segment filter chips** — segments object was not being converted to array for filter chip rendering (`c374a94`)
14. **Fix admin tab content visibility** — added `initial={false}` to AnimatePresence in admin tabs for ProtectedRoute compatibility (`6150872`)

**Batch 6 — Entitlement & Billing Enhancements**
15. **#3 Real-time entitlement sync** — `refreshUser()` called on BillingPage `?success=true` redirect after Stripe/BTCPay checkout, so user sees updated access immediately
16. **#4 Premium feature gating** — `premiumFeatures` array added to all 22 products in `products.js`, `requireEntitlement(slug)` and `requirePremium` middleware created (`server/middleware/entitlement.js`), enhanced `GET /api/entitlements/:product` response with `isPremium` and `premiumFeatures`
17. **#5 Invoice PDF generation** — PDFKit installed, `server/services/invoiceService.js` creates branded PDFs, `GET /api/billing/invoice/:transactionId` endpoint, download button in BillingPage transaction history

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `0ba7114` | feat: Session 10 — 12 enhancements in 5 batches (32 files changed, 3426 insertions, 1811 deletions) |
| `c374a94` | fix: convert segments object to array for filter chips (AdminUsers) |
| `6150872` | fix: admin tab content initial={false} for ProtectedRoute compat |
| `2278e1d` | feat: entitlement sync, premium gating, invoice PDF generation |

### Pending — Owner Action:
1. **Stripe Dashboard** — Create 6 products with 12 prices (see FUTURE_WORK_TODO.md for table)
2. **BTCPay** — Regenerate API key with full store permissions (current returns 403)

### For next session (Session 11):
- **#16 RS256 JWT upgrade** (multi-repo, 2-3 days) — plan written and approved at `.claude/plans/humble-sparking-wigderson.md`, ready to execute
- #21 Test coverage expansion (frontend + billing + entitlements)

---

## SESSION 9 (March 30, 2026) — Pricing Redesign + Premium UI Polish
- Updated pricing: 6 products with 25% annual discount (FlowState $5, CWG $15, SW $10, Arcade $10, IL $20, MBS $30)
- Redesigned BillingPage (monthly/annual toggle, hero card, strikethrough pricing) + new IndividualPlansPage
- Premium UI polish on all 5 platform pages (particles, aurora, glass cards, hover elevations)
- Fixed ProtectedRoute animation bugs (SectionHeading, AnimatePresence initial={false})
- 9 commits: `65d0612` through `6ba2e57`

---

## SESSION 8 (March 29, 2026) — GDPR + Roadmap Features + Standards
- GDPR deletion committed/pushed to all 7 repos + Whispering House + MBS cascade service
- 5 roadmap features: subscribe gating + product picker + free trial (7-day), admin dashboard, CWG entitlements
- 12 enhancements: AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin
- Standards compliance: Winston logger, centralized error handler, response helpers, userId shape fix
- Key decisions: three-tier subscription, CWG stays on test, 7-day trial, RS256 deferred
- All verified live via Chrome. Commits: `b0edb9b` through `e2a0364`

---

## SESSION 6 (March 28, 2026) — Phase 5 Complete
- All 11 standalone products deployed and verified live via Chrome
- WildLens SSO login loop root causes identified and cascaded
- Architecture docs created, GDPR three-level deletion designed
- Platform-instructions re-synced to all 15 project folders

---

## ALL 5 PHASES COMPLETE

| Phase | Status | Verified |
|-------|--------|----------|
| Phase 1: MBS Platform | DONE — deployed at magicbusstudios.com | Yes |
| Phase 2: IL Middleware + Auth | DONE — deployed at innerlab.ai | Yes |
| Phase 3: CWG Migration | DONE — on `test` branch (needs promotion to main) | Yes |
| Phase 4: FlowState Migration | DONE — live on production | Yes |
| Phase 5: Standalone Products (11) | DONE — all deployed and verified live | Yes (Chrome, 2026-03-28) |

---

## WHAT NEEDS TO HAPPEN NEXT

### Immediate (Owner Action)
1. ~~Add `JWT_PUBLIC_KEY` to child apps~~ — **DONE**
2. **Keep `JWT_SECRET`** on all services — HS256 fallback is still active (Phase 2 reverted)
3. **Create Stripe products** — 6 products, 12 prices (see FUTURE_WORK_TODO.md for table)
4. **Regenerate BTCPay API key** — current returns 403

### Next Session (Session 18)
5. ~~CWG GDPR endpoint~~ — **Already implemented** (found Session 13) — **Session 17: bugs found** (missing il_reflections, no source_module filter)
6. ~~FlowState GDPR endpoint~~ — **Already implemented** (found Session 13) — **Session 17: bugs found** (inconsistent user_id fields)
7. ~~#21 Test coverage expansion (billing + entitlements)~~ — **DONE Session 13** (40 new tests)
8. ~~Commit + push MBS test files and MBSPlatform doc updates~~ — **DONE Session 13**
9. ~~LazyChef Sonner migration~~ — **Already done** (commit `21881ba`)
10. ~~WildLens Sonner migration~~ — **Cleaned up Session 13** (commit `dfcb726`)
11. ~~TaskTracker CRA → Vite migration~~ — **DONE Session 14** (commit `0b75506`)
12. ~~Architecture docs for Codex onboarding~~ — **DONE Session 14** (4 docs created/updated)
13. ~~TaskTracker SSO redirect fix~~ — **DONE Session 15** (VITE_PRODUCT_DOMAIN prefix bug)
14. ~~TaskTracker Chrome verification~~ — **DONE Session 15** (8/8 standards PASS)
15. ~~Brand doc gap analysis + FlowState brief~~ — **DONE Session 15**
16. ~~Documentation consolidation (Marketing = single source of truth)~~ — **DONE Session 15**
17. ~~Architecture review + shared data decisions~~ — **DONE Session 17** (14 decisions confirmed, bugs found, docs updated)
18. **Fix CWG GDPR endpoint** — add il_reflections + il_activity_feed, add source_module filtering
19. **Fix il_reflections visibility default** — change from "private" to "shared"
20. **CWG refactor continuation** — finish remaining 11 files referencing cwg_journal_entries
21. CWG: merge `test` → `main` when ready (after refactor + GDPR fix)
22. More MBS Platform tests — admin, friends, promos, referrals (routes already analyzed, ready to write)
23. Per-app standards improvements (response helpers, input validation, rate limiting)
24. BrokenChain/MindHacker/FakeArtist SSO activation in Coolify
25. Consciousness DNS completion

### Future (Requires Migration Work)
15. LazyChef SSO migration — frontend must redirect to MBS Platform before `create_jwt_token` removal
16. RS256 Phase 2 redo — only after LazyChef SSO migration + verify all Coolify redeploys

---

## WHERE CODE GETS BUILT

| Folder | Domain | Database | Phase | Status |
|--------|--------|----------|-------|--------|
| `MBS/` | magicbusstudios.com | `mbs_platform` | 1 | DONE |
| `Innerlab/` | innerlab.ai | `inner_lab` | 2 | DONE |
| `CWG/` | conversationswithgod.ai | `inner_lab` (cwg_*) | 3 | DONE (test branch) |
| `YogaGhost/` | yoga.magicbusstudios.com | `inner_lab` (yoga_*) | 4 | DONE |
| `Brokenchain/` | brokenchain.magicbusstudios.com | `brokenchain` | 5 | DONE |
| `Mindhacker/` | mindhacker.magicbusstudios.com | `mindhacker` | 5 | DONE |
| `Trivia/` | triviaroast.magicbusstudios.com | `triviaroast` | 5 | DONE |
| `Fakeartist/` | fakeartist.magicbusstudios.com | `fakeartist` | 5 | DONE |
| `Whispering House/` | whisperinghouse.magicbusstudios.com | `whispering_house` | 5 | DONE |
| `Wildlife/` | wildlens.magicbusstudios.com | `wildlens` | 5 | DONE |
| `LazyChef/` | lazy-chef.magicbusstudios.com | `lazy_chef` | 5 | DONE |
| `Movie/` | moviepicker.magicbusstudios.com | `moviepicker` | 5 | DONE |
| `Shopping/` | smartcart.magicbusstudios.com | `smartcart` | 5 | DONE |
| `TaskTracker/` | tasktracker.magicbusstudios.com | `tasktracker` | 5 | DONE |
| `Tutor/` | tutor.magicbusstudios.com | `ai_tutor` | 5 | DONE |

---

## PREVIOUS SESSIONS

| Session | Date | Highlight |
|---------|------|-----------|
| 5 | March 27 | Phase 3B + Phase 4 reports reviewed |
| 4 | March 27 | Phase 3A complete, docs hardened, marketing updated |
| 3 | March 26 | Email/password auth + 2FA decisions, Inner Lab auth pages |
| 2 | March 26 | Full audit (50+ fixes), marketing briefs, orchestration guide |
| 1 | March 25 | Architecture finalized (17 decisions), three-layer design |
