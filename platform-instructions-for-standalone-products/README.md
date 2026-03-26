# Platform Instructions for Standalone Products (Arcade + Studio Works)

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into any Arcade game or Studio Works app as `platform-instructions/`

## What This Contains
SSO migration instructions for standalone products (Arcade games and Studio Works apps) to use the centralized MBS Platform for authentication and billing.

## For the Agent
Read `PLATFORM_MIGRATION.md` in this folder before starting any migration work. It contains:
- Step-by-step migration process (add JWT middleware, remove standalone auth, remove billing, update frontend)
- JWT validation middleware code
- Login flow redirect pattern
- Entitlement check pattern
- Environment variables to add/remove
- Complete list of all 11 products that need this migration
- What NOT to do

**This is a simpler migration than CWG or FlowState** — no database migration, no shared data layer. Just auth and billing changes.

**Do NOT start until the MBS Platform (Layer 1) is built and tested.**

## How to Use
1. Copy this folder into the product's project as `platform-instructions/`
2. Fill in the product-specific values at the top of PLATFORM_MIGRATION.md (slug, domain, current auth/billing)
3. The agent reads it and has full context

These instructions take priority over existing project patterns when there's a conflict.
