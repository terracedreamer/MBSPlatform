# FUTURE WORK TODO — MBS Platform

**Last Updated**: April 5, 2026 (Session 22)

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

- [x] CWG pricing: $15/mo, $135/yr (updated Session 9) + 21K sats/mo, 126K sats/yr (Lightning)
- [x] Inner Lab All Access: $20/mo, $180/yr (updated Session 9)
- [x] MBS All Access: $30/mo, $270/yr (updated Session 9)
- [x] Structured billing page design (category tabs → product → plans)
- [x] Lightning as equal payment option alongside Stripe
- [x] FlowState pricing — $5/mo, $45/yr (Session 9)
- [x] Arcade pricing — $10/mo bundle, $5/mo individual (Session 9)
- [x] Studio Works pricing — $10/mo bundle, $5/mo individual (Session 9)
- [ ] Create actual Stripe products in Dashboard (OWNER ACTION — see Stripe price table below in Reminders section)

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
- [x] #16 JWT upgrade to RS256 — **COMPLETED Session 11**. All 15 repos upgraded, all Coolify env vars configured, verified working.
- [x] #21 Test coverage expansion — billing + entitlements done (Session 13, 40 new tests). Frontend tests still pending.

### Completed — Session 11 (RS256 JWT Upgrade)
- [x] **#16 RS256 JWT upgrade** — All 15 repos upgraded + all Coolify env vars configured. MBS Platform signs RS256, all child apps verify RS256→HS256 dual-mode. Fully deployed and verified.

### Session 12 — RS256 Phase 2 Attempted and Reverted
- [x] RS256 Phase 2 attempted — HS256 fallback removed from all 15 apps
- [x] Chrome verification found issues — Coolify hadn't redeployed some apps, LazyChef frontend needs SSO migration first
- [x] **Reverted** — all 15 repos back to dual-mode (RS256 first, HS256 fallback)
- [ ] RS256 Phase 2 (future) — requires: verify all Coolify services have redeployed, Chrome-verify each app. LazyChef SSO blocker RESOLVED (Session 20).
- [x] LazyChef SSO migration — **DONE Session 20** (commit `2505f1b`). Dead local auth removed, SSO redirect verified via Chrome.

### Session 13 — Completed
- [x] #21 Test coverage expansion — billing.test.js (19 tests) + entitlements.test.js (21 tests) = 40 new tests. Total: 61.
- [x] GDPR documentation fix — CWG + FlowState were already implemented, docs were stale

### Session 14 Completed
- [x] ~~Commit + push MBS test files and MBSPlatform doc updates~~ — DONE Session 13
- [x] TaskTracker CRA → Vite migration — DONE Session 14 (commit `0b75506`)
- [x] Architecture docs for Codex onboarding — DONE Session 14 (4 docs)

### Session 15 Completed
- [x] Brand documentation gap analysis — FlowState brief created, Consciousness layer added to master doc + InnerLab brief
- [x] Documentation consolidation — Marketing folder is now single source of truth, duplicates deleted from MBSPlatform
- [x] TaskTracker SSO redirect fix — `VITE_PRODUCT_DOMAIN` had extra `https://` prefix causing double protocol
- [x] TaskTracker Chrome verification — 8/8 standards PASS, all 6 API routes returning 200

### Session 16 Completed
- [x] MBS Platform tests verified — 107 tests across 7 suites (admin, friends, promos, referrals already written)
- [x] BrokenChain/MindHacker/FakeArtist SSO — all 3 verified working via Chrome
- [x] Inner Lab platform build — 8 dashboard pages, 13 backend route files, 13 il_* collections (Session 18: added il_birth_profiles)
- [x] il_reflections shared journal — model + CRUD API + Journal page + first-class module fields
- [x] Notification routes + bell UI, Wellness routes + page, Check-In History, Weekly Review
- [x] Dashboard enhancements — state-aware greeting, continue-where-left-off, sparkline, memory resurfacing
- [x] Incubator cross-reference review — 30 questions + 7 follow-ups answered in doc 28
- [x] BreathArc provisionally removed (FlowState keeps yoga+breathwork+meditation)
- [x] CWG partial refactor — journal_routes.py, journal_insights_routes.py, chat_routes.py refactored to il_reflections

### Session 17 Completed (Architecture Review + Visibility Fix)
- [x] Full architecture review across CWG, Inner Lab, FlowState, MBS Platform codebases
- [x] 14 architecture decisions confirmed (shared data, GDPR scoping, sharing toggle, dashboard access)
- [x] Critical bugs documented: CWG GDPR missing collections + no source_module filter, FlowState zero il_* writes, FlowState GDPR user_id inconsistency
- [x] il_reflections visibility default fixed: "private" → "shared" + smart GET filtering (Innerlab commit `117d99b`)
- [x] 20+ documentation files updated across Marketing/Architecture Docs, Innerlab, CWG, YogaGhost, global rules, Incubator doc 28, global + MBS CLAUDE.md
- [x] Created `Inner_Lab_GDPR_Deletion_Scoping.md` architecture doc
- [x] Created `Innerlab/docs/session-17-architecture-decisions.md` + `sharing-toggle-implementation.md`
- [x] Fixed 14 stale architecture doc path references across all project CLAUDE.md files
- [x] Cleaned up 26 OneDrive Nitro/MSI sync conflict files (all compared and verified safe to delete)

### Session 18 Completed
- [x] CWG refactor: all 17 backend files migrated from cwg_journal_entries to il_reflections — **DONE** (commit `88b4e11`)
- [x] Fix CWG GDPR endpoint: il_reflections + il_activity_feed added, source_module filtering, singletons removed — **DONE**
- [x] Fix il_reflections Mongoose model: change visibility default from "private" to "shared" — **DONE Session 17**
- [x] Add `visibility: "shared"` filter to Inner Lab dashboard Journal page queries — **DONE Session 17**
- [x] CWG: mood/include_in_profile moved to first-class fields (not under metadata) — **DONE**
- [x] CWG: personal history sync to il_personal_histories (dual-write) — **DONE**
- [x] CWG: identity data migration script created (`scripts/migrate_identity_to_il.py`) — **DONE**
- [x] Fixed journal_insights_routes.py metadata.mood bug, insights_routes.py moods typo, chat_routes.py metadata.include_in_profile — **DONE**

### Session 19 Completed (April 4, 2026)
- [x] CWG: run migration scripts — journals (11 in il_reflections), identity (0 to migrate)
- [x] CWG: verify journal, consciousness, personal history — all API calls 200
- [x] Inner Lab: verify Journal page shows CWG data — confirmed via Chrome
- [x] Inner Lab: Birth Profile dashboard UI — live at innerlab.ai/birth-profile
- [x] Inner Lab: Sharing toggle — model, routes, page, GDPR (14 collections)
- [x] Inner Lab: Dashboard nav — Birth Profile + Sharing cards
- [x] Inner Lab: Response helpers migration — all 14 route files
- [x] Inner Lab: Test suites — 33 tests across 2 suites
- [x] CWG: Bidirectional sync — reads identity from il_* first
- [x] FlowState: il_activity_feed integration — writes on session completion
- [x] FlowState: GDPR user_id verified — NOT a bug
- [x] Codex review — DreamLens DB confirmed, architecture summary for Codex
- [x] env-standards.md — DB naming table restructured
- [x] Codex Review Guide — created CODEX_REVIEW_GUIDE.md
- [ ] CWG: merge `test` → `development` (requires passphrase + env var audit, deferred)
- [ ] Consciousness DNS completion

### Session 20 Completed (April 4, 2026)
- [x] LazyChef SSO frontend migration — dead LoginPage/SignupPage deleted, backend local auth removed (~1,187 lines), create_jwt_token removed. Chrome verified SSO redirect. (commit `2505f1b`)
- [x] Rich Consciousness Profile UI in Inner Lab — 12 archetypes, 16 questions, 7 dimensions, Quick/Full assessment wizard with type reveal (commit `0fd0420`)
- [x] Rich Personal History (My Story) UI in Inner Lab — 8-section guided questionnaire, Quick (5) / Full (8) modes, section editor + view (commits `808c792`, `45ed8c0`)
- [x] Inner Lab OG image — 1200x630 PNG via Canva MCP, dark theme with teal accents (commit `35b7b5f`)
- [x] CWG Settings page crash fix — root cause: missing `Lock` icon import (commit `46113e4`)
- [x] YogaGhost FUTURE_WORK_TODO.md — marked Session 19 il_activity_feed + GDPR items done (commit `ce6f4f2`)
- [x] CWG entitlement enforcement — Chrome-verified (admin access, 21 guides, profile API 200)
- [x] Inner Lab RS256 auth — Chrome-verified (`GET /api/consciousness` returns 200 with JWT)
- [x] Response helpers audit — all 15 apps compliant (11 Express + 4 FastAPI)
- [x] Inner Lab OG/Twitter meta tags — added to index.html
- [x] Inner Lab CLAUDE.md — updated with 13 dashboard pages, /consciousness + /my-story
- [x] YogaGhost friends/invites — verified NOT orphaned (active social feature)
- [x] Cleaned up 32 Nitro conflict files from Movie/ and Shopping/

### Session 21 Completed (April 4, 2026)
- [x] Per-app test suites — MoviePicker (78 tests), SmartCart (29 tests), BrokenChain (33 tests). All passing.
- [x] Per-app input validation + rate limiting audit — all 3 projects already compliant (express-validator + express-rate-limit)
- [x] DreamLens Codex review — 5 violations found, feedback in doc 28 Round 5 + `CODEX_FEEDBACK_DREAMLENS.md`
- [x] Architecture evolution standards — `architecture-evolution.md` rules file, 3 bootstrap skills updated, global CLAUDE.md updated
- [x] FlowState il_* integration — il_check_ins + il_user_wellness_profiles added (commit `e8a3c79`). GDPR updated to 10 collections. FlowState now writes 3 il_* collections.
- [ ] RS256 Phase 2 — deferred by user decision, keep HS256 fallback for now

### Session 22 Completed (April 5, 2026)
- [x] End-to-end test of Inner Lab consciousness assessment via Chrome — PASS (Quick Assessment, "The Quiet Sage", data saved to il_consciousness_profiles.assessment_data)
- [x] FlowState dev redeploy — Yoga D redeployed on Coolify, commit e8a3c79 live, container healthy
- [x] DreamLens Codex review — 4/5 violations fixed (V1 Winston, V2 rate limiting, V4 GDPR, V5 tests). V3 express-validator still open.
- [x] dream-service.ts — exists, no restoration needed
- [x] FlowState test suite — 29 tests (3→29), commit `795b322` on `dev`

### Session 23 Items
- [ ] More test suites — CWG, MindHacker, Trivia Roast, Fake Artist, Whispering House, WildLens, TaskTracker
- [ ] DreamLens V3 follow-up — express-validator still missing
- [ ] FlowState merge dev → main when ready
- [ ] Consciousness assessment Full mode (16 questions) E2E test

### Post-Phase 5 Cleanup — All Complete
Dead code cleanup, GDPR endpoints (all 8 apps), MBS deletion UI + cascade, CWG entitlements, TaskTracker transactions. See CHANGELOG.md Session 8.

### Decided & Built — Session 8
Subscribe gating, product picker, CWG entitlements, friends consolidation, admin dashboard, free trial (7 days). All built and deployed. See CHANGELOG.md Session 8.

### Completed — Session 8 Enhancements
AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin. See CHANGELOG.md Session 8.

### Future Work (Decided but Deferred)
- [x] FlowState il_* integration — **DONE Sessions 19+21.** FlowState now writes to 3 il_* collections: il_activity_feed (S19 `cc65a40`), il_check_ins + il_user_wellness_profiles (S21 `e8a3c79`). Remaining: il_reflections (no journaling feature), il_user_memories (no AI insight extraction).
- [x] FlowState GDPR user_id fix — **RESOLVED Session 19.** Verified NOT a bug — yoga_activity uses `userId` consistently in writes and deletes. GDPR also updated to delete il_activity_feed by source_module.
- [ ] Stripe subscription portal testing — "Manage Subscription" button calls `/api/billing/portal`. Test full upgrade/downgrade/cancel flow once Stripe products are created.
- [ ] BTCPay expiry reminders — Lightning is a 30-day one-time pass with manual renewal. Add email reminders when the 30 days are about to expire.
- [ ] Win-back offers — needs email + promo system working together
- [ ] User dashboard (My Products, billing history) — needs pricing + real subscriptions
- [ ] Email campaigns + announcements + newsletter — post-launch
- [ ] Multi-currency, family plan, teams — long-term

---

## Standards Compliance

### Audit History
Full audit reports in `audits/`:
- `2026-03-29-compliance-audit.md` — 17-project audit (62% compliant, 78 items remaining at time of audit)
- `2026-03-29-arcade-brand-audit.md` — Brand DNA audit of 5 Arcade games
- `2026-03-30-platform-review.md` — SSO, entitlements, GDPR, billing verification against actual code

### Remaining Per-App Items (check each project's FUTURE_WORK_TODO.md)
- [x] CWG — GDPR `DELETE /api/user-data` endpoint (fixed Session 18 — 6 il_* activity collections filtered by source_module, singletons excluded)
- [x] FlowState (YogaGhost) — GDPR `DELETE /api/user-data` endpoint (implemented — 7 yoga_* collections)
- [x] LazyChef — react-hot-toast → Sonner migration (already completed, commit `21881ba`)
- [x] WildLens — ToastContext → Sonner migration (orphaned files deleted Session 13, commit `dfcb726`)
- [x] TaskTracker — CRA → Vite migration (Session 14) + Chrome verified (Session 15)
- [x] Test suites — MoviePicker (78), SmartCart (29), BrokenChain (33) added Session 21. FlowState (29), MindHacker (32), Trivia Roast (41), TaskTracker (38) added Session 22. CWG (66) already had tests. LazyChef, AI Tutor, Inner Lab (33), MBS Platform (107) already had tests. Remaining: Fake Artist, Whispering House, WildLens (3 of 15).
- [x] Per-app response helpers — **ALL 15 APPS COMPLIANT** (Session 20 audit: 11 Express with extracted `responseHelpers.js`, 4 FastAPI with inline `{"success": True}` pattern)
- [ ] Per-app input validation + rate limiting — see individual project TODOs

### Resolved Since Audit (Sessions 8-12)
- [x] MBS P0 logger crash — Winston logger + gdprCascade fix (Session 8)
- [x] MBS response helpers — all 14 route files converted (Session 10)
- [x] MBS centralized error handler (Session 8)
- [x] GDPR endpoints — all 15 apps confirmed implemented (Session 8 + Session 13 doc fix)
- [x] RS256 JWT upgrade — dual-mode deployed to all 15 apps (Session 11)
- [ ] RS256 Phase 2 — attempted and **reverted** (Session 12). All 15 apps back to dual-mode.
- [x] LazyChef self-issued auth — **RESOLVED Session 20.** SSO migration complete, `create_jwt_token` removed (commit `2505f1b`).
