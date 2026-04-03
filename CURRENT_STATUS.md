# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: April 3, 2026 (Session 18)

## Repo Purpose: MBS Ecosystem Documentation

This repo contains architecture decisions, migration plans, reference files, audits, and project status tracking for the entire MBS ecosystem. No code is built here. Code gets built in the actual project folders. Claude Code infrastructure (skills, rules, tasks) is tracked separately in `~/Desktop/Codes/Claude Setup/`.

## Phase Status

| Phase | Status | Next Action |
|-------|--------|-------------|
| Phase 1: MBS Platform | **ALL DONE — Addendum #1-15 complete** (deployed at magicbusstudios.com) | Nothing |
| Phase 2: IL Middleware + Auth | **FULLY COMPLETE** (api.innerlab.ai + innerlab.ai/auth/*) | Nothing |
| Phase 3A: CWG Migration Script | **DONE** — 20 users migrated, 14 collections copied | Nothing |
| Phase 3B: CWG Refactor | **COMPLETE** — running on `test` branch, needs promotion to `main` | Merge test → main when ready |
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
| RS256 JWT upgrade | ✅ Session 11: RS256 signing live, all 15 apps have dual-mode (RS256→HS256). Session 12: Phase 2 attempted + reverted — back to dual-mode. | Dual-mode is the safe steady state. Phase 2 needs LazyChef SSO migration first. |
| CWG entitlement enforcement | check_entitlement() wired on `test` branch (`f9c38ab`). | Verify on CWG test site |
| CWG on `test` branch | Running on test, not main — intentional. Session 18: il_reflections refactor COMPLETE (all 17 files), GDPR endpoint FIXED, identity data sync added. Migration scripts still need to run. | Run migration scripts, verify, then merge test → main |
| CWG Settings page crash | "Illegal constructor" TypeError on /settings | Pre-existing, mitigated with ErrorBoundary (root cause unknown) |
| CWG → Inner Lab data migration | Journals, consciousness profile, personal history migration CODE complete. Migration scripts written but NOT YET RUN. | Run `migrate_journals_to_il_reflections.py` + `migrate_identity_to_il.py` inside Docker |
| ~~CWG GDPR missing collections~~ | ✅ RESOLVED Session 18 — il_reflections + il_activity_feed added to deletion scope | Fixed in commit `88b4e11` |
| ~~CWG GDPR no source_module filter~~ | ✅ RESOLVED Session 18 — all il_* deletes now filter by `source_module: "cwg"`, identity singletons removed from app-level deletion | Fixed in commit `88b4e11` |
| FlowState zero il_* writes | FlowState writes no data to any il_* shared collections (no check-ins, memories, reflections) | FlowState il_* integration deferred — not blocking CWG work |
| FlowState GDPR user_id inconsistency | FlowState GDPR endpoint has mix of `userId` and `user_id` field references | Fix user_id field consistency (deferred) |
| il_reflections visibility default wrong | ✅ RESOLVED — Mongoose model default changed to "shared", route create fallback changed to "shared", GET route now defaults to visibility: "shared" for unified dashboard view (Session 17) | Fixed in Reflection.js + reflections.js |
| TaskTracker VITE_PRODUCT_DOMAIN | ✅ RESOLVED — had `https://` prefix causing double protocol in redirect URL | Fixed Session 15 — removed prefix from Coolify build arg |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website + Platform | **Deployed — Phase 1 complete** | magicbusstudios.com |
| Inner Lab website + Middleware + Auth | **Deployed — Session 18: 8 dashboard pages, 13 il_* collections (added il_birth_profiles), 13 route files, full Journal + notifications + wellness** | innerlab.ai / api.innerlab.ai |
| CWG | **Migrated** — running on `test` branch | conversationswithgod.ai |
| FlowState | **Migrated** — live on production | yoga.magicbusstudios.com |
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
- [ ] RS256 Phase 2 — attempted Session 12, **reverted**. All 15 apps back to dual-mode. Needs LazyChef SSO migration first.
- [ ] LazyChef SSO migration — frontend still uses local auth routes. Must migrate before removing `create_jwt_token`.
- [x] Inner Lab platform interface — 8 dashboard pages, 13 route files, 13 il_* collections (Session 16 + Session 18 il_birth_profiles)
- [x] il_reflections — shared journal store with first-class module fields (Session 16)
- [x] MBS Platform tests — 107 tests across 7 suites (Session 16 verified)
- [x] Arcade SSO — BrokenChain, MindHacker, FakeArtist all verified via Chrome (Session 16)
- [x] Session 17 architecture review — 14 decisions confirmed, critical bugs documented, 20+ docs updated, 26 OneDrive conflicts cleaned (Session 17)
- [x] CWG GDPR fix — il_reflections + il_activity_feed added, source_module filtering, singletons removed (Session 18, commit `88b4e11`)
- [x] il_reflections visibility default fix — changed to "shared" + smart GET filtering (Session 17, commit `117d99b`)
- [x] CWG journal migration to il_reflections — all 17 files refactored (Session 18, commit `88b4e11`)
- [x] CWG consciousness profile + personal history sync to il_* — dual-write added (Session 18, commit `88b4e11`)
- [ ] CWG migration scripts — `migrate_journals_to_il_reflections.py` + `migrate_identity_to_il.py` not yet run
- [ ] Stripe bundle price IDs created in Dashboard
- [ ] BTCPay API key regenerated
