# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 30, 2026 (Session 9 — End)

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference files. No code is built here. Code gets built in the actual project folders.

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
| Stripe bundle price IDs | IL All Access + MBS All Access checkout buttons fail | Create products/prices in Stripe Dashboard |
| GDPR deletion | Deployed and verified. All endpoints live, cascade service working, deletion UI on AccountPage. | Owner: test delete from /account |
| Subscribe gating + trial | Three-tier model deployed. ProductPickerPage at /products. 7-day free trial. Lazy downgrade. | Owner: test "Start Free Trial" |
| Admin dashboard | 5 tabs (Overview, Users, Entitlements, Flags, Activity). Real-time "Live" badge. Push notifications. | Verified live |
| 12 enhancements | AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push, real-time. All deployed. | Verified live |
| CWG entitlement enforcement | check_entitlement() wired to profile endpoint on `test` branch (`f9c38ab`). | Verify on CWG test site |
| CWG on `test` branch | Running on test, not main — intentional | Owner decision: stay on `test` indefinitely |
| CWG Settings page crash | "Illegal constructor" TypeError on /settings | Pre-existing, not migration-related |
| BTCPay API key 403 | Lightning payments fail | Regenerate API key (OWNER ACTION) |
| Stripe bundle price IDs | IL All Access + MBS All Access checkout fail | Create in Stripe Dashboard (OWNER ACTION) |
| Framer Motion opacity | ~~All 5 platform pages dark under ProtectedRoute~~ | **FIXED** in Session 9 (multiple commits) — initial={false} everywhere |
| SectionHeading on protected pages | ~~whileInView never fires under ProtectedRoute~~ | **FIXED** — replaced with inline headings (`f66205e`) |
| AnimatePresence initial mount | ~~Price toggles and tab content invisible~~ | **FIXED** — AnimatePresence initial={false} (`f9aa92d`, `6ba2e57`) |
| Modern pricing page | BillingPage redesigned with toggle + hero | **DEPLOYED** — `0e542c5` |
| IndividualPlansPage | New page at /billing/individual | **DEPLOYED** — `0e542c5` |
| Premium UI polish | All 5 platform pages match marketing quality | **DEPLOYED** — `ff2a4c9` |
| VAPID keys for push | ✅ RESOLVED — VAPID keys configured, push working | Session 8 |
| Response helpers adoption | sendSuccess/sendError created but routes still use inline { success } | Gradual migration when touching routes |

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
- [x] Phase 5 learnings cascaded to instructions (commit `0aa6711`)
- [x] All 11 standalone products verified live via Chrome browser
- [x] Phase 5 reports collected from all 11 project folders
- [x] Platform-instructions synced to all 15 project folders (Session 6)
- [x] CWG stays on `test` branch (owner decision — intentional)
- [ ] Stripe bundle price IDs created in Dashboard
- [ ] BTCPay API key regenerated
- [x] GDPR cascade to standalone products (code built Session 7, committed/pushed Session 8 — needs deploy)
- [ ] Dead code cleanup across apps
