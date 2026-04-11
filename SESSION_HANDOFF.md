# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: April 11, 2026 (Session 32)
**Git Branch**: main
**Last Commit**: See per-repo commits below
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## SESSION 32 SUMMARY (April 10-11, 2026) — IL Module Domain Convention, Module Coolify Instruction Reviews

### What was done this session:

**1. Domain Convention Decision — All IL Modules → `*.innerlab.ai`**
- Owner decision: all new Inner Lab modules deploy to `{slug}.innerlab.ai` (frontend) and `api.{slug}.innerlab.ai` (backend)
- Standalone products (Arcade, Studio Works) stay on `*.magicbusstudios.com`
- Historical exceptions: CWG (`conversationswithgod.ai`), FlowState (`yoga.magicbusstudios.com`) — keep as-is
- Critical distinction documented: `VITE_PLATFORM_URL` and `PLATFORM_URL` stay at `magicbusstudios.com` (they point to MBS Platform, not to the module itself). Agents repeatedly confuse this.

**2. Five Docs Updated for Domain Convention**
- `~/.claude/CLAUDE.md` (global) — added IL domain convention to Architecture section
- `~/Desktop/CLAUDE.md` — same update
- `MBSPlatform/CLAUDE.md` (root) — new "Module Domain Convention" section with date
- `platform-instructions-for-new-modules/CLAUDE.md` — explicit convention + warning box about PLATFORM_URL vs module domain + fixed env vars (JWT_PUBLIC_KEY, CORS www+non-www, SERVICE_NAME, OPENAI_MODEL)
- `platform-instructions-for-innerlab/CLAUDE.md` — catalog table: all 10 TBD modules assigned `{slug}.innerlab.ai` domains

**3. Module Coolify Instruction Reviews — 5 Modules Reviewed**
Reviewed compile/deploy instructions from 5 module agents. Found recurring patterns of mistakes:

| Module | Issues Found | Status |
|--------|-------------|--------|
| Rituals | 5 (domain, JWT, CORS, invented env var, no GDPR) | ✅ Corrected — v2 approved |
| StarMap | 5 (domain, hallucinated gpt-5.4-mini, VITE_PLATFORM_URL wrong, JWT note, no GDPR) | ⏳ Correction sent twice (agent ignored first round) |
| InnerQuest | 4 (domain, JWT either/or, CORS, JWT_PUBLIC_KEY escaping) | ⏳ Correction sent |
| LifeMap | 7 (domain, JWT, JWT escaping, gpt-4o model, no VITE_PRODUCT_SLUG, GDPR scope, MBS CORS line) | ⏳ Correction sent |
| Archetypes | 5 (domain, JWT, JWT escaping, CORS, no GDPR + MBS CORS line) | ⏳ Correction sent |

**4. MODULE_REVIEW_CHECKLIST.md Created**
Comprehensive checklist for reviewing all future module Coolify instructions. Covers: domain convention, JWT dual-mode, GDPR, env vars, frontend build args, common mistakes, correction prompt template. Includes tracking table of reviewed vs not-yet-reviewed modules.

**5. Recurring Agent Mistakes Identified & Documented**
- Domain: agents default to `*.magicbusstudios.com` for IL modules
- JWT: agents present both keys as "either/or" instead of "both required"
- JWT_PUBLIC_KEY: agents say escape `\n` instead of using Coolify "Is Multiline?" checkbox
- CORS: agents list single origin, miss www variant
- GDPR: often omitted entirely, or scope too narrow (misses `il_*` entries)
- OPENAI_MODEL: hallucinated model names (gpt-5.4-mini)
- VITE_PLATFORM_URL: agents change to innerlab.ai (should stay magicbusstudios.com)
- Invented env vars: ALLOW_TRANSPORT_ONLY_ACCESS (not a standard)
- "Add module to MBS CORS": not needed, server-to-server calls

**6. Correction Prompt Convention Established**
- All correction prompts end with: "Apply corrections and regenerate, OR respond with questions. Questions will be routed to the MBS Platform architecture agent."
- Added explicit "Do NOT change PLATFORM_URL or VITE_PLATFORM_URL" warning to all domain corrections

### Files changed this session:

| File | Change |
|------|--------|
| `~/.claude/CLAUDE.md` | Added IL domain convention |
| `~/Desktop/CLAUDE.md` | Added IL domain convention |
| `CLAUDE.md` (root) | New "Module Domain Convention" section |
| `platform-instructions-for-new-modules/CLAUDE.md` | Domain convention, warning box, env var fixes |
| `platform-instructions-for-innerlab/CLAUDE.md` | Catalog table — 10 TBD → assigned domains |
| `MODULE_REVIEW_CHECKLIST.md` | **NEW** — review checklist + tracking |

### What's next (Session 33):
- Continue reviewing remaining 5 module Coolify instructions (Bonds, AstroCompass, Arcana, DreamLens, Nexus)
- Follow up on StarMap, InnerQuest, LifeMap, Archetypes — check if corrected instructions came back
- MBS Platform code work (free tier entitlement, subscribe page) — deferred from Session 31
- FlowState GDPR singleton bug verification (il_user_wellness_profiles)
- CWG feature removal before alignment

---

## SESSION 30 (continued) SUMMARY (April 10, 2026) — Admin Fixes, Module Alignment Prompts, IL Agent Sync, Owner Decisions

### What was done this session:

**1. MBS Admin Product Dashboard — Bug Fixes + Enhancements (code in MBS repo)**
- **Link bug fix**: URLs without `https://` (e.g., `lifemap.magicbusstudios.com`) were treated as relative paths → `magicbusstudios.com/lifemap.magicbusstudios.com`. Added `normalizeUrl()` to admin dashboard and ModuleCard.
- **Homepage shows all 3 categories**: Previously only Inner Lab modules appeared in "What We're Building." Now Arcade and Studio Works products from the API render as cards too.
- **Admin-created products appear on frontend**: Products added via admin dashboard now show as cards on the homepage under their category.
- **Per-category "Add to..." buttons**: Each category table in admin has an "Add to Inner Lab / Arcade / Studio Works" quick-add row.
- **More prominent Add Product button**: Larger, bolder styling.

**2. Inner Lab Module Alignment Prompts — 3 Documents Created**
- **INNERLAB_MODULE_ALIGNMENT.md** — 16-section prompt for the 10 new IL modules (Bonds through Nexus). Covers: visual alignment (concrete design tokens, centralized API client, animation presets, form patterns), entitlement gating (three states, effectivePremium, downgrade), GDPR, backend standards, IL data layer, env vars, deployment. Phased approach (A→B→C→D).
- **CWG_ALIGNMENT.md** — Targeted prompt for CWG. Visual polish + entitlement gating only. Preserves Python/FastAPI patterns, existing SSO/GDPR/il_* integration. Notes feature removal must happen FIRST.
- **FLOWSTATE_ALIGNMENT.md** — Targeted prompt for FlowState. CSS Modules decision (keep vs migrate to Tailwind). Custom toast → Sonner migration. Flags potential GDPR bug: `il_user_wellness_profiles` is identity singleton, should NOT be deleted at app-level.

**3. IL Agent Sync — Cross-Agent Review Completed**
- Reviewed all 6 sync points between MBS Platform agent and IL agent.
- IL agent confirmed all three alignment prompts are accurate and ready to use.
- **PLATFORM_URL dual-use pattern** clarified: frontend `VITE_PLATFORM_URL` = `magicbusstudios.com` (nginx proxies /api/*), backend `PLATFORM_URL` = `api.magicbusstudios.com` (server-to-server). Both correct for their context.
- IL agent fixed: module count (10→12), Bonds + LifeMap added, admin dashboard registration noted.
- FlowState GDPR singleton bug flagged — `il_user_wellness_profiles` may be incorrectly deleted at app-level.

**4. Global Reference Files Updated**
- `~/.claude/reference/innerlab-module-alignment.md` — copied from MBSPlatform for all agents to access
- `~/.claude/CLAUDE.md` — updated module count (11→12), registration note (now includes admin dashboard), added module alignment reference
- `~/Desktop/CLAUDE.md` — same registration update
- `platform-instructions-for-new-modules/CLAUDE.md` — fixed entitlement response (added `isPremium`), fixed redirect URL (`/subscribe/innerlab` not `/billing`), fixed PLATFORM_URL (`api.magicbusstudios.com`), added `JWT_PUBLIC_KEY`

**5. Owner Decisions on Active Module Sessions**
- **FlowState**: Keep CSS Modules (Option A), keep Inter body font, push to dev (not main), Sonner migration approved
- **CWG**: Feature removal before alignment, IL reference hex values in prompt sufficient
- **ALL modules (including CWG + FlowState)**: effectivePremium removed — agents must propose and implement real free/premium split now. Owner reviews later.
- **Reference file distribution**: All 5 reference files (`innerlab-module-alignment.md`, `entitlement-integration.md`, `innerlab-data.md`, `data-sovereignty.md`, `env-standards.md`) manually copied to all laptops in `~/.claude/reference/`

**6. Final Module Prompt Created**
- Single autonomous prompt for any IL module agent — reads 3 reference files, does everything in one pass, proposes real free/premium split

### Per-repo commits this session:

| Repo | Branch | Commits | Description |
|------|--------|---------|-------------|
| MBS | main | `1b08acf` | Fix link normalization + show all categories on homepage |
| MBSPlatform | main | `51fb0fb` | Add module alignment prompts with PLATFORM_URL dual-use |
| MBSPlatform | main | `6f4161d` | Incorporate IL agent review feedback on all three prompts |
| MBSPlatform | main | `e846b08` | End session docs |
| MBSPlatform | main | `1699ac6` | Apply owner decisions (FlowState CSS/font/branch, CWG reference) |
| MBSPlatform | main | `0ff58db` | Module alignment — real free/premium split, no effectivePremium |
| MBSPlatform | main | `ea9481c` | CWG + FlowState — same real gating update |

### What's next (Session 31):
- MBS Platform code work: free_tier entitlement, subscribe page, GDPR email confirmation
- Module alignment: give prompts to individual module agents (prompt ready)
- FlowState GDPR bug verification (il_user_wellness_profiles singleton)
- CWG feature removal before alignment

---

## SESSION 31 SUMMARY (April 8, 2026) — Free Tier Architecture, Admin Product Management, Entitlement Instructions

### What was done:
- Admin dashboard: link editing, product creation, category grouping, unified status source
- Free tier architecture: MBS passes NO limits, modules define their own, IL module picker on MBS
- Dedicated `/subscribe/innerlab` page spec
- Premium→free downgrade flow (not removal)
- GDPR email confirmation architecture (DataDeletionRequest model)
- IL entitlement instructions (INNERLAB_ENTITLEMENT_INSTRUCTIONS.md)
- Global reference file: `~/.claude/reference/entitlement-integration.md`
- Billing→Subscribe rename tracked as deferred

### Per-repo commits:
| Repo | Branch | Commits |
|------|--------|---------|
| MBS | main | `8e4e516` — admin product management |
| MBSPlatform | main | `4a543be` — IL entitlement instructions + architecture update |

---

## NEXT-SESSION PROMPT

See bottom of this document for the comprehensive prompt to start Session 33.

---

## NEXT-SESSION PROMPT — Session 33: Continue Module Reviews + Platform Code Work

```
## Session 33 — Continue Module Reviews + Platform Code Work

This is the MBS Platform architecture think tank (MBSPlatform repo — no code here).

### Context
Session 32 established the domain convention (all new IL modules → *.innerlab.ai) and reviewed Coolify deployment instructions from 5 module agents (Rituals, StarMap, InnerQuest, LifeMap, Archetypes). Rituals v2 was approved. The others had corrections sent back. A MODULE_REVIEW_CHECKLIST.md was created as a comprehensive reference for reviewing all module instructions.

### Read first
- MBSPlatform/SESSION_HANDOFF.md (Session 32 summary)
- MBSPlatform/MODULE_REVIEW_CHECKLIST.md (the review checklist + tracking table)
- MBSPlatform/FUTURE_WORK_TODO.md

### Session 32 decisions (already applied to docs):
- All new IL modules deploy to {slug}.innerlab.ai / api.{slug}.innerlab.ai
- CWG (conversationswithgod.ai) and FlowState (yoga.magicbusstudios.com) are historical exceptions
- PLATFORM_URL and VITE_PLATFORM_URL always stay at magicbusstudios.com (MBS Platform, not module)
- All correction prompts end with "ask questions or regenerate" pattern — questions route to this agent

### Immediate work:
1. **Check for returned corrections** from StarMap, InnerQuest, LifeMap, Archetypes agents
2. **Review remaining 5 modules** when owner pastes their instructions: Bonds, AstroCompass, Arcana, DreamLens, Nexus
3. Use MODULE_REVIEW_CHECKLIST.md for every review — update the tracking table as modules are reviewed

### Deferred platform code work (from Session 31):
- Free tier entitlement (subscribe-free endpoint, free_tier type, downgrade handling)
- Subscribe page (billing→subscribe rename, /subscribe/innerlab)
- GDPR email confirmation (DataDeletionRequest model)
- FlowState GDPR singleton bug verification
- CWG feature removal before alignment

### Owner actions still pending (not code):
- Docker prune on VPS
- Redeploy MBS frontend + backend on Coolify
- Create 6 Stripe products / 12 prices in Stripe Dashboard
- BTCPay API key regeneration
- CWG promote test → dev
```
