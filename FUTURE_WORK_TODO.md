# FUTURE WORK TODO — MBS Platform

**Last Updated**: April 11, 2026 (Session 32)

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

### Email — Centralized Digest System
- [ ] **Centralize email digests at MBS Platform level**

**Current State (Session 30 audit):**
Inner Lab already has a working digest system: `digestCron.js` (node-cron, Mondays 9am UTC), `emailDigest.js` routes (preferences + preview), SendGrid integration, HTML email template with check-in stats, journal themes, archetype info. EmailPreference model stores `digest_enabled`, `frequency` (weekly/biweekly/monthly), and `email`.

**Architecture for Centralization:**
```
MBS Platform (Layer 1) owns:
├── Unified email preferences per user (marketing, digests, billing alerts, per-product)
├── SendGrid sender identity (noreply@magicbusstudios.com)
├── Master cron job that orchestrates all digest types
├── Unsubscribe endpoint (one-click, already exists: GET /api/auth/unsubscribe/:token)
└── Email preference UI at magicbusstudios.com/settings

Each product provides:
├── GET /api/digest-data — returns structured weekly summary (no HTML)
└── Product-specific aggregation logic stays in the product
```

**Implementation Steps:**
1. Add `GET /api/digest-data` to Inner Lab (extract `aggregateWeeklyData()` from existing digestCron.js)
2. Add `GET /api/digest-data` to Arcade apps (game stats, achievements) — simple
3. Add `GET /api/digest-data` to Studio Works apps (usage stats) — simple
4. Build MBS Platform cron job: iterates users with digest_enabled, calls each product's `/api/digest-data`, composes unified email
5. Migrate Inner Lab's EmailPreference to MBS Platform's existing EmailPreference model
6. Remove Inner Lab's standalone digestCron.js (keep routes for preview)

**Dependencies:** None — can be built anytime. Inner Lab digest keeps working as-is until migration.
**Effort:** Medium (2-3 sessions). Inner Lab's existing code is 80% of the work done.

### GDPR — Email Confirmation Before Data Deletion (ALL APPS)

- [ ] **Every "Delete My Data" action must send a confirmation email before deleting anything**

**Principle:** No data is deleted immediately. User clicks "Delete My Data" → receives email with confirmation link → clicks link → data is deleted. This protects against accidental deletion, unauthorized access, and provides an audit trail.

**Three Levels (same as existing deletion architecture):**

| Level | Where Triggered | What Happens |
|-------|----------------|--------------|
| **App-level** | Settings within each app (e.g., WildLens → "Delete my data") | App calls its own backend → backend sends confirmation email → user clicks link → app deletes its own data only |
| **Category-level** | magicbusstudios.com/settings | MBS Platform sends confirmation email → user clicks link → MBS calls each app's DELETE endpoint in that category |
| **Full account** | magicbusstudios.com/settings | MBS Platform sends confirmation email → user clicks link → MBS calls ALL apps' DELETE endpoints → deletes MBS user record |

**Implementation — MBS Platform (Layer 1):**
1. New model: `DataDeletionRequest` — `{ user_id, level ("app"/"category"/"full"), target (product slug or category name), confirmation_token, status ("pending"/"confirmed"/"expired"/"completed"), created_at, expires_at (24h), confirmed_at, completed_at }`
2. `DELETE /api/auth/account` — instead of deleting immediately, creates a DataDeletionRequest + sends confirmation email. Returns `{ success: true, message: "Confirmation email sent. Please check your inbox." }`
3. `DELETE /api/user-data/category/:category` — same pattern: creates request + sends email
4. `GET /api/user-data/confirm-delete/:token` — validates token, checks expiry, executes the actual deletion cascade, marks request as completed
5. Email template: "You requested to delete your [app/category/account] data. Click to confirm. This action is irreversible. Link expires in 24 hours."
6. If user doesn't confirm within 24 hours, request expires silently (no data deleted)

**Implementation — Each Child App (Layer 3):**
1. `DELETE /api/user-data` — instead of deleting immediately, calls MBS Platform to create a deletion request: `POST {PLATFORM_URL}/api/user-data/request-delete { product: MY_SLUG }`
2. MBS Platform sends the confirmation email (centralized — apps don't send their own)
3. When user confirms, MBS Platform calls the app's `DELETE /api/user-data/confirmed` (new endpoint) which actually deletes the data
4. Alternative simpler approach: app calls MBS Platform, MBS sends email, confirmation link goes to `magicbusstudios.com/confirm-delete/:token`, MBS calls app's existing `DELETE /api/user-data` after confirmation

**Implementation — Inner Lab (Layer 2):**
1. Same pattern as child apps — IL's delete endpoint creates a request via MBS Platform
2. IL module-level deletion: if user deletes just one IL module's data, that goes through MBS Platform confirmation too
3. IL sharing data (il_* collections): respects source_module filtering on confirmed deletion

**Frontend Changes (ALL apps):**
- "Delete My Data" button → shows modal: "We'll send a confirmation email to your registered address. You must click the link to complete deletion."
- After clicking: button changes to "Confirmation Sent ✓" (disabled state)
- No immediate deletion, no instant feedback that data was deleted

**Agent Prompts Needed:**
- MBS Platform agent: build DataDeletionRequest model, modify DELETE endpoints, add confirmation route, email template
- Each child app agent: modify DELETE /api/user-data to go through MBS confirmation flow
- Inner Lab agent: same as child apps, plus module-level deletion confirmation

**Dependencies:** SendGrid (already configured). MBS Platform email sending (already works for password reset, verification).
**Effort:** Medium (2-3 sessions). Pattern is identical to password reset flow (token → email → confirm → execute).

**Build Order:**
1. MBS Platform — DataDeletionRequest model + confirmation flow + email template
2. MBS Platform frontend — update Account settings page deletion UI
3. Inner Lab — update GDPR endpoint to use confirmation flow
4. CWG — update GDPR endpoint
5. FlowState — update GDPR endpoint
6. All 11 standalone apps — update GDPR endpoints (can be parallelized with agent prompts)

### Free Tier + Subscribe Page (Session 31-32 decisions, not yet built)

- [ ] **Add `free_tier` to Entitlement model type enum**
- [ ] **Build `POST /api/entitlements/subscribe-free`** — creates free_tier entitlement, handles IL category + module selections
- [ ] **Update `GET /api/entitlements/category/:cat`** to return `products[]` array with `registered: true/false`
- [ ] **Handle premium→free downgrade in Stripe webhook** — auto-create free_tier when subscription cancelled
- [ ] **Remove `freeTierLimits` from products.js** — MBS doesn't pass limits
- [ ] **Rename Billing → Subscribe** — `BillingPage.jsx` → `SubscribePage.jsx`, `/billing` → `/subscribe` (add redirect)
- [ ] **Build `/subscribe/innerlab` dedicated page** — SubscribeInnerLabPage.jsx with IL module picker
- [ ] **Redesign main subscribe page** — free vs premium per category

### Module Coolify Instruction Reviews (Session 32 — in progress)

Domain convention decided (2026-04-10): all new IL modules → `{slug}.innerlab.ai`. See `MODULE_REVIEW_CHECKLIST.md` for full checklist and tracking.

**Reviewed (5/10):**
- [x] Rituals — v2 approved ✅
- [ ] StarMap — correction sent (twice), awaiting v2
- [ ] InnerQuest — correction sent, awaiting v2
- [ ] LifeMap — correction sent, awaiting v2
- [ ] Archetypes — correction sent, awaiting v2

**Not yet reviewed (5/10):**
- [ ] Bonds
- [ ] AstroCompass
- [ ] Arcana
- [ ] DreamLens
- [ ] Nexus

### Module Alignment (Session 30 (continued) — prompts ready, not yet executed)

- [ ] **Give INNERLAB_MODULE_ALIGNMENT.md to 10 new module agents** (Bonds through Nexus)
- [ ] **Give CWG_ALIGNMENT.md to CWG agent** — AFTER feature removal is complete
- [ ] **Give FLOWSTATE_ALIGNMENT.md to FlowState agent** — includes GDPR singleton bug fix
- [ ] **Verify FlowState GDPR bug** — does DELETE /api/user-data delete il_user_wellness_profiles? If yes, fix (identity singleton)

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
- [x] #21 Test coverage expansion — billing + entitlements done (Session 13, 40 new tests). Frontend tests: 52 tests across 6 suites (Session 30, commit `8aa813c`). BillingPage, AdminPage, OnboardingModal added.

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
- [x] Inner Lab consciousness assessment E2E — PASS ("The Quiet Sage", data saved to il_consciousness_profiles.assessment_data)
- [x] FlowState dev redeploy — Yoga D redeployed, container healthy
- [x] FlowState dev → main merge — 5 commits fast-forward, pushed to origin
- [x] FlowState production redeploy — Yoga P redeployed, il_* integration live on yoga.magicbusstudios.com
- [x] DreamLens Codex review — 4/5 violations fixed (V1 Winston, V2 rate limiting, V4 GDPR, V5 tests). V3 express-validator open (uses Zod instead).
- [x] dream-service.ts — exists, no restoration needed
- [x] FlowState test suite — 29 tests (3→29), commit `795b322`
- [x] MindHacker test suite — 32 tests, commit `9f1b0ae`
- [x] Trivia Roast test suite — 41 tests
- [x] TaskTracker test suite — 38 tests (mocked DB/Redis)
- [x] CWG tests verified — 66 pre-existing (pytest + httpx), no work needed
- [x] Inner Lab vision roadmap — 12 features (3 tiers) appended to Innerlab/FUTURE_WORK_TODO.md
- [x] Inner Lab marketing brief — updated InnerLab_Product_Brief.md in Marketing/Brand Overview/
- [x] Skipped test suites for Fake Artist, Whispering House, WildLens (user decision — low usage)

### Session 23 Items (delegated to per-project agents)
- [x] Inner Lab Daily Briefing page (innerlab.ai/daily) — **COMPLETE**. DailyBriefingPage.jsx + dailyBriefing.js (archetype-based intentions, time-of-day awareness). Wired at `/daily`. Verified Session 30.
- [ ] Cross-module "Continue with..." suggestions — delegated to Inner Lab agent
- [x] Weekly Consciousness Review page (innerlab.ai/weekly) — **COMPLETE**. WeeklyConsciousnessPage.jsx (378 lines) + weeklyReview.js. Full aggregation: streak, sparklines, week-over-week stats, dimension changes, module breakdown, journal themes. Tested in session23.test.js. Verified Session 30.
- [x] Consciousness assessment Full mode (16 questions) — **COMPLETE**. ConsciousnessPage.jsx (655 lines) + consciousnessAssessment.js (1136 lines). Quick (8Q, ~3 min) and Full (16Q, ~10 min) both implemented. 12 archetypes, 7 dimensions, results view with evolution chart. Verified Session 30.
- [x] DreamLens V3 decision — **RESOLVED**: Zod accepted as valid for next-gen TypeScript builds. Express-validator remains standard for existing JS apps. Removed from backlog Session 30.

### Session 29 Completed (April 7, 2026)
- [x] MBS git corruption fix — index reset + nginx.conf restored (OneDrive dehydration)
- [x] MBS_PLATFORM_URL verified — CWG prod correct, dev fixed by owner in Coolify
- [x] Product catalog (products.js) — Bonds + LifeMap added, BreathArc removed (12 IL modules). Commit `ba72201`.
- [x] SSO redirect improvements — auto-allow product catalog URLs, subdomain trust, unified performRedirect. Commit `ba72201`.
- [x] Account dropdown with Sign Out — "My Account" + "Sign Out" in nav header. Chrome-verified live. Commit `95c4ab6`.
- [x] GDPR cascade test — full account deletion verified end-to-end. `DELETE /api/auth/account` → 200. JWT invalidated.
- [x] MBS frontend test suite — 19 tests (Vitest + React Testing Library): AuthContext (8), AuthButton (5), ProtectedRoute (6). Commit `12a260f`.
- [x] BreathArc removal — cleaned from products.js + all 6 CLAUDE.md files + platform-instructions
- [x] MSI conflict cleanup — 6 OneDrive conflict files + billing-Nitro.js deleted
- [x] Doc updates — all CLAUDE.md files updated to 12 IL modules (global, desktop, MBSPlatform, innerlab instructions, mbs instructions, new-modules template)

### Session 30 Completed (April 7, 2026)
- [x] MBS frontend test expansion — 52 tests across 6 suites (was 19). Added BillingPage (8), AdminPage (8), OnboardingModal (12). Fixed vitest config (jsdom env, setupTests with IntersectionObserver/ResizeObserver polyfills, @testing-library/jest-dom).
- [x] Centralized email digest architecture — detailed implementation plan written (6-step migration from Inner Lab's existing digestCron.js to MBS Platform orchestration)
- [x] User dashboard architecture — MyProductsPage.jsx spec with entitlement cards, billing section, Stripe portal integration. All APIs already exist.
- [x] Long-term features architecture — win-back offers, email campaigns, family plan, teams. Detailed specs with dependencies and effort estimates.
- [x] DreamLens V3 resolved — Zod accepted as valid for next-gen TypeScript builds. Express-validator remains standard for existing JS apps.
- [x] Daily Briefing — verified complete (DailyBriefingPage.jsx + dailyBriefing.js, time-of-day awareness, archetype intentions)
- [x] Weekly Consciousness Review — verified complete (378-line page, sparklines, week-over-week, dimension changes, tested)
- [x] Consciousness Full Assessment — verified complete (655-line page, Quick 8Q + Full 16Q, 12 archetypes, 7 dimensions)
- [x] CWG Dev B redeploy verification — **PASS**. Backend healthy (`mbs_platform` mode), 21 guides load, MBS Platform API reachable, SSO flow correct (→ innerlab.ai/auth/login → api.magicbusstudios.com)
- [x] Per-app validation + rate limiting audit — **14/14 rate limited, 12/14 validated**. Gaps: Trivia Roast (3 POST routes), Tutor (2 raw JSON routes), Fake Artist (low risk)

### Post-Phase 5 Cleanup — All Complete
Dead code cleanup, GDPR endpoints (all 8 apps), MBS deletion UI + cascade, CWG entitlements, TaskTracker transactions. See CHANGELOG.md Session 8.

### Decided & Built — Session 8
Subscribe gating, product picker, CWG entitlements, friends consolidation, admin dashboard, free trial (7 days). All built and deployed. See CHANGELOG.md Session 8.

### Completed — Session 8 Enhancements
AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin. See CHANGELOG.md Session 8.

### Future Work (Decided but Deferred)
- [x] FlowState il_* integration — **DONE Sessions 19+21.** FlowState now writes to 3 il_* collections: il_activity_feed (S19 `cc65a40`), il_check_ins + il_user_wellness_profiles (S21 `e8a3c79`). Remaining: il_reflections (no journaling feature), il_user_memories (no AI insight extraction).
- [x] FlowState GDPR user_id fix — **RESOLVED Session 19.** Verified NOT a bug — yoga_activity uses `userId` consistently in writes and deletes. GDPR also updated to delete il_activity_feed by source_module.
- [ ] **Rename "Billing" → "Subscription" across MBS frontend** — BillingPage.jsx → SubscriptionPage.jsx, /billing → /subscription, nav links, page titles, button labels. Keep API routes as `/api/billing/*` (backend unchanged). Update IndividualPlansPage references too.
- [ ] Stripe subscription portal testing — "Manage Subscription" button calls `/api/billing/portal`. Test full upgrade/downgrade/cancel flow once Stripe products are created.
- [ ] BTCPay expiry reminders — Lightning is a 30-day one-time pass with manual renewal. Add email reminders when the 30 days are about to expire.
- [ ] Win-back offers — needs centralized email digest + promo system working together. See architecture below.
- [ ] User dashboard (My Products, billing history) — See architecture below.
- [ ] Email campaigns + announcements + newsletter — See architecture below.
- [ ] Multi-currency, family plan, teams — long-term, see architecture below.

### User Dashboard Architecture (Session 30 Planning)

**What it is:** A "My Products" page at `magicbusstudios.com/dashboard` (or `/my-products`) showing:
- All products the user has access to (from entitlements)
- Active subscriptions with renewal dates
- Billing history (from transactions collection)
- Quick-launch links to each product
- Subscription management (upgrade/downgrade/cancel via Stripe portal)

**Current State:**
- Entitlements API exists: `GET /api/entitlements` returns all user entitlements
- Billing history exists: `GET /api/billing/history` returns transactions
- Stripe portal exists: `POST /api/billing/portal` creates a portal session
- AccountPage.jsx exists but is profile-focused (name, avatar, email prefs, connected accounts)
- No dedicated "My Products" or "My Subscriptions" view

**Implementation:**
```
Frontend: MyProductsPage.jsx
├── Product grid — card per entitlement (icon, name, status, expiry)
│   ├── Active: green badge, "Open" button → product URL
│   ├── Trial: yellow badge, days remaining, "Upgrade" button
│   └── Expired: gray badge, "Renew" button
├── Billing section
│   ├── Current plan summary (bundle vs individual)
│   ├── "Manage Subscription" → Stripe portal
│   └── Transaction history table (date, product, amount, status)
└── Quick actions: "Explore Products" → /billing, "Get Support" → /contact

Backend: No new routes needed — uses existing entitlements + billing APIs
```

**Dependencies:** Stripe products must be created first (owner action). Without real price IDs, the subscription data is empty.
**Effort:** Small (1 session). Pure frontend — all APIs exist.

### Win-Back & Email Campaigns Architecture (Session 30 Planning)

**Win-back offers:**
- Trigger: user's subscription expires or trial ends without conversion
- MBS Platform detects via Stripe `customer.subscription.deleted` webhook (already wired)
- After N days (7? 14?), send win-back email with promo code
- Promo system already exists: `POST /api/promos`, `POST /api/promos/validate`
- Implementation: add a scheduled job that queries expired entitlements, generates time-limited promo codes, sends via SendGrid

**Email campaigns + announcements:**
- Announcement model already exists in mbs_platform DB
- Admin can create announcements (title, message, type, target, valid_from, valid_until)
- Missing: email blast capability — send announcement as email to segment
- Implementation: admin route `POST /api/admin/campaigns/send` that queries users by segment (all, category, product) and sends via SendGrid
- Unified unsubscribe already works

**Newsletter:**
- Inner Lab already has a blog system with `blogDraftCron.js` (auto-generates blog drafts Mon/Thu)
- Newsletter = curated blog posts + product updates → SendGrid campaign
- Low priority — blog content exists, distribution channel doesn't

### Long-Term Features (Session 30 Planning)

**Multi-currency:**
- Stripe supports multi-currency natively
- BTCPay is already in sats
- Implementation: add `currency` field to checkout, let Stripe handle conversion
- Low effort but needs pricing strategy decision per currency

**Family plan:**
- New entitlement type: `family_plan` with `max_members` field
- New model: FamilyGroup { owner_id, members: [userId], plan_entitlement_id }
- Owner invites members, members get `mbs_all_access` while plan is active
- Billing: single subscription billed to owner at family rate
- Effort: Medium (new model, new routes, invitation flow)

**Teams:**
- Similar to family but with admin controls: team admin can assign product access per member
- New models: Team, TeamMember, TeamEntitlement
- Integration with Stripe: team billing, seat-based pricing
- Effort: Large (significant new functionality)

Both family and teams are post-revenue features — build only after individual subscriptions prove product-market fit.

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
- [ ] Per-app input validation + rate limiting — **Session 30 audit complete**:
  - Rate limiting: **14/14 compliant** (all apps have it installed AND wired)
  - Input validation: **12/14 compliant**. Gaps:
    - **Trivia Roast**: Missing express-validator on 3 POST routes (`/api/chat`, `/games`, `/leaderboard`)
    - **Tutor**: 2 routes use raw `request.json()` instead of Pydantic (`toggle-day`, `chat/summarize`)
    - **Fake Artist**: No express-validator but low risk (Socket.io game, minimal REST)
  - SmartCart (Shopping/) has 29 tests, was missed by agent due to folder name

### Resolved Since Audit (Sessions 8-12)
- [x] MBS P0 logger crash — Winston logger + gdprCascade fix (Session 8)
- [x] MBS response helpers — all 14 route files converted (Session 10)
- [x] MBS centralized error handler (Session 8)
- [x] GDPR endpoints — all 15 apps confirmed implemented (Session 8 + Session 13 doc fix)
- [x] RS256 JWT upgrade — dual-mode deployed to all 15 apps (Session 11)
- [ ] RS256 Phase 2 — attempted and **reverted** (Session 12). All 15 apps back to dual-mode.
- [x] LazyChef self-issued auth — **RESOLVED Session 20.** SSO migration complete, `create_jwt_token` removed (commit `2505f1b`).
