# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 25, 2026

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference files. No code is built here. Code gets built in the actual project folders (`MBS/`, `Innerlab/`, `CWG/`, `YogaGhost/`).

## What Exists

| Component | Status | Where |
|-----------|--------|-------|
| MBS website (marketing + forms) | Deployed | `MBS/` → magicbusstudios.com |
| Inner Lab website (marketing) | Deployed | `Innerlab/` → innerlab.ai |
| MBS Platform (SSO + billing) | **Not started** | Will be added to `MBS/` |
| Inner Lab Middleware (il_* APIs) | **Not started** | Will be added to `Innerlab/` |
| Inner Lab Dashboard (UI) | **Not started** | Will be added to `Innerlab/` |
| CWG | Deployed, needs migration | `CWG/` → conversationswithgod.ai |
| FlowState | Deployed, needs migration | `YogaGhost/` → yoga.magicbusstudios.com |
| Arcade games (5) | Deployed, no changes yet | Individual folders |
| Studio Works tools (6) | Deployed, no changes yet | Individual folders |

## Reference Files Ready

| File | Copy to | Purpose |
|------|---------|---------|
| `copy-to-mbs/CLAUDE.md` | `MBS/` project | Build Layer 1 (SSO, billing, entitlements) |
| `copy-to-innerlab/CLAUDE.md` | `Innerlab/` project | Build Layer 2 (middleware + dashboard) |
| `copy-to-cwg/PLATFORM_MIGRATION.md` | `CWG/` project | CWG migration instructions |
| `copy-to-yogaghost/PLATFORM_MIGRATION.md` | `YogaGhost/` project | FlowState migration instructions |

## Next Steps

1. Copy `copy-to-mbs/CLAUDE.md` into `MBS/` folder → start building Layer 1
2. Copy `copy-to-innerlab/CLAUDE.md` into `Innerlab/` folder → start building Layer 2
3. Run CWG + FlowState migration scripts (after both layers are built)
4. Copy migration docs into `CWG/` and `YogaGhost/` → refactor to use platform
