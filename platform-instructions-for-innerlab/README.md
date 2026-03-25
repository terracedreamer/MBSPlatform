# Platform Instructions for Inner Lab (innerlab.ai)

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into `Innerlab/` project as `platform-instructions/`

## What This Contains
Instructions for upgrading the Inner Lab website (innerlab.ai) into the Inner Lab Middleware + Dashboard — Layer 2 of the three-layer architecture.

## For the Agent
Read `CLAUDE.md` in this folder before starting any work. It contains:
- What to add to the existing Inner Lab website (server/ backend, dashboard pages)
- Shared `il_*` collection schemas (consciousness profiles, user memories, check-ins, wellness profiles)
- API endpoints to build (consciousness, memories, check-ins, activity feed, encryption/export)
- User memory privacy model (opt-in cross-module sharing)
- New frontend pages (dashboard, consciousness profile, memories, activity feed)
- Environment variables needed
- Relationship to MBS Platform (Layer 1) and Inner Lab modules (CWG, FlowState, etc.)
- What NOT to do (don't handle auth/billing — that's the MBS Platform's job)

These instructions take priority over existing project patterns when there's a conflict.
