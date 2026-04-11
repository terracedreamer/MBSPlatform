# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: April 11, 2026 (Sessions 31 + 32)
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
- ~~MBS Platform code work (free tier entitlement, subscribe page)~~ — **DONE Session 31** (commit `0fc98c4`, Chrome-verified live)
- ~~FlowState GDPR singleton bug verification~~ — **DONE Session 31** (NOT a bug, il_user_wellness_profiles correctly protected)
- **Owner actions**: Run seed script, Docker prune, Stripe products, BTCPay, CWG promote test→dev
- GDPR email confirmation rollout to child apps
- CWG feature removal before alignment

---

## SESSION 31 SUMMARY (April 10, 2026) — Free Tier Entitlements, Subscribe Pages, GDPR Email Confirmation, Coming Soon

### What was done this session:

**1. Entitlement Model + Free Tier Support (code in MBS repo)**
- Added `free_tier` to Entitlement model type enum (three-state: not_subscribed → free_tier → premium)
- Rewrote `POST /api/entitlements/subscribe-free` — two modes: single product `{ product: "cwg" }` and category+modules `{ category: "innerlab", modules: ["cwg", "flowstate"] }`
- Rewrote `GET /api/entitlements/category/:cat` — returns `{ hasAccess, isPremium, reason }` + `products[]` with `registered: true/false` per product
- Updated `checkAccess()` — returns `isPremium: true/false` explicitly, removed `limits` from response
- Stripe webhook `customer.subscription.deleted` — auto-creates `free_tier` entitlement on cancellation (premium→free downgrade, not removal)
- Removed `freeTierLimits` from all 23 products in products.js (MBS passes NO limits — apps enforce own)
- Updated entitlement middleware — matches `free_tier` type, removed `limits` from `req.entitlement`

**2. Subscribe Page Rename (code in MBS repo)**
- `SubscribePage.jsx` — renamed from BillingPage, updated SEO, success/cancel URLs, added IL modules link
- `SubscribeInnerLabPage.jsx` — new dedicated page at `/subscribe/innerlab` with 12 IL module cards as checkboxes, select all, sticky register bar, premium upsell
- Routes: `/subscribe` (SubscribePage), `/subscribe/innerlab` (SubscribeInnerLabPage), `/subscribe/individual` (IndividualPlansPage)
- Redirects: `/billing` → `/subscribe`, `/billing/individual` → `/subscribe/individual`
- Marketing `/subscribe` (newsletter) moved to `/newsletter`
- Updated all `/billing` references in AccountPage, IndividualPlansPage, ProductPickerPage, email templates

**3. GDPR Email Confirmation (code in MBS repo)**
- `DataDeletionRequest` model — user_id, level (app/category/full), target, confirmation_token, status (pending/confirmed/expired/completed), 24h TTL
- `DELETE /api/auth/account` — no longer deletes immediately. Creates DataDeletionRequest + sends confirmation email. Checks for existing pending requests (409). Requires email on account.
- `GET /api/user-data/confirm-delete/:token` — validates token, checks expiry, executes deletion cascade (full or category), returns styled HTML result page
- `DELETE /api/user-data/category/:category` — category-level deletion request with email confirmation
- `sendDeletionConfirmationEmail()` — red-themed HTML email, 24h expiry, level descriptions
- `executeFullDeletion(userId)` — cascade to child apps, cancel Stripe, delete from 14 mbs_platform collections, anonymize ConsentAuditLog, cross-DB inner_lab cleanup
- Account settings UI — "Confirmation email sent" state with CheckCircle icon, explanatory text, button text changed to "Send Confirmation Email"
- User data routes registered at `/api/user-data` in server/index.js

**4. Coming Soon Toggle**
- Seed script `server/seeds/setComingSoon.js` — sets 5 apps to `coming_soon`: brokenchain, mindhacker, whisperinghouse, fakeartist, moviepicker
- Script needs to be run against production DB (owner action)

**5. FlowState GDPR Bug Verification**
- Checked YogaGhost/server/index.js — bug NOT present. Line 696 explicitly excludes `il_user_wellness_profiles` from app-level deletion (identity singleton protected).

**6. Cleanup**
- Deleted 4 OneDrive -MSI conflict files (CURRENT_STATUS-MSI.md, FUTURE_WORK_TODO-MSI.md, SESSION_HANDOFF-MSI.md, PHASE_3A_MIGRATION_REPORT-MSI.md)
- Removed `freeTierLimits` display from ProductPickerPage (40+ lines of `formatLimits()`)

**7. Exhaustive Documentation Sweep (commit `38e68c8`)**
- Grep'd entire repo for stale `/billing` URLs, `freeTierLimits`, `effectivePremium` patterns
- Fixed 14 files: platform-instructions, alignment prompts, orchestration guide, audit annotations, new modules CLAUDE.md, .gitignore

**8. Phase Report Consolidation (commit `ed35545`)**
- Merged all 9 phase reports (3,113 lines across 10 files in `phase-reports/`) into single `archive/PHASE_REPORTS_CONSOLIDATED.md` (3,121 lines)
- Historical archive header added with table of contents
- Updated `CLAUDE.md` folder structure to reflect `phase-reports/` removal

**9. Chrome Verification (Post-Deploy) — April 11, 2026**
- MBS auto-deployed via Coolify on push to main
- Verified 10 pages/endpoints live:
  - `magicbusstudios.com/` — all 23 products, nav, footer links to `/subscribe`
  - `magicbusstudios.com/subscribe` — 4 plan cards, monthly/annual toggle, free tier messaging, active entitlement shown
  - `magicbusstudios.com/subscribe/innerlab` — 12 module picker, "Register Free", CWG+FlowState active, 10 coming soon
  - `magicbusstudios.com/billing` — correctly redirects to `/subscribe`
  - `magicbusstudios.com/auth/login` — Google SSO above email/password, forgot password link
  - `magicbusstudios.com/auth/signup` — Google SSO, name/email/password, age consent, ToS
  - `api.magicbusstudios.com/api/health` — `{"success":true,"service":"mbs-platform","database":true}`
  - `api.magicbusstudios.com/api/auth/public-key` — RS256 public key served
  - `api.magicbusstudios.com/api/entitlements/subscribe-free` — returns 401 (route exists, requires auth)
  - `innerlab.ai` — placeholder page live with all 12 modules
- `/products` page shows "Loading products..." — needs seed script run (owner action)
- `/settings` and `/subscribe/individual` — require login to render full React SPA

### Per-repo commits this session:

| Repo | Branch | Commits | Description |
|------|--------|---------|-------------|
| MBS | main + development | `0fc98c4` | Session 31 — free tier entitlements, subscribe pages, GDPR email confirmation |
| MBSPlatform | main | `68fe106` | End session 31 docs |
| MBSPlatform | main | `38e68c8` | Exhaustive doc sweep — fix stale /billing, freeTierLimits, effectivePremium |
| MBSPlatform | main | `ed35545` | Consolidate 9 phase reports into single archive file |

### What's next (Session 32 picked up):
- ~~Chrome verification~~ — **DONE** (10 pages/endpoints verified, see item 9 above)
- ~~Redeploy MBS on Coolify~~ — **DONE** (auto-deployed on push to main)
- **Owner actions** (still pending): Run seed script, Docker prune, create Stripe products, BTCPay API key regen, CWG promote test→dev
- **GDPR email confirmation rollout to child apps**: IL, CWG, FlowState, all 11 standalone apps
- **Module alignment**: Give prompts to individual module agents (prompts ready from Session 30)

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
- **INNERLAB_MODULE_ALIGNMENT.md** — 16-section prompt for the 10 new IL modules (Bonds through Nexus). Covers: visual alignment (concrete design tokens, centralized API client, animation presets, form patterns), entitlement gating (three states, real free/premium split, downgrade), GDPR, backend standards, IL data layer, env vars, deployment. Phased approach (A→B→C→D).
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

**7. StarMap Env Var Clarification → All IL Modules**
- PLATFORM_URL (backend) = `https://api.magicbusstudios.com` — NOT innerlab.ai. This is where entitlement checks live.
- VITE_GOOGLE_CLIENT_ID NOT needed for IL modules — they delegate auth to innerlab.ai/auth/login and never render a Google sign-in button.
- Created env var clarification prompt for all module agents (correct PLATFORM_URL + remove VITE_GOOGLE_CLIENT_ID)

**8. Domain Convention Change (owner-applied to global CLAUDE.md)**
- New IL modules deploy to `{slug}.innerlab.ai` / `api.{slug}.innerlab.ai`
- Standalone products (Arcade, Studio Works) deploy to `*.magicbusstudios.com`
- CWG (`conversationswithgod.ai`) and FlowState (`yoga.magicbusstudios.com`) are historical exceptions
- Frontend VITE_BACKEND_URL for IL modules is now `https://api.{slug}.innerlab.ai` (not `api.{slug}.magicbusstudios.com`)

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

## SESSION 30 (planning) SUMMARY (April 8, 2026) — Free Tier Architecture, Admin Product Management, Entitlement Instructions

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

## NEXT-SESSION PROMPT — Session 33: Module Reviews + GDPR Rollout

```
## Session 33 — Module Reviews + GDPR Rollout

This is the MBS Platform architecture think tank (MBSPlatform repo — no code here). The actual code work happens in the MBS/ repo.

### Context
Session 31 built and deployed: free_tier entitlements, subscribe pages (`/subscribe`, `/subscribe/innerlab`), GDPR email confirmation, coming soon seed. All Chrome-verified live (10 pages/endpoints confirmed working April 11). MBS auto-deployed via Coolify.
Session 32 established the domain convention (all new IL modules → *.innerlab.ai) and reviewed Coolify deployment instructions from 5 of 10 module agents (Rituals approved, StarMap/InnerQuest/LifeMap/Archetypes had corrections sent back).

### Read first
- MBSPlatform/SESSION_HANDOFF.md (Sessions 31 + 32 summaries)
- MBSPlatform/MODULE_REVIEW_CHECKLIST.md (review checklist + tracking table)
- MBSPlatform/FUTURE_WORK_TODO.md

### Key decisions already applied:
- All new IL modules → {slug}.innerlab.ai / api.{slug}.innerlab.ai (Session 32)
- PLATFORM_URL and VITE_PLATFORM_URL always stay at magicbusstudios.com (MBS Platform, not module)
- MBS passes NO limits — only hasAccess + isPremium. Each module enforces its own limits (Session 31)
- Free tier is LIVE — three-state model: not_subscribed → free_tier → premium (Session 31)

### Priority 1 — Continue module Coolify instruction reviews:
1. Check for returned corrections from StarMap, InnerQuest, LifeMap, Archetypes
2. Review remaining 5 modules when owner pastes instructions: Bonds, AstroCompass, Arcana, DreamLens, Nexus
3. Use MODULE_REVIEW_CHECKLIST.md for every review — update tracking table

### Priority 2 — GDPR email confirmation rollout to child apps:
MBS Platform (Layer 1) confirmation flow is built and live. Each child app needs to route through it.
Build order: Inner Lab → CWG → FlowState → 11 standalone apps
Architecture: see FUTURE_WORK_TODO.md "GDPR — Email Confirmation" section

### Priority 3 — Module alignment:
Give alignment prompts to individual module agents (INNERLAB_MODULE_ALIGNMENT.md, CWG_ALIGNMENT.md, FLOWSTATE_ALIGNMENT.md)

### Owner actions still pending (not code — owner must do manually):
- [ ] Run seed script: `node server/seeds/setComingSoon.js` against production DB (/products page stuck on "Loading...")
- [ ] Docker prune on VPS (97% disk): `docker system prune -a -f && docker builder prune -a -f`
- [ ] Create 6 Stripe products / 12 prices in Stripe Dashboard (see FUTURE_WORK_TODO.md)
- [ ] BTCPay API key regeneration (403 error)
- [ ] CWG promote test → dev (requires passphrase)

### Do NOT start building immediately. List all work items, confirm the plan, then I'll tell you which to start on.
```
