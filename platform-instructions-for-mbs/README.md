# Platform Instructions for MBS (magicbusstudios.com)

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into `MBS/` project as `platform-instructions/`

## What This Contains
Instructions for upgrading the MBS website (magicbusstudios.com) into the centralized MBS Platform — Layer 1 of the three-layer architecture.

## For the Agent
Read `CLAUDE.md` in this folder before starting any work. It contains:
- What to add to the existing MBS website (SSO, billing, entitlements, branded login)
- New API endpoints to build
- New frontend pages to add (login, billing, account)
- Database schema for `mbs_platform`
- Environment variables needed
- Migration script requirements for CWG and FlowState
- What NOT to do (don't break existing marketing pages or form handler)

These instructions take priority over existing project patterns when there's a conflict.
