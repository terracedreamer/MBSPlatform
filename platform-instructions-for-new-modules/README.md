# Platform Instructions — New Inner Lab Module Starter Kit

**Source**: MBS Platform Architecture Repo (think tank)
**Target**: Copy this entire folder into a NEW Inner Lab module project as the starting `platform-instructions/`

## What This Contains
A comprehensive starter CLAUDE.md for any new Inner Lab module. It provides:
- Full architecture context (three-layer model, how modules connect)
- Database conventions (inner_lab DB, collection prefixing, shared il_* tables)
- User memory privacy model (opt-in cross-module sharing)
- JWT authentication pattern (validates MBS Platform tokens)
- Entitlement checking (how to verify user has access)
- Login flow (redirects to MBS Platform with Inner Lab branding)
- Project structure template
- Coding rules and conventions
- List of existing modules and their prefixes
- What NOT to do

## How to Use
1. Create your new module's project folder (e.g., `BreathArc/`)
2. Copy this entire folder into it as `platform-instructions/`
3. The CLAUDE.md in this folder becomes the foundation — add a section at the top describing YOUR module (name, slug, features, what it does)
4. The agent reads this and has full context from day one

## What You Still Need to Add (Module-Specific)
- Module name and slug
- Module description and features
- Which il_* shared data your module reads (e.g., consciousness profile, wellness data)
- What your module contributes to shared data (e.g., breathwork sessions → check-ins)
- Your module's specific collections and their schemas
- Any AI/OpenAI usage specific to your module
