# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 27, 2026 (Session 4)

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference files. No code is built here. Code gets built in the actual project folders.

## Phase Status

| Phase | Status | Next Action |
|-------|--------|-------------|
| Phase 1: MBS Platform | **ALL DONE — Addendum #1-15 complete** (deployed at magicbusstudios.com) | Nothing |
| Phase 2: IL Middleware + Auth | **FULLY COMPLETE** (api.innerlab.ai + innerlab.ai/auth/*) | Nothing — all auth pages live, ProtectedRoute fixed |
| Phase 3A: CWG Migration Script | **DONE** — 20 users migrated, 14 collections copied | Report filed in phase-reports/ |
| Phase 3B: CWG Refactor | **NOT STARTED** — no report found in CWG/ | Run Step B prompt from ORCHESTRATION_GUIDE.md |
| Phase 4: FlowState Migration | **READY TO START** — can parallel with Phase 3 | Same pattern as CWG, simpler (0 users) |
| Phase 5: Standalone Products | **READY TO START** — can parallel with Phase 3+4 | 11 independent products, all independent |

## Known Issues

| Issue | Impact | Resolution |
|-------|--------|-----------|
| BTCPay API key 403 | Lightning payments fail | Regenerate API key with full store permissions in BTCPay |
| Stripe bundle price IDs | IL All Access + MBS All Access checkout buttons fail | Create products/prices in Stripe Dashboard |
| MBS /auth/forgot-password | May have been built by Phase 1 Addendum agent (AuthForgotPasswordPage.jsx in updated report) — verify on live site | Low priority if missing — IL has its own |
| GDPR cascade incomplete | Account delete removes mbs_platform data but NOT il_* data | Phase 1 was built before Phase 2; MBS delete endpoint needs to call Innerlab to cascade — future work |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website (marketing + forms) | Deployed | `MBS/` → magicbusstudios.com |
| MBS Platform (SSO + billing) | **Deployed — Phase 1 complete** | `MBS/` → magicbusstudios.com |
| Inner Lab website (marketing) | Deployed | `Innerlab/` → innerlab.ai |
| Inner Lab Middleware (il_* APIs) | **Deployed — Phase 2 complete** | `Innerlab/` → api.innerlab.ai |
| Inner Lab Auth Pages (4 pages) | **Deployed** (login/signup/forgot-pw/reset-pw) | `Innerlab/` → innerlab.ai/auth/* |
| Inner Lab Dashboard (4 pages) | **Deployed and unblocked** (full login flow working) | `Innerlab/` → innerlab.ai/dashboard |
| CWG | Deployed, **data migrated** (Step A done), needs refactor (Step B) | `CWG/` → conversationswithgod.ai |
| FlowState | Deployed, needs migration | `YogaGhost/` → yoga.magicbusstudios.com |
| Arcade games (5) | Deployed, needs SSO migration | Individual folders |
| Studio Works tools (6) | Deployed, needs SSO migration | Individual folders |

## Sync Status (verified Session 3)

| Target | Source → Target | Status |
|--------|----------------|--------|
| MBS/platform-instructions/ | platform-instructions-for-mbs/ | **IN SYNC** (addendum #14-15 added) |
| Innerlab/platform-instructions/ | platform-instructions-for-innerlab/ | **IN SYNC** (login/signup spec added) |
| CWG/platform-instructions/ | platform-instructions-for-cwg/ | **IN SYNC** (user copy-across + auth_methods) |
| YogaGhost/platform-instructions/ | platform-instructions-for-yogaghost/ | **IN SYNC** (login redirect updated) |

## Pre-Build Checklist

- [x] JWT_SECRET generated
- [x] Platform-instructions synced to all 4 project folders
- [x] Project CLAUDE.md files updated to point agents to platform-instructions/
- [x] Completion report requirements baked into all platform-instructions docs
- [x] Marketing briefs updated and synced
- [x] Orchestration guide created with copy-paste prompts
- [x] Phase 1 built and deployed
- [x] Phase 1 report reviewed, learnings cascaded to all downstream instructions
- [x] Phase 2 built and deployed
- [x] Phase 2 report reviewed, learnings noted
- [x] Email/password auth + 2FA added to MBS Platform (addendum #14-15)
- [x] Inner Lab login/signup pages built (Phase 2 Addendum — all 4 pages live)
- [x] Phase 2 Addendum report reviewed and filed
- [x] Phase 3A: CWG data migration script run (20 users, 14+1 collections)
- [ ] Phase 3B: CWG refactor (remove standalone auth/billing, switch to JWT + inner_lab DB)
- [ ] Phase 4 started (FlowState Migration)
- [ ] Phase 5 started (11 standalone products)
