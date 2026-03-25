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

| Folder in This Repo | Copy to | Purpose |
|------|---------|---------|
| `platform-instructions-for-mbs/` | `MBS/platform-instructions/` | Build Layer 1 (SSO, billing, entitlements) |
| `platform-instructions-for-innerlab/` | `Innerlab/platform-instructions/` | Build Layer 2 (middleware + dashboard) |
| `platform-instructions-for-cwg/` | `CWG/platform-instructions/` | CWG migration instructions |
| `platform-instructions-for-yogaghost/` | `YogaGhost/platform-instructions/` | FlowState migration instructions |
| `platform-instructions-for-new-modules/` | New module project as `platform-instructions/` | Starter kit for future modules |

## Next Steps

1. Copy `platform-instructions-for-mbs/` into `MBS/` as `platform-instructions/` → start building Layer 1
2. Copy `platform-instructions-for-innerlab/` into `Innerlab/` as `platform-instructions/` → start building Layer 2
3. Run CWG + FlowState migration scripts (after both layers are built)
4. Copy migration docs into `CWG/` and `YogaGhost/` as `platform-instructions/` → refactor to use platform
