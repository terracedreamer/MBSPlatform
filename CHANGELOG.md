# CHANGELOG — MBS Platform

## March 26, 2026 — Session 2: Full Audit, Marketing Briefs, Orchestration Guide, Build Prep

### Summary
6 audit passes, 50+ issues found and fixed. Architecture is implementation-ready. Orchestration workflow established with auto-generated phase reports.

### Marketing Brief Enhancements
- Absorbed Arcade detail into MBS master doc (URLs, taglines, pricing, Time Banks, target audience)
- Expanded Studio Works from one-liners to full feature descriptions
- Added Inner Lab Dashboard section to IL brief (module launcher, activity feed, cross-module intelligence, memory privacy)
- Enhanced FlowState description (breathwork, streaks, achievements, programs)
- Added design language to IL brief (exact colors, typography from live innerlab.ai)
- Added website state documentation to MBS brief (page listing, known issues)
- Synced all briefs to Desktop/Marketing/Overview/

### Architecture Fixes (across 6 audit passes)
- Fixed stale folder names in CLAUDE.md, CURRENT_STATUS.md (copy-to-* → platform-instructions-for-*)
- Fixed wrong MBS_PLATFORM_URL in IL middleware (platform.magicbusstudios.com → magicbusstudios.com)
- Fixed contradictory build-later/build-now in IL middleware
- Removed redundant il_shared_memories and il_daily_check_ins collections
- Synced memory document schema across IL middleware and new-modules kit
- Added friends API endpoints to main CLAUDE.md
- Fixed entitlement check URLs (added https://)
- Clarified IL middleware CORS (dashboard frontend only, modules use shared DB)
- Assigned prefixes: AstroCompass → astrocart_*, Archetypes → archetype_*
- Standardized status to "Coming Soon" across module tables
- Added il_user_wellness_profiles and il_blockchain_anchors to collection lists
- Fixed CWG collection count (39 → 42)
- Added user dedup/merge logic (upsert on email) to both migration docs
- Fixed avatar/picture, google_id/googleId, stripeCustomerId field renames in migrations
- Added all 12 missing FlowState User model fields with defaults

### New Architectural Specs Added
- JWT payload specification (userId, email, name, avatar, isAdmin, iat, exp)
- il_* schema contracts for 5 collections (check-ins, consciousness, histories, wellness, activity)
- MongoDB JSON Schema validation recommendations
- Open redirect protection (ALLOWED_REDIRECT_DOMAINS from CORS_ORIGINS)
- Token-in-URL handling (extract → store → replaceState)
- Deep link preservation through login redirect
- Free tier entitlement logic (product catalog freeTier flag, 5 reason values)
- BTCPay → Entitlement flow (webhook → create entitlement with expiry)
- Entitlement priority order (mbs_all_access > category > product > free)
- Entitlement cache spec (5min TTL, ?refresh=true invalidation)
- Entitlement category validation (innerlab|arcade|studioworks, no underscores)
- Admin endpoints: verify is_admin from DB not JWT
- JWT security model (shared secret risk, RS256 upgrade path)
- Token refresh strategy (Phase 1: simple expiry, Phase 2: refresh tokens)
- Entitlement API resilience (cache, graceful degradation)
- GDPR deletion cascade (cross-database: mbs_platform + inner_lab + all module prefixes)
- CORS complete domain enumeration (18 domains)
- Deployment checklist (env vars first, health check, defensive route loading)
- Removed stripe_customer_id from Entitlement model (lives on User only)
- stripe_customer_id placement clarification
- Dual write path for il_* clarified (modules via DB, dashboard via HTTP)
- userId → user_id mapping documented (JWT camelCase, DB snake_case)
- CWG cutover sequence (simplified: run script, deploy, users re-login)
- source_module field name fixed (was inconsistently "source" in one place)
- FlowState friends/invites collection naming fixed (no mbs_ prefix needed)

### New Files Created
- `platform-instructions-for-standalone-products/` — SSO migration for 11 Arcade + Studio Works apps
- `ORCHESTRATION_GUIDE.md` — Step-by-step with exact copy-paste prompts for each Claude Code session

### Files Updated
- CLAUDE.md (main), CURRENT_STATUS.md, FUTURE_WORK_TODO.md, SESSION_HANDOFF.md, CHANGELOG.md
- platform-instructions-for-mbs/CLAUDE.md
- platform-instructions-for-innerlab/CLAUDE.md
- platform-instructions-for-cwg/PLATFORM_MIGRATION.md
- platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md
- platform-instructions-for-new-modules/CLAUDE.md
- marketing-docs/MagicBusStudios_Brand_And_Company.md
- marketing-docs/InnerLab_Product_Brief.md
- marketing-docs/TheArcade_Marketing_Brief.md

### Completion Report System (added late Session 2)
- Added `Completion Report (REQUIRED)` section to all 5 platform-instructions docs
- Phase 1 → PHASE_1_REPORT.md, Phase 2 → PHASE_2_REPORT.md, etc.
- Each report tailored to phase-specific concerns (migration results, JWT integration, etc.)
- Reports feed back to orchestrator session for review before starting next phase

### Marketing Brief Final Sync
- Updated all 4 briefs to March 26 date
- Arcade brief game descriptions synced with live website copy (Broken Chain, MindHacker, Whispering House, Fake Artist)
- Trivia Roast URL typo note added to Arcade brief

### Build Prep
- JWT_SECRET generated
- Orchestration workflow documented (build → report → review → next phase)
- Pre-build checklist added to CURRENT_STATUS.md

---

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
