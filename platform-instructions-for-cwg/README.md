# Platform Instructions for CWG (conversationswithgod.ai)

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into `CWG/` project as `platform-instructions/`

## What This Contains
Migration instructions for CWG to use the centralized MBS Platform for auth/billing and the shared `inner_lab` database for data.

## For the Agent
Read `PLATFORM_MIGRATION.md` in this folder before starting any migration work. It contains:
- Step-by-step migration process (auth, billing, data, backend, frontend)
- Complete mapping of 56 collections into 3 buckets (platform, cwg_*, il_*)
- Dual-format field normalization (personal_history, consciousness_profile)
- User memory privacy rules
- What NOT to do (don't delete old database, don't run migration until platform is ready)

**Do NOT start migration until BOTH the MBS Platform (Layer 1) and Inner Lab Middleware (Layer 2) are built and tested.**

These instructions take priority over existing project patterns when there's a conflict.
