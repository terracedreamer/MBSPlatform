# SESSION HANDOFF — MBS Platform Architecture Planning

**Last Updated**: March 24, 2026
**Session Type**: Architecture planning (no code written yet)
**Git Branch**: N/A — no code exists yet
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git

---

## WHERE WE LEFT OFF

We were discussing architecture decisions and had just finalized the **B+C hybrid frontend model** (modules work standalone AND participate in a unified Inner Lab dashboard for subscribers).

**The NEXT step is**: Inspect the CWG MongoDB database to understand what data actually exists, so we can design:
- What migrates to `mbs_platform` (user identity)
- What migrates to `inner_lab` database with `cwg_` prefix (product-specific data)
- What might eventually become shared `il_*` tables (consciousness profile, mood, state)

**Start the next session by asking**: "Can you connect to / look at the CWG MongoDB database so we can see what collections and fields exist?"

---

## THREE-LAYER ARCHITECTURE (Agreed)

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 1: MBS Platform  (DB: mbs_platform)                   │
│  SSO + billing + entitlements — serves ALL 22 products       │
│  One backend container. No frontend (API only).              │
│  Domain: platform.magicbusstudios.com:3002                   │
└──────────┬───────────────────────────────┬───────────────────┘
           │                               │
┌──────────▼───────────────────┐   ┌───────▼──────────────────┐
│  Layer 2: Inner Lab          │   │  Layer 3: Standalone     │
│  Ecosystem                   │   │  Products                │
│                              │   │                          │
│  ALL Inner Lab modules       │   │  Arcade (5 games)        │
│  share ONE database:         │   │  Studio Works (6 tools)  │
│  DB: inner_lab               │   │                          │
│                              │   │  Each = own container    │
│  Each module = separate      │   │  Each = own database     │
│  backend container but       │   │  Only use MBS Platform   │
│  same database               │   │  for SSO. No shared data │
│                              │   │  between them.           │
│  Shared tables: il_*         │   │                          │
│  Module tables: cwg_*, yoga_*│   │                          │
│  etc.                        │   │                          │
│                              │   │                          │
│  Inner Lab Core container    │   │                          │
│  (future): daily briefing,   │   │                          │
│  cross-module insights,      │   │                          │
│  consciousness engine        │   │                          │
└──────────────────────────────┘   └──────────────────────────┘
```

---

## ARCHITECTURE DECISIONS (All Confirmed)

### 1. Fresh `inner_lab` database
- Create new `inner_lab` database (don't reuse `conversations_with_god`)
- Migrate CWG's ~10 users to `mbs_platform.users`
- Migrate CWG product data to `inner_lab` with `cwg_` prefix
- Old `conversations_with_god` DB stays untouched as backup

### 2. User identity lives ONLY in `mbs_platform`
- MBS Platform is the single source of truth for who the user is
- `mbs_platform.users`: email, name, avatar, google_id, preferences
- Inner Lab does NOT maintain its own user table
- Inner Lab stores only spiritual/consciousness data in `inner_lab.il_profiles`, linked by `user_id`

### 3. Shared Inner Lab data — TBD (need to inspect CWG DB first)
- We know Inner Lab modules will share consciousness/spiritual data
- We don't know the exact schema yet
- Need to inspect CWG's actual MongoDB collections to understand what exists
- Shared tables will use `il_` prefix (e.g., `il_profiles`, `il_consciousness`, `il_state`)
- Module-specific tables use module prefix (e.g., `cwg_conversations`, `yoga_sessions`)

### 4. Inner Lab Core container — build later
- No existing backend infrastructure to integrate with
- Only CWG exists as a built module right now
- Build the Core container when 2-3 modules exist and cross-module intelligence has value
- Core will handle: daily briefing, cross-module insights, check-in, state engine, consciousness profile API

### 5. Frontend — B+C Hybrid model
**Each module = standalone website (Path C)**:
- conversationswithgod.ai → CWG (own frontend container)
- yoga.magicbusstudios.com → FlowState (own frontend container)
- breatharc.innerlab.ai → BreathArc (own frontend container, when built)
- Each works independently, login redirects to MBS Platform SSO

**Inner Lab dashboard = unified experience (Path B)**:
- innerlab.ai → becomes a logged-in dashboard for Inner Lab All Access subscribers
- innerlab.ai/today → daily briefing
- innerlab.ai/insights → cross-module intelligence
- innerlab.ai/profile → unified consciousness profile
- Only available to users with `category_access: "innerlab"` entitlement

**Tiered experience**:
- `product_pass: "cwg"` → standalone CWG only, no cross-module features, upsell to Inner Lab
- `category_access: "innerlab"` → all modules + unified dashboard + cross-module intelligence
- `mbs_all_access` → everything across all MBS products

### 6. Arcade & Studio Works — no shared data
- Fully independent products
- Only share SSO through MBS Platform
- Each has own database, own containers
- Maybe merge/share data in the future, but not now

### 7. Admin — simple flag
- `is_admin` boolean on user record
- Admin = your email + a few others
- No complex role-based access for now

---

## INNER LAB MODULE DEPLOYMENT MODEL

Each Inner Lab module = **separate backend container** (NOT one monolith). Reasons:
- Modules could grow very complex individually
- One crashing module shouldn't take down the whole Inner Lab
- Independent development and deployment
- Can be as simple or complex as needed

BUT all Inner Lab containers connect to the **same database** (`inner_lab`):
- Shared data is just tables in the same DB, not API calls between services
- Daily briefing (when built) reads from all module tables directly — no need to call 11 backends
- Each module reads/writes its own prefixed tables + reads shared `il_*` tables

---

## BUILD ORDER (Agreed)

1. **MBS Platform** (Layer 1) — SSO + billing + entitlements. Unblocks all 22 products.
2. **Refactor CWG** — Migrate to `inner_lab` DB, use MBS Platform for auth, `cwg_` prefix on all tables
3. **Build next module** (BreathArc? StarMap?) — standalone, writes to `inner_lab` DB with its own prefix
4. **Build Inner Lab Core** + unified dashboard — when 2-3 modules exist and cross-module intelligence has value
5. **Connect Arcade/Studio Works** — add `checkAccess` middleware to each product backend

---

## REFERENCE DOCUMENTS IN THIS REPO

| File | What it contains |
|------|-----------------|
| `CLAUDE.md` | Project-level instructions and architecture spec |
| `MBS_Platform_Technical_Architecture.docx` | Full MBS Platform spec (SSO, billing, entitlements, Stripe, API endpoints, schemas) |
| `marketing-docs/InnerLab_Product_Brief.docx` | Inner Lab product vision, 11 modules, 4-step framework, differentiators |
| `marketing-docs/ConversationsWithGod_Product_Brief.docx` | CWG product detail — 21 guides, 8 feature categories, 36 languages |
| `marketing-docs/MagicBusStudios_Brand_Architecture.docx` | MBS brand architecture |
| `marketing-docs/MagicBusStudios_Company_Overview.docx` | MBS company overview |
| `marketing-docs/TheArcade_Marketing_Brief.docx` | Arcade product line brief |
| `ChatGPT-architecture/INNER_LAB_MASTER.md` | Inner Lab shared architecture (from ChatGPT session) — single DB, shared collections, module prefixes |
| `ChatGPT-architecture/CWG_AGENT_PROMPT.md` | CWG refactoring instructions for Inner Lab alignment |
| `ChatGPT-architecture/YOGA_AGENT_PROMPT.md` | Yoga/FlowState refactoring instructions |
| `ChatGPT-architecture/NEW_APP_AGENT_PROMPT.md` | Template for building new Inner Lab modules |
| `ChatGPT-architecture/Inner Lab Blueprint.docx` | Older "Spiritual OS" blueprint (may not be current) |

---

## WHAT'S STILL OPEN

- [ ] Inspect CWG MongoDB database — see actual collections and fields
- [ ] Design the `il_*` shared table schema based on what CWG data exists
- [ ] Decide which CWG data stays module-specific vs becomes shared
- [ ] Finalize Inner Lab Core API surface (when ready to build it)
- [ ] Decide exact domains for future modules (e.g., `breatharc.innerlab.ai` vs `breatharc.magicbusstudios.com`)
- [ ] No code has been written yet — project is spec-only

---

## GLOBAL CONVENTIONS REMINDER

- Node.js v22, Express, MongoDB (Mongoose), React (Vite)
- Auth: Google SSO only (no email/password) — JWT
- MongoDB: same cluster, DB-per-app pattern
- Stripe: existing MBS account, prefixed by category ([IL], [Arcade], [SW], [MBS])
- Email: SendGrid (noreply@magicbusstudios.com)
- Deployment: Coolify (self-hosted VPS)
- Logger (never console.log), { success: true/false } response format
- See global CLAUDE.md for full conventions
