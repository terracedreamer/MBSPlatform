# CHANGELOG — MBS Platform

## March 25, 2026 — Session 1 (final): Merged Architecture + Think Tank Model

### Major decision: Merge platforms into existing website projects
- MBS Platform code gets built inside `MBS/` folder (magicbusstudios.com) — NOT a separate project
- Inner Lab Middleware + Dashboard gets built inside `Innerlab/` folder (innerlab.ai) — NOT a separate project
- `ILPlatform/` folder no longer needed
- This repo (`MBSPlatform/`) becomes the **architecture think tank** — no code, just docs + reference CLAUDE.md files
- Created `copy-to-mbs/CLAUDE.md` — copy into `MBS/` to build Layer 1
- Updated `copy-to-innerlab/CLAUDE.md` — copy into `Innerlab/` to build Layer 2

### Result: 4 containers instead of 8
- magicbusstudios.com: frontend (marketing + login + billing) + backend (forms + SSO + entitlements)
- innerlab.ai: frontend (marketing + dashboard) + backend (il_* APIs)
- Same pattern as every other MBS product — no new containers needed

### Previous revision (also this date)
- Inner Lab Middleware backend must be built NOW, not later
- Inner Lab frontend/dashboard also built NOW (not deferred)
- Reason: migrations need il_* shared tables, dashboard has value even with just CWG data

### Files updated
- CLAUDE.md: revised build order, three-layer architecture clarification
- copy-to-innerlab/CLAUDE.md: changed from "build later" to "build now"
- SESSION_HANDOFF.md: revised build order and decision #4
- CURRENT_STATUS.md: updated next steps
- FUTURE_WORK_TODO.md: moved middleware to Priority 2B (build now), frontend stays later
- CHANGELOG.md: this entry

---

## March 25, 2026 — Session 1 (continued): Architecture Finalized

### What happened
- Inspected CWG database schema (56 collections documented in DATABASE_SCHEMA.md)
- Inspected FlowState database schema (7 collections documented in DATA_SCHEMA.md)
- Both product agents created MBS_DATABASE_MIGRATION_PLAN.md with bucket categorizations
- Discovered CWG has extensive sovereignty features: Nostr, LNURL-Auth, BTCPay Lightning, client encryption, blockchain anchors, GDPR consent/data requests
- Discovered FlowState has MongoDB backend (7 collections), not frontend-only as initially assumed
- Reviewed live sites: magicbusstudios.com, innerlab.ai, conversationswithgod.ai
- Confirmed branded login model (Inner Lab branding vs MBS branding based on origin)

### All architecture decisions finalized (17 total)
1. Fresh `inner_lab` database — old DBs stay as backups
2. User identity ONLY in `mbs_platform`
3. Shared IL data starts as module-prefixed, promote to `il_*` when needed
4. Inner Lab Core container — build later when 2-3 modules exist
5. Frontend: B+C hybrid (standalone modules + future unified dashboard)
6. Branded login (IL modules → IL branding, Arcade/SW → MBS branding)
7. Arcade/Studio Works — SSO only, no shared data
8. Admin — simple `is_admin` flag
9. Nostr/LNURL auth — MBS Platform level (all products)
10. Payments — Stripe + BTCPay both at MBS Platform
11. Friends/Invites — MBS Platform level with product context
12. Activity feed — Inner Lab level (built when IL frontend exists)
13. Push subscriptions — MBS Platform level
14. Feature flags — MBS Platform level
15. User memories (AI) — Inner Lab level with user opt-in (Option C)
16. Encryption & data export — Inner Lab level
17. Modules as separate containers sharing `inner_lab` database

### Files created/updated
- Updated `CLAUDE.md` — added sovereignty features, branded login, three-layer architecture, migration details
- Updated `SESSION_HANDOFF.md` — complete context with all 17 decisions
- Updated `CHANGELOG.md`, `CURRENT_STATUS.md`, `FUTURE_WORK_TODO.md`
- Created `copy-to-innerlab/CLAUDE.md` — full reference for building Layer 2

### No code written
- Project remains spec-only. Architecture is complete. Ready for Phase 1 implementation.

---

## March 24, 2026 — Session 1: Architecture Planning (Started)

### What happened
- Full architecture discussion across MBS Platform + Inner Lab ecosystem
- Read and analyzed all reference documents (Technical Architecture, Product Briefs, ChatGPT architecture docs, Inner Lab Blueprint)
- Reviewed live innerlab.ai website (marketing site + modules page)

### Initial decisions made
- Three-layer architecture established
- B+C hybrid frontend model agreed
- Module isolation (separate containers, shared DB) confirmed
