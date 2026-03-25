# CURRENT STATUS — MBS Platform Architecture Repo

**Last Updated**: March 25, 2026

## Repo Purpose: Architecture Think Tank (No Code)

This repo contains architecture decisions, migration plans, and reference CLAUDE.md files. No code is built here. Code gets built in the actual project folders (`MBS/`, `Innerlab/`).

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

| File | Copy to | Status |
|------|---------|--------|
| `MBS-platform-reference/CLAUDE.md` | `MBS/` project | Ready to copy |
| `InnerLab-middleware/CLAUDE.md` | `Innerlab/` project | Ready to copy |

## Next Steps

1. Copy `MBS-platform-reference/CLAUDE.md` into `MBS/` folder, start building Layer 1
2. Copy `InnerLab-middleware/CLAUDE.md` into `Innerlab/` folder, start building Layer 2
3. Write CWG + FlowState migration scripts
4. Refactor CWG + FlowState to use platform auth
