# Platform Instructions for FlowState / YogaGhost (yoga.magicbusstudios.com)

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into `YogaGhost/` project as `platform-instructions/`

## What This Contains
Migration instructions for FlowState to use the centralized MBS Platform for auth/billing and the shared `inner_lab` database for data.

## For the Agent
Read `PLATFORM_MIGRATION.md` in this folder before starting any migration work. It contains:
- Step-by-step migration process (auth, billing, data, backend, frontend)
- Complete mapping of 7 collections into 3 buckets (platform, yoga_*, il_*)
- Embedded array handling (breathwork/meditation/flow data in users doc)
- deviceId legacy pattern sunset (90-day window)
- What NOT to do (don't delete old database, don't run migration until platform is ready)

**Do NOT start migration until BOTH the MBS Platform (Layer 1) and Inner Lab Middleware (Layer 2) are built and tested.**

These instructions take priority over existing project patterns when there's a conflict.
