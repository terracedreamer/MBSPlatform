# Inner Lab Module Review Guide

> **Purpose**: Comprehensive reference for reviewing all Inner Lab modules built by Codex and providing feedback via the Document 27/28 system.
> **Location**: MBSPlatform repo (accessible from any session)
> **Last updated**: 2026-04-05 (Session 23)

---

## How to Use This File

Start a Claude Code session and say:

```
Read MBSPlatform/INNERLAB_MODULE_REVIEW_GUIDE.md, then review the modules
listed and provide feedback in Document 28.
```

Claude will:
1. Read this guide for full context
2. Navigate to each module repo and inspect code, architecture, and il_* integration
3. Compare against live runtime in `Innerlab/`, `MBS/`, and `MBSPlatform/`
4. Append findings to Document 28 (`Innerlab Incubator/docs/28-claude-response-log.md`)

---

## The Document 27/28 Feedback Loop

This is a structured review system between **Codex** (OpenAI, builds modules) and **Claude Code** (Anthropic, reviews architecture).

### Document 27 — Questions (Codex writes, Claude reads)

| File | Scope |
|------|-------|
| `Innerlab Incubator/docs/27-claude-cross-reference-pack.md` | General architecture questions across all modules |
| `DreamLens/docs/27-claude-dreamlens-runtime-review-pack.md` | DreamLens-specific review questions |
| `Codex Setup/docs/01-shared-codex-skills-claude-cross-reference-pack.md` | Codex skills review |

### Document 28 — Answers (Claude writes, Codex reads)

| File | Scope |
|------|-------|
| `Innerlab Incubator/docs/28-claude-response-log.md` | **Primary answer log** — all module reviews go here |
| `DreamLens/docs/28-claude-dreamlens-response-log.md` | DreamLens-specific (currently empty — answers folded into main) |
| `Codex Setup/docs/02-shared-codex-skills-claude-response-log.md` | Codex skills answers |

### Review Process

1. **Codex** adds questions to Doc 27, tagged with IDs (e.g., `LAYER-01`, `RITUAL-AUTH-01`)
2. **User** starts a Claude Code session and points to this guide
3. **Claude** reads Doc 27, inspects the relevant module repo and live Inner Lab code
4. **Claude** appends answers to Doc 28 in a new dated round (never overwrites prior rounds)
5. **Codex** reads Doc 28 and reconciles corrections into its planning docs

### Answer Format

```md
## Round YYYY-MM-DD - [Module/Area] Review

Reviewed by: Claude Opus 4.6

Scope: [what was reviewed]

Responses:

- ID: `QUESTION-ID`
  Status: `confirmed` | `corrected` | `safe enhancement` | `risky / not recommended` | `needs live-code check`
  Answer: ...
  Why: ...
  Sources checked: ...
  Recommended incubator update: ...
```

### What to Review Per Module

When reviewing a Codex-built module, check:

1. **Standards compliance** — Winston logger (not pino/console.log), response helpers, express-validator, rate limiting, GDPR endpoint, CORS order
2. **il_* integration** — correct collections, `source_module` field, `user_id` (snake_case), sharing toggle support
3. **Auth compatibility** — MBS Platform JWT verification, `req.user` shape `{ userId, email, name, avatar, isAdmin }`
4. **Database** — must use `DB_NAME=inner_lab`, module-owned collections use correct prefix
5. **GDPR** — `DELETE /api/user-data` filters shared collections by `source_module`, protects identity singletons
6. **Architecture patterns** — next-gen builds should use `createApp()` factory, asyncHandler, TypeScript preferred
7. **Deployment readiness** — Dockerfiles, env vars, Coolify compatibility

---

## The Inner Lab Ecosystem

### Three-Layer Architecture

```
Layer 1: MBS Platform (magicbusstudios.com)
         SSO, billing, entitlements, user accounts

Layer 2: Inner Lab Middleware (innerlab.ai / api.innerlab.ai)
         Shared il_* collections, dashboard, consciousness profile, journal,
         check-ins, activity feed, memories, wellness, birth profile, sharing

Layer 3: Individual Modules
         Own backend + DB collections, JWT verification from Layer 1,
         Reads/writes to il_* shared collections in Layer 2
```

### Shared Database: `inner_lab`

All Inner Lab modules use `DB_NAME=inner_lab`. Each module owns its prefixed collections and writes to shared `il_*` collections.

### 14 Shared il_* Collections (owned by Layer 2)

| Collection | Type | Description |
|------------|------|-------------|
| `il_consciousness_profiles` | Identity (singleton) | Archetype, dimensions, assessment data |
| `il_personal_histories` | Identity (singleton) | 8-section life questionnaire |
| `il_user_wellness_profiles` | Identity (singleton) | Health conditions, injuries, goals |
| `il_birth_profiles` | Identity (singleton) | Birth date/time/place for astrology |
| `il_check_ins` | Activity (per-module toggle) | Mood/energy/stress snapshots |
| `il_user_memories` | Activity (per-module toggle) | AI-extracted insights, default private |
| `il_activity_feed` | Activity (per-module toggle) | Cross-module event log |
| `il_reflections` | Activity (per-module toggle) | Shared journal (the ONLY journal store) |
| `il_consciousness_snapshots` | System | Historical assessment snapshots |
| `il_sharing_preferences` | System | Per-module sharing toggles |
| `il_analytics_events` | System | Usage analytics |
| `il_notifications` | System | User notifications |
| `il_blockchain_anchors` | System | Future integrity proofs |
| `il_sync_backups` | System | Future sync/backup data |

---

## Module Catalog

### Live Modules (Deployed and Production-Ready)

#### 1. Conversations With God (CWG)

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/CWG/` |
| **URL** | `conversationswithgod.ai` |
| **Stack** | FastAPI (Python) + React (Vite) |
| **Collection prefix** | `cwg_*` |
| **il_* writes** | `il_reflections`, `il_check_ins`, `il_user_memories`, `il_activity_feed`, `il_consciousness_profiles`, `il_personal_histories` |
| **Status** | Live on production. Test branch has Phase 3B migration (MBS SSO + inner_lab DB). Production still on standalone auth with `conversations_with_god` DB. |
| **Branches** | `test` (migrated) / `development` (production) |
| **Key feature** | 21 AI-powered spiritual guides for guided dialogue |

#### 2. FlowState (YogaGhost)

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/YogaGhost/` |
| **URL** | `yoga.magicbusstudios.com` |
| **Stack** | Express 5 + React 19 (TypeScript), CSS Modules, Zustand, PWA |
| **Collection prefix** | `yoga_*` |
| **il_* writes** | `il_activity_feed`, `il_check_ins`, `il_user_wellness_profiles` (all `source_module: "flowstate"`) |
| **Status** | Live on production. Dev merged to main (Session 22). 29 tests. |
| **Branches** | `main` |
| **Key feature** | Three pillars: Yoga (pose detection via MediaPipe), Breathwork, Meditation |

### Codex-Built Modules (Feature-Complete, Not Yet Deployed)

#### 3. DreamLens

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/DreamLens/` |
| **Incubator** | `Innerlab Incubator/modules/dreamlens/` (24 planning docs) |
| **Stack** | Express + TypeScript, React + Vite + TypeScript, npm workspaces, native MongoDB driver |
| **Collection prefix** | `dream_entries`, `dream_symbols`, `dream_saved_patterns`, `dream_preferences` |
| **il_* writes** | `il_activity_feed`, `il_reflections`, `il_user_memories` (`source_module: "dreamlens"`) |
| **Status** | Feature-complete. Locally verified. Dockerfiles ready. NOT deployed. |
| **Architecture** | Next-gen: `createApp()` factory, asyncHandler, Zod env validation |
| **Known issues from review** | V3 (express-validator vs Zod) pending decision. 4/5 violations fixed by Codex. See `MBSPlatform/CODEX_FEEDBACK_DREAMLENS.md`. |
| **Key feature** | Private dream capture, symbol tracking, pattern recognition, optional AI interpretation |

#### 4. Arcana

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/Arcana/` |
| **Incubator** | `Innerlab Incubator/modules/arcana/` (24 planning docs) |
| **Stack** | Express + TypeScript, React + Vite + TypeScript, npm workspaces |
| **Collection prefix** | `tarot_spreads`, `tarot_readings`, `tarot_saved_cards`, `tarot_preferences` |
| **il_* writes** | `il_activity_feed` (spread_completed, card_saved). `il_reflections` intentionally deferred. |
| **Status** | Scaffolded. Local validation passes. NOT deployed. Needs product registration. |
| **Key feature** | Reflective tarot practice — daily draws, spread catalog, symbolic exploration |

#### 5. Rituals

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/Rituals/` |
| **Incubator** | `Innerlab Incubator/modules/rituals/` (24 planning docs) |
| **Stack** | Express + TypeScript (`server/`), React + Vite + TypeScript (`client/`), npm workspaces |
| **Collection prefix** | `ritual_templates`, `ritual_sessions`, `ritual_favorites`, `ritual_preferences` |
| **il_* writes** | `il_activity_feed` (ritual_completed, `source_module: "rituals"`). `il_reflections` deferred. |
| **Status** | Implemented, locally verified. Working tree dirty (interrupted AI companion work). NOT deployed. |
| **Key feature** | Short guided rituals for emotional states — "When life feels a certain way, start here" |

#### 6. Bonds

| Key | Value |
|-----|-------|
| **Repo** | `Desktop/Codes/Bonds/` |
| **Incubator** | `Innerlab Incubator/modules/bonds/` (24 planning docs) |
| **Stack** | Express + TypeScript, React + Vite |
| **Collection prefix** | `bond_*` |
| **il_* writes** | `il_activity_feed` (compact activity events) |
| **Status** | Repo created, first vertical slice in progress. Blocked on product catalog registration. |
| **Key feature** | Private relationship reflection, repair, and conversation preparation |

### Incubator-Only Modules (Planning Docs, No Real Repo Yet)

| Module | Incubator Folder | Planning Docs | Key Concept | Readiness |
|--------|-----------------|---------------|-------------|-----------|
| **Archetypes** | `modules/archetypes/` | 24 docs | Inner voice exploration (Warrior, Sage, Healer) | Later-wave |
| **StarMap** | `modules/starmap/` | 24 docs | Astrological chart cycles and patterns | Later-wave |
| **AstroCompass** | `modules/astrocompass/` | 24 docs | Location-based energy and astrocartography | Later-wave |
| **InnerQuest** | `modules/innerquest/` | 24 docs | Multi-day structured growth journeys | Later-wave (strategy prereqs) |
| **LifeMap** | `modules/lifemap/` | 24 docs | Life story and narrative mapping | Later-wave (writing-heavy) |
| **Nexus** | `modules/nexus/` | 24 docs | Marketplace for spiritual practitioners | Later-wave (not standard module) |
| **BreathArc** | `modules/breatharc/` | 12 docs (partial) | Guided breathing | **FROZEN** — FlowState owns breathwork |

---

## Inner Lab Middleware (Layer 2) — Dashboard Features

**Repo**: `Desktop/Codes/Innerlab/`
**URL**: `innerlab.ai` / `api.innerlab.ai`

### Dashboard Pages (15 total as of Session 23)

| Route | Page | Status |
|-------|------|--------|
| `/dashboard` | Main dashboard — check-in, modules, activity, suggestions | Live |
| `/daily` | Daily Briefing — time-aware intention, practice, check-in | **New (Session 23)** |
| `/weekly` | Weekly Consciousness Review — streaks, trends, dimensions | **New (Session 23)** |
| `/weekly-review` | General weekly recap — stats, modules, reflections | Live |
| `/journal` | Shared reflection journal — create, filter, bookmark, tag | Live |
| `/check-ins` | Check-in history with averages and trends | Live |
| `/consciousness` | Consciousness assessment (Quick 8q / Full 16q) + dimension evolution graph | Live |
| `/wellness` | Wellness profile — conditions, injuries, goals | Live |
| `/memories` | User memories with sharing controls | Live |
| `/activity` | Full cross-module activity feed | Live |
| `/birth-profile` | Birth date/time/place for astrology modules | Live |
| `/sharing` | Per-module sharing toggle manager | Live |
| `/my-story` | Personal history guided questionnaire | Live |

### Backend Routes (17 total as of Session 23)

| Mount | Route File | Purpose |
|-------|-----------|---------|
| `/api/check-in` | `checkins.js` | Check-in CRUD |
| `/api/consciousness` | `consciousness.js` | Profile + snapshots |
| `/api/personal-history` | `personalHistory.js` | Personal history upsert |
| `/api/memories` | `memories.js` | Memory CRUD + sharing |
| `/api/activity` | `activity.js` | Activity feed logging |
| `/api/reflections` | `reflections.js` | Journal CRUD |
| `/api/notifications` | `notifications.js` | Notification management |
| `/api/wellness` | `wellness.js` | Wellness profile |
| `/api/birth-profile` | `birthProfile.js` | Birth profile CRUD |
| `/api/sharing` | `sharing.js` | Sharing preferences |
| `/api/daily-briefing` | `dailyBriefing.js` | Daily briefing aggregation |
| `/api/suggestions` | `suggestions.js` | Cross-module suggestions |
| `/api/weekly-review` | `weeklyReview.js` | Weekly consciousness review |
| `/api/export` | `export.js` | Data export (JSON) |
| `/api/encryption` | `encryption.js` | Encryption keys (future) |
| `/api/user-data` | `userData.js` | GDPR deletion |
| `/api/contact,subscribe,waitlist` | `forms.js` | Public form submissions |

---

## Inner Lab Incubator Reference

**Location**: `Desktop/Codes/Innerlab Incubator/`

This is the Codex planning repo — NOT a runtime repo. It contains architecture documents and per-module planning packs.

### Key Architecture Documents

| Doc # | File | Purpose |
|-------|------|---------|
| 00 | `source-of-truth-hierarchy.md` | What overrides what |
| 10 | `codex-session-workflow.md` | How Codex works in this repo |
| 11 | `current-architecture-snapshot.md` | Live architecture state |
| 12 | `live-repo-reference-map.md` | Where real repos live |
| 23 | `innerlab-platform-capability-architecture.md` | Platform-level capabilities |
| 24 | `exact-module-capability-blueprints.md` | Per-module feature specs |
| 25 | `cross-module-coherence-review.md` | Module boundary checks |
| 26 | `architecture-doc-alignment-and-revalidation.md` | Doc alignment audit |
| **27** | **`claude-cross-reference-pack.md`** | **Review questions (Codex → Claude)** |
| **28** | **`claude-response-log.md`** | **Review answers (Claude → Codex)** |
| 29 | `claude-touchpoint-protocol.md` | When/how Claude reviews happen |
| 30 | `assumption-status-model.md` | How assumptions are tracked |
| 31 | `current-live-safe-buildpack-track.md` | First-wave build readiness |
| 33 | `agent-build-readiness-standard.md` | Build agent requirements |
| 39 | `full-catalog-readiness-ranking.md` | Module readiness ranking |
| 40 | `implementation-agent-handoff-protocol.md` | Handoff from planning to build |

### Per-Module Planning Pack (24 docs each)

Each module folder in `modules/` contains up to 24 numbered documents:

| Range | Layer | Contents |
|-------|-------|----------|
| 01-06 | Core Planning | Product brief, architecture, data model, UX flows, brand/copy, competitive position |
| 07-08 | Capabilities | Feature map, capability blueprint |
| 09-12 | Build Pack | Tech stack, env vars, Codex build prompts, implementation details |
| 13 | Graduation | Handoff checklist for moving to a real repo |
| 14-18 | Implementation Readiness | Launch decisions, testing strategy, deployment plan, monitoring, build worklist |
| 19-23 | Launch Readiness | Visual QA, copy review, analytics plan, accessibility, risk/privacy/safety |
| 24 | Repo Kickoff | Clean start prompt for the implementation agent |

---

## Review Workflow Prompt

Copy-paste this prompt to start a review session:

```
Read these files for context:
1. MBSPlatform/INNERLAB_MODULE_REVIEW_GUIDE.md (this file)
2. Innerlab Incubator/docs/27-claude-cross-reference-pack.md (questions)
3. Innerlab Incubator/docs/28-claude-response-log.md (prior answers)

Then for each module that has a real repo (DreamLens, Arcana, Rituals, Bonds):
- Read the module's CLAUDE.md or AGENTS.md
- Inspect server code for standards compliance
- Check il_* integration against Layer 2 rules
- Verify auth, GDPR, logging, validation patterns

Also review:
- Innerlab Incubator planning docs for modules without repos yet
- Compare incubator assumptions against live runtime code

Append all findings to: Innerlab Incubator/docs/28-claude-response-log.md
Use the standard round format with question IDs and statuses.
```

---

## Quick Reference: Module Collection Prefixes

| Module | Prefix | Status |
|--------|--------|--------|
| CWG | `cwg_*` | Live |
| FlowState | `yoga_*` | Live |
| DreamLens | `dream_*` | Built, not deployed |
| Arcana | `tarot_*` | Built, not deployed |
| Rituals | `ritual_*` | Built, not deployed |
| Bonds | `bond_*` | In progress |
| Archetypes | `archetype_*` | Planning only |
| StarMap | `astro_*` | Planning only |
| AstroCompass | `astrocart_*` | Planning only |
| InnerQuest | `quest_*` | Planning only |
| LifeMap | `lifemap_*` | Planning only |
| Nexus | `nexus_*` | Planning only |
| BreathArc | `breath_*` | **FROZEN** |

---

## Quick Reference: Standards Checklist

When reviewing any module, verify:

- [ ] Winston logger (never console.log, never pino)
- [ ] Response format: `{ success: true/false }`
- [ ] Response helpers: `sendSuccess`, `sendError`, etc.
- [ ] express-validator on all routes (or Zod for TypeScript next-gen)
- [ ] express-rate-limit on API routes
- [ ] CORS before helmet()
- [ ] `req.user` shape: `{ userId, email, name, avatar, isAdmin }`
- [ ] `DB_NAME=inner_lab`
- [ ] `user_id` (snake_case) in all il_* documents
- [ ] `source_module` field on all il_* writes
- [ ] `DELETE /api/user-data` endpoint (GDPR)
- [ ] GDPR filters shared collections by `source_module`
- [ ] GDPR protects identity singletons
- [ ] Dockerfiles ready (frontend nginx + backend)
- [ ] Tests exist (Jest/Vitest + supertest)
- [ ] `createApp()` factory pattern (next-gen builds)
