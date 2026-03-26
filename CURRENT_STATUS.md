# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 26, 2026

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference files. No code is built here. Code gets built in the actual project folders.

## Architecture Readiness

| Component | Status | Ready to Build? |
|-----------|--------|-----------------|
| MBS Platform instructions | 6 audit passes complete, all issues fixed | **Yes — Grade A** |
| Inner Lab Middleware instructions | Schema contracts defined, write paths clarified | **Yes — Grade A** |
| CWG Migration instructions | Cutover plan, field renames, dedup logic done | **Yes — Grade A-** |
| FlowState Migration instructions | All fields mapped with defaults | **Yes — Grade A** |
| Standalone Products instructions | SSO migration guide for all 11 apps | **Yes — Grade A** |
| New Module Starter Kit | Full template with all context | **Yes — Grade A** |
| Orchestration Guide | Step-by-step with copy-paste prompts | **Ready** |
| Marketing briefs | Enhanced with website data, design language | **Up to date** |

## What Exists (Live Products)

| Component | Status | Where |
|-----------|--------|-------|
| MBS website (marketing + forms) | Deployed | `MBS/` → magicbusstudios.com |
| Inner Lab website (marketing) | Deployed | `Innerlab/` → innerlab.ai |
| MBS Platform (SSO + billing) | **Ready to build (Phase 1)** | Will be added to `MBS/` |
| Inner Lab Middleware (il_* APIs) | **Not started** | Will be added to `Innerlab/` |
| Inner Lab Dashboard (UI) | **Not started** | Will be added to `Innerlab/` |
| CWG | Deployed, needs migration | `CWG/` → conversationswithgod.ai |
| FlowState | Deployed, needs migration | `YogaGhost/` → yoga.magicbusstudios.com |
| Arcade games (5) | Deployed, needs SSO migration | Individual folders |
| Studio Works tools (6) | Deployed, needs SSO migration | Individual folders |

## Reference Files (All Copied to Target Projects)

| Folder in This Repo | Copy to | Purpose | Synced? |
|------|---------|---------|---------|
| `platform-instructions-for-mbs/` | `MBS/platform-instructions/` | Build Layer 1 | Yes (verified via hash) |
| `platform-instructions-for-innerlab/` | `Innerlab/platform-instructions/` | Build Layer 2 | Yes (verified via hash) |
| `platform-instructions-for-cwg/` | `CWG/platform-instructions/` | CWG migration | Yes (verified via hash) |
| `platform-instructions-for-yogaghost/` | `YogaGhost/platform-instructions/` | FlowState migration | Yes (verified via hash) |
| `platform-instructions-for-standalone-products/` | Each Arcade/SW project | SSO migration | Not yet copied to individual projects |
| `platform-instructions-for-new-modules/` | New module projects | Starter kit | Ready when needed |

## Build Order

1. **Phase 1: MBS Platform** → `MBS/` (must be first — everything depends on it)
2. **Phase 2: IL Middleware + Dashboard** → `Innerlab/` (must be before migrations)
3. **Phase 3: CWG Migration** → migration script from `MBS/`, then refactor `CWG/`
4. **Phase 4: FlowState Migration** → same pattern (can parallel with Phase 3)
5. **Phase 5: 11 Standalone Products** → can ALL parallel after Phase 1
6. **Phase 6+: New IL modules** → use starter kit

See `ORCHESTRATION_GUIDE.md` for exact prompts and steps.

## Orchestration Workflow

Each phase agent generates a `PHASE_X_REPORT.md` when done. Bring the report back to this orchestrator session before starting the next phase. The orchestrator reviews and updates downstream instructions if needed.

## Pre-Build Checklist

- [x] JWT_SECRET generated
- [x] Platform-instructions synced to all 4 project folders (hash-verified)
- [x] Project CLAUDE.md files updated to point agents to platform-instructions/
- [x] Completion report requirements baked into all platform-instructions docs
- [x] Marketing briefs updated and synced to Desktop/Marketing/Overview/
- [x] Orchestration guide created with copy-paste prompts
- [ ] Phase 1 started (MBS Platform)
- [ ] Phase 1 report reviewed by orchestrator
- [ ] Phase 2 started (IL Middleware)
- [ ] Phase 2 report reviewed
- [ ] Phase 3 started (CWG Migration)
- [ ] Phase 4 started (FlowState Migration)
- [ ] Phase 5 started (11 standalone products)
