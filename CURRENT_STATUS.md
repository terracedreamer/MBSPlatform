# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 26, 2026 (End of Session 2)

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference files. No code is built here. Code gets built in the actual project folders.

## Phase Status

| Phase | Status | Next Action |
|-------|--------|-------------|
| Phase 1: MBS Platform | **Core + Addendum DONE** (deployed at magicbusstudios.com) | Fix BTCPay API key, create Stripe bundle price IDs, regenerate package-lock.json |
| Phase 2: IL Middleware | **Ready to build** (instructions synced) | Open Claude Code in `Innerlab/`, paste prompt from orchestration guide |
| Phase 3: CWG Migration | Ready (instructions synced) | After Phase 1+2 are deployed |
| Phase 4: FlowState Migration | Ready (instructions synced) | After Phase 1+2 are deployed (can parallel with Phase 3) |
| Phase 5: Standalone Products | Ready (instructions synced) | After Phase 1 is deployed (can all parallel) |

## Known Issues

| Issue | Impact | Resolution |
|-------|--------|-----------|
| BTCPay API key 403 | Lightning payments fail | Regenerate API key with full store permissions in BTCPay |
| Stripe bundle price IDs | IL All Access + MBS All Access checkout buttons fail | Create products/prices in Stripe Dashboard |
| MBS server package-lock.json | Deploy will fail | Run `npm install` in MBS/server/ and push |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website (marketing + forms) | Deployed | `MBS/` → magicbusstudios.com |
| MBS Platform (SSO + billing) | **Deployed — Phase 1 complete** | `MBS/` → magicbusstudios.com |
| Inner Lab website (marketing) | Deployed | `Innerlab/` → innerlab.ai |
| Inner Lab Middleware (il_* APIs) | **Not started** | Will be added to `Innerlab/` |
| CWG | Deployed, needs migration | `CWG/` → conversationswithgod.ai |
| FlowState | Deployed, needs migration | `YogaGhost/` → yoga.magicbusstudios.com |
| Arcade games (5) | Deployed, needs SSO migration | Individual folders |
| Studio Works tools (6) | Deployed, needs SSO migration | Individual folders |

## Sync Status (verified end of Session 2)

| Target | Source → Target | Status |
|--------|----------------|--------|
| MBS/platform-instructions/ | platform-instructions-for-mbs/ | **IN SYNC** |
| Innerlab/platform-instructions/ | platform-instructions-for-innerlab/ | **IN SYNC** |
| CWG/platform-instructions/ | platform-instructions-for-cwg/ | **IN SYNC** |
| YogaGhost/platform-instructions/ | platform-instructions-for-yogaghost/ | **IN SYNC** |
| Desktop/Marketing/Overview/ | marketing-docs/ | **IN SYNC** |

## Pre-Build Checklist

- [x] JWT_SECRET generated
- [x] Platform-instructions synced to all 4 project folders
- [x] Project CLAUDE.md files updated to point agents to platform-instructions/
- [x] Completion report requirements baked into all platform-instructions docs
- [x] Marketing briefs updated and synced
- [x] Orchestration guide created with copy-paste prompts
- [x] Phase 1 built and deployed
- [x] Phase 1 report reviewed, learnings cascaded to all downstream instructions
- [ ] Phase 2 started (IL Middleware)
- [ ] Phase 2 report reviewed
- [ ] Phase 3 started (CWG Migration)
- [ ] Phase 4 started (FlowState Migration)
- [ ] Phase 5 started (11 standalone products)
