# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: April 11, 2026 (Session 32)

## Repo Purpose: MBS Ecosystem Documentation

This repo contains architecture decisions, migration plans, reference files, audits, and project status tracking for the entire MBS ecosystem. No code is built here. Code gets built in the actual project folders. Claude Code infrastructure (skills, rules, tasks) is tracked separately in `~/Desktop/Codes/Claude Setup/`.

## Phase Status

| Phase | Status | Next Action |
|-------|--------|-------------|
| Phase 1: MBS Platform | **ALL DONE — Addendum #1-15 complete** (deployed at magicbusstudios.com) | Nothing |
| Phase 2: IL Middleware + Auth | **FULLY COMPLETE** (api.innerlab.ai + innerlab.ai/auth/*) | Nothing |
| Phase 3A: CWG Migration Script | **DONE** — 20 users migrated, 14 collections copied | Nothing |
| Phase 3B: CWG Refactor | **COMPLETE** — running on `test` branch, needs promotion to `dev` | Promote test → dev when ready |
| Phase 4: FlowState Migration | **COMPLETE** — live on production | Nothing |
| Phase 5: Standalone Products | **ALL 11 COMPLETE — deployed and verified live** | Dead code cleanup (optional) |

## All 11 Standalone Products — Verified Live (2026-03-28)

| Product | Category | Domain | SSO Status |
|---------|----------|--------|-----------|
| BrokenChain | arcade | brokenchain.magicbusstudios.com | Authenticated (Abhinav Gupta) |
| MindHacker | arcade | mindhacker.magicbusstudios.com | SSO ready (Sign In button) |
| Trivia Roast | arcade | triviaroast.magicbusstudios.com | Authenticated |
| Fake Artist | arcade | fakeartist.magicbusstudios.com | SSO ready (Sign In button) |
| Whispering House | arcade | whisperinghouse.magicbusstudios.com | Authenticated + Sign Out verified |
| WildLens | studioworks | wildlens.magicbusstudios.com | Authenticated (100 XP, Naturalist) |
| Lazy Chef | studioworks | lazy-chef.magicbusstudios.com | Authenticated + live verified |
| Movie Picker | studioworks | moviepicker.magicbusstudios.com | Authenticated + live verified |
| SmartCart | studioworks | smartcart.magicbusstudios.com | Authenticated (My Lists loaded) |
| TaskTracker | studioworks | tasktracker.magicbusstudios.com | ✅ Authenticated — Chrome verified Session 15 (all 6 API routes 200, Vite build confirmed) |
| AI Tutor | studioworks | tutor.magicbusstudios.com | Authenticated (dashboard, 50 XP) |

## Known Issues

| Issue | Impact | Resolution |
|-------|--------|-----------|
| BTCPay API key 403 | Lightning payments fail | Regenerate API key with full store permissions in BTCPay |
| Stripe bundle price IDs | Checkout buttons fail for all 6 products | Create products/prices in Stripe Dashboard (see FUTURE_WORK_TODO.md) |
| RS256 JWT upgrade | ✅ Session 11: RS256 signing live, all 15 apps have dual-mode (RS256→HS256). Session 12: Phase 2 attempted + reverted — back to dual-mode. Session 20: LazyChef SSO blocker resolved. | Dual-mode is the safe steady state. Phase 2 can proceed — LazyChef SSO migration done. |
| CWG entitlement enforcement | check_entitlement() wired on `test` branch (`f9c38ab`). Session 20: Chrome-verified admin access works (21 guides, profile API 200). | Full enforcement testing needs non-admin test account |
| CWG on `test` branch | Running on test, not main — intentional. Session 19: migration scripts RUN, bidirectional sync ADDED (CWG reads il_* first). All verified via Chrome. Session 29: MBS_PLATFORM_URL fixed on dev. | Promote test → dev when ready (requires passphrase + env var audit) |
| ~~CWG Settings page crash~~ | ✅ RESOLVED Session 20 — Root cause: missing `Lock` icon import from lucide-react. React tried `new undefined()`. Fixed in commit `46113e4` on `test`. | Fixed |
| ~~CWG → Inner Lab data migration~~ | ✅ RESOLVED Session 19 — Migration scripts run. Journals: 11 entries in il_reflections. Identity: 0 to migrate (dual-write handles future). | Done |
| ~~CWG GDPR missing collections~~ | ✅ RESOLVED Session 18 — il_reflections + il_activity_feed added to deletion scope | Fixed in commit `88b4e11` |
| ~~CWG GDPR no source_module filter~~ | ✅ RESOLVED Session 18 — all il_* deletes now filter by `source_module: "cwg"`, identity singletons removed from app-level deletion | Fixed in commit `88b4e11` |
| ~~FlowState zero il_* writes~~ | ✅ RESOLVED Session 19+21 — FlowState now writes to 3 il_* collections: il_activity_feed (S19), il_check_ins + il_user_wellness_profiles (S21, commit `e8a3c79`) | Done |
| ~~FlowState GDPR user_id inconsistency~~ | ✅ RESOLVED Session 19 — Verified NOT a bug. yoga_activity uses `userId` in both writes and deletes (field names match). Style inconsistency only. GDPR also updated to delete il_activity_feed entries. | Done |
| il_reflections visibility default wrong | ✅ RESOLVED — Mongoose model default changed to "shared", route create fallback changed to "shared", GET route now defaults to visibility: "shared" for unified dashboard view (Session 17) | Fixed in Reflection.js + reflections.js |
| TaskTracker VITE_PRODUCT_DOMAIN | ✅ RESOLVED — had `https://` prefix causing double protocol in redirect URL | Fixed Session 15 — removed prefix from Coolify build arg |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website + Platform | **Deployed — Phase 1 complete. Session 31: free_tier entitlements, SubscribePage + SubscribeInnerLabPage, GDPR email confirmation, coming soon seed. Pushed to main + development, awaiting Coolify redeploy.** | magicbusstudios.com |
| Inner Lab website + Middleware + Auth | **Deployed — Session 20: 13 dashboard pages, OG image live, 14 il_* collections, 15 route files, 33 tests, all 14 routes use response helpers** | innerlab.ai / api.innerlab.ai |
| CWG | **Migrated** — running on `test` branch | conversationswithgod.ai |
| FlowState | **Migrated** — live on production. Session 22: dev merged to main, production redeployed. il_check_ins + il_user_wellness_profiles + il_activity_feed now live. | yoga.magicbusstudios.com |
| Arcade games (5) | **All 5 SSO migrated and deployed** | *.magicbusstudios.com |
| Studio Works tools (6) | **All 6 SSO migrated and deployed** | *.magicbusstudios.com |

## Phase 5 Learnings (Cascaded to Instructions)

Two critical SSO bugs discovered and documented:
1. **Legacy user collision** — existing users with different `_id` than platform `userId` crash on unique email index. Fix pattern added to `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` Step 5.
2. **Python JWT_SECRET_KEY vs JWT_SECRET** — Python config reads wrong env var name. Warning added to CWG instructions.

Both fixes confirmed working across all affected apps.

## Pre-Build Checklist

- [x] All phases 1-5 complete
- [x] All 11 standalone products verified live via Chrome (2026-03-28)
- [x] Platform-instructions synced to all 15 project folders
- [x] GDPR cascade deployed and verified (Session 8)
- [x] RS256 JWT upgrade — all 15 services configured with keys, verified working (Session 11)
- [x] CWG GDPR endpoint — implemented + fixed Session 18 (commit `88b4e11`). Now filters 6 il_* activity collections by `source_module: "cwg"`, excludes identity singletons.
- [x] FlowState GDPR endpoint — already implemented in `index.js` (7 yoga_* collections). Docs were stale.
- [ ] RS256 Phase 2 — attempted Session 12, **reverted**. All 15 apps back to dual-mode. LazyChef SSO blocker resolved Session 20 — can proceed.
- [x] LazyChef SSO migration — **DONE Session 20** (commit `2505f1b`). Dead local auth removed, SSO redirect verified via Chrome.
- [x] Inner Lab platform interface — 13 dashboard pages, 15 route files, 14 il_* collections (Session 20: consciousness assessment wizard, personal history questionnaire, OG image, my-story nav)
- [x] il_reflections — shared journal store with first-class module fields (Session 16)
- [x] MBS Platform tests — 107 backend tests across 7 suites (Session 16). 52 frontend tests across 6 suites (Session 30, commit `8aa813c`)
- [x] Arcade SSO — BrokenChain, MindHacker, FakeArtist all verified via Chrome (Session 16)
- [x] Session 17 architecture review — 14 decisions confirmed, critical bugs documented, 20+ docs updated, 26 OneDrive conflicts cleaned (Session 17)
- [x] CWG GDPR fix — il_reflections + il_activity_feed added, source_module filtering, singletons removed (Session 18, commit `88b4e11`)
- [x] il_reflections visibility default fix — changed to "shared" + smart GET filtering (Session 17, commit `117d99b`)
- [x] CWG journal migration to il_reflections — all 17 files refactored (Session 18, commit `88b4e11`)
- [x] CWG consciousness profile + personal history sync to il_* — dual-write added (Session 18, commit `88b4e11`)
- [x] CWG migration scripts — run Session 19. Journals: 11 in il_reflections. Identity: 0 to migrate (dual-write handles future).
- [x] CWG bidirectional sync — CWG reads identity from il_* first, falls back to cwg_user_profiles (Session 19, commit `49803ba`)
- [x] FlowState il_activity_feed — writes on session completion with source_module "flowstate" (Session 19, commit `cc65a40`)
- [x] FlowState il_check_ins + il_user_wellness_profiles — auto check-in on session save, wellness sync on profile push (Session 21, commit `e8a3c79`)
- [x] Per-app test suites — MoviePicker (78), SmartCart (29), BrokenChain (33), FlowState (29), MindHacker (32), Trivia Roast (41), TaskTracker (38) all passing. CWG (66) pre-existing. 12/15 apps covered, 548+ tests total (Sessions 21-22)
- [x] DreamLens Codex review — 5 violations documented (Session 21), 4 of 5 fixed by Codex (Session 22: V1 Winston, V2 rate limiting, V4 GDPR, V5 tests). V3 express-validator still open.
- [x] Architecture evolution standards — `architecture-evolution.md` rules file, bootstrap skills updated, global CLAUDE.md updated (Session 21)
- [x] Inner Lab response helpers — all 14 routes migrated (Session 19, commit `295a962`)
- [x] Inner Lab tests — 33 tests across 2 suites (Session 19, commit `0f3f896`)
- [x] BreathArc removal confirmed — removed from products.js + all platform-instructions (Session 29). Replaced with Bonds + LifeMap.
- [x] GDPR cascade test — full account deletion verified end-to-end (Session 29). `DELETE /api/auth/account` → 200, all child apps called.
- [x] MBS frontend tests — 52 tests across 6 suites (Session 30, commit `8aa813c`). BillingPage (8), AdminPage (8), OnboardingModal (12), AuthContext (9), AuthButton (5), ProtectedRoute (5).
- [x] Account dropdown — "My Account" + "Sign Out" in nav header, Chrome-verified live (Session 29, commit `95c4ab6`)
- [x] SSO redirect improvements — auto-allow product catalog URLs + subdomain trust + unified performRedirect (Session 29, commit `ba72201`)
- [x] MBS_PLATFORM_URL verified — CWG prod correct, CWG dev fixed (Session 29). CWG Dev Chrome-verified healthy (Session 30).
- [x] Per-app validation audit — 14/14 rate limited, 12/14 validated. Gaps: Trivia Roast, Tutor, Fake Artist (Session 30).
- [x] Inner Lab features verified — Daily Briefing, Weekly Review, Consciousness Full Mode all complete (Session 30).
- [x] Architecture plans — centralized email digest, user dashboard, win-back/campaigns, family/teams (Session 30).
- [x] Admin product link fix + all 3 categories on homepage (Session 30 (continued), MBS commit `1b08acf`)
- [x] Module alignment prompts — 3 docs created, IL agent reviewed and confirmed (Session 30 (continued))
- [x] PLATFORM_URL dual-use pattern documented — frontend=magicbusstudios.com, backend=api.magicbusstudios.com (Session 30 (continued))
- [x] Global reference files updated — innerlab-module-alignment.md added, CLAUDE.md module count fixed, registration updated (Session 30 (continued))
- [x] New modules CLAUDE.md fixed — entitlement response (isPremium), redirect URL, PLATFORM_URL, JWT_PUBLIC_KEY (Session 30 (continued))
- [x] Free tier entitlements — free_tier type added, subscribe-free rewritten, category registered:true/false, Stripe downgrade (Session 31, commit `0fc98c4`)
- [x] Subscribe pages — BillingPage→SubscribePage, SubscribeInnerLabPage, /billing→/subscribe redirects (Session 31)
- [x] GDPR email confirmation — DataDeletionRequest model, confirmation email flow, no immediate deletion (Session 31)
- [x] FlowState GDPR — verified il_user_wellness_profiles NOT deleted at app-level (identity singleton correctly protected)
- [x] IL module domain convention — all new modules → `{slug}.innerlab.ai`, 5 docs updated (Session 32)
- [x] MODULE_REVIEW_CHECKLIST.md — comprehensive review checklist for module Coolify instructions (Session 32)
- [x] Module instruction reviews — Rituals approved, StarMap/InnerQuest/LifeMap/Archetypes corrections sent (Session 32)
- [ ] Module instruction reviews — 5 remaining: Bonds, AstroCompass, Arcana, DreamLens, Nexus
- [ ] Module instruction corrections — 4 awaiting v2: StarMap, InnerQuest, LifeMap, Archetypes
- [ ] Stripe bundle price IDs created in Dashboard
- [ ] BTCPay API key regenerated
- [ ] MBS Coolify redeploy needed (Session 31 changes pushed but not deployed)
- [ ] Run coming soon seed script against production DB
