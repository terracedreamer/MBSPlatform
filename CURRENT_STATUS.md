# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 31, 2026 (Session 12 — consolidation pass)

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
| TaskTracker | studioworks | tasktracker.magicbusstudios.com | Authenticated (parent dashboard) |
| AI Tutor | studioworks | tutor.magicbusstudios.com | Authenticated (dashboard, 50 XP) |

## Known Issues

| Issue | Impact | Resolution |
|-------|--------|-----------|
| BTCPay API key 403 | Lightning payments fail | Regenerate API key with full store permissions in BTCPay |
| Stripe bundle price IDs | Checkout buttons fail for all 6 products | Create products/prices in Stripe Dashboard (see FUTURE_WORK_TODO.md) |
| RS256 JWT upgrade | ✅ Complete. Phase 2 done — HS256 fallback removed from all 15 apps. RS256-only. | **DONE** (Session 11 deploy + Session 12 cleanup) |
| CWG entitlement enforcement | check_entitlement() wired on `test` branch (`f9c38ab`). | Verify on CWG test site |
| CWG on `test` branch | Running on test, not main — intentional | Owner decision: merge when ready |
| CWG Settings page crash | "Illegal constructor" TypeError on /settings | Pre-existing, not migration-related |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website + Platform | **Deployed — Phase 1 complete** | magicbusstudios.com |
| Inner Lab website + Middleware + Auth | **Deployed — Phase 2 complete** | innerlab.ai / api.innerlab.ai |
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
- [x] RS256 Phase 2 cleanup — HS256 fallback removed from all 15 apps (Session 12)
- [x] LazyChef self-issued auth removed — fully relies on MBS Platform SSO (Session 12)
- [ ] Stripe bundle price IDs created in Dashboard
- [ ] BTCPay API key regenerated
