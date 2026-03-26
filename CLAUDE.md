# MBS Platform — Architecture & Planning Repository

## Purpose of This Repo
**This is NOT a deployable project.** This is the **architecture think tank** for the MBS Platform ecosystem. It contains:
- All architecture decisions, specs, and planning documents
- Migration plans for CWG (56 collections) and FlowState (7 collections)
- Reference CLAUDE.md files that get copied into the actual project folders for building
- Marketing docs and product briefs for reference

**No code is built here.** The actual platform code is built inside:
- `MBS/` folder (magicbusstudios.com) — gets the MBS Platform backend + frontend additions
- `Innerlab/` folder (innerlab.ai) — gets the Inner Lab Middleware backend + frontend additions

## How This Repo Is Used

```
MBSPlatform/ (THIS REPO — think tank, no code)
│
├── MBS-platform-reference/CLAUDE.md
│     ↓ Copy into MBS/ project
│     MBS/ adds: SSO, billing, entitlements, login page, account settings
│     Deploys to: magicbusstudios.com (2 containers: frontend + backend)
│
├── InnerLab-middleware/CLAUDE.md
│     ↓ Copy into Innerlab/ project
│     Innerlab/ adds: il_* APIs, consciousness, memories, dashboard
│     Deploys to: innerlab.ai (2 containers: frontend + backend)
│
├── Architecture docs, migration plans, product briefs
└── SESSION_HANDOFF.md for continuity across sessions
```

## Key Design Principle
**The platform answers one question: "Does user X have access to product Y?"** Everything else (billing, promos, referrals, email, analytics) is built on top of that simple access check.

## Three-Layer Architecture (Finalized 2026-03-25)

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 1: MBS Platform                                        │
│  Lives in: MBS/ folder → magicbusstudios.com                 │
│  Frontend: marketing + login (branded) + billing + admin      │
│  Backend:  SSO + entitlements + Stripe + BTCPay + friends     │
│  Database: mbs_platform                                       │
│  Containers: 2 (frontend + backend) — existing Coolify deploy │
└──────────┬──────────────────────────┬────────────────────────┘
           │                          │
┌──────────▼──────────┐       ┌───────▼──────────────────────┐
│  Layer 2: Inner Lab │       │  Layer 3: Standalone         │
│  Lives in: Innerlab/│       │  Products                    │
│  → innerlab.ai      │       │                              │
│                      │       │  Arcade (5 games) — own DBs  │
│  Frontend: marketing│       │  Studio Works (6 tools)      │
│  + dashboard +      │       │    — own DBs                 │
│  consciousness +    │       │                              │
│  memories + feed    │       │  SSO only via Layer 1.       │
│                      │       │  No shared data.             │
│  Backend: il_* APIs │       └──────────────────────────────┘
│  + SendGrid forms   │
│                      │
│  Database: inner_lab│
│  Containers: 2      │
│  (frontend + backend)│
└──────────────────────┘
```

## Deployment — 4 New Containers (Merged Into Existing Projects)

| Domain | Frontend | Backend | Database | Source Folder |
|--------|----------|---------|----------|---------------|
| magicbusstudios.com | Marketing + login + billing + account | Form handler + SSO API + entitlements + Stripe + BTCPay | `mbs_platform` | `MBS/` |
| innerlab.ai | Marketing + dashboard + consciousness + memories + feed | Form handler + il_* middleware APIs | `inner_lab` | `Innerlab/` |

**These are the same Coolify deployments that already exist** — we're adding backend functionality to existing projects, not creating new containers.

## Auth: Three Methods (All Passwordless)
- Google SSO (primary — existing across MBS)
- Nostr authentication (challenge → signature verification)
- LNURL-Auth (Lightning wallet login)

## Payments: Dual System
- Stripe (subscriptions + one-time) — primary
- BTCPay / Lightning (sats) — alternative

Both handled at Layer 1. No per-product payment handling.

## Branded Login
Login page at `magicbusstudios.com/auth/login` changes branding based on origin:

| User comes from | Brand shown | After login, redirect to |
|---|---|---|
| Any Inner Lab module | Inner Lab branding | Back to the module |
| Any Arcade game | MBS branding | Back to the game |
| Any Studio Works tool | MBS branding | Back to the tool |
| innerlab.ai | Inner Lab branding | innerlab.ai |
| magicbusstudios.com | MBS branding | magicbusstudios.com |

## Entitlement Types
- `product_pass` — access to one specific product (one-time or subscription)
- `category_access` — access to all products in a category (Inner Lab / Arcade / Studio Works)
- `mbs_all_access` — access to everything

## Product Catalog
### Inner Lab (11 modules)
`cwg` (Active), `flowstate` (Active), `breatharc`, `starmap`, `astrocompass`, `arcana`, `archetypes`, `dreamlens`, `rituals`, `innerquest`, `nexus` (all Coming Soon)

### The Arcade (5 games)
`brokenchain`, `mindhacker`, `triviaroast`, `whisperinghouse`, `fakeartist`

### Studio Works (6 tools)
`wildlens`, `lazychef`, `tasktracker`, `tutor`, `smartcart`, `moviepicker`

**Do NOT hardcode slugs** — store in config or database.

## Core Data Models (mbs_platform DB)
- **User**: email, name, avatar, google_id, nostr_npub?, lnurl_linking_key?, auth_provider (google|nostr|lnurl), preferred_language, preferences {}, consent_preferences {}, is_admin, referral_code?, referred_by?, referral_count, created_at, updated_at, last_login
- **Entitlement**: user_id, category, type, product, status, stripe_subscription_id?, stripe_customer_id, purchased_at, expires_at?, trial_ends_at?
- **Transaction**: user_id, type, category, product?, amount, currency, stripe_payment_id, description, promo_code?, created_at
- **Promotion**: code (unique), type, value, applies_to, category?, product?, max_uses, current_uses, valid_from, valid_until, created_by
- **Referral**: referrer_id, referral_code, referred_email, referred_user_id?, status, reward_type, reward_value, created_at
- **EmailPreference**: user_id, marketing_emails, product_updates, billing_alerts, promotional_offers, unsubscribe_token
- **Friend**: users (sorted pair), source_product, created_at
- **Invite**: code, from_user_id, product, expires_at, created_at
- **ActiveSession**: user_id, session_token, ip_address (hashed), user_agent, created_at, last_active
- **PushSubscription**: user_id, subscription (endpoint, keys), user_agent, subscribed_at
- **FeatureFlag**: flag_name, enabled, category, description, target_plans, target_users
- **ConsentAuditLog**: user_id, change_type, old_value, new_value, timestamp
- **DataRequest**: user_id, request_type, status, created_at
- **NostrChallenge**: challenge, user_id?, expires_at
- **LnurlChallenge**: k1, status, user_id?, expires_at
- **BtcpayInvoice**: user_id, invoice_id, amount_sats, status, product?, created_at
- **ActivityLog**: user_id, product, category, action, metadata {}, created_at
- **Announcement**: title, message, type, target, active, valid_from, valid_until

## API Endpoints (Layer 1 — MBS Platform at magicbusstudios.com)

**Phase 1 (MVP)**
- POST /api/auth/google — Google SSO
- POST /api/auth/nostr — Nostr authentication
- POST /api/auth/lnurl — LNURL-Auth
- GET /api/auth/me — current user + entitlements
- POST /api/auth/logout — invalidate session
- DELETE /api/auth/account — GDPR delete
- GET /api/entitlements — all entitlements for user
- GET /api/entitlements/:product — check access
- GET /api/entitlements/category/:cat — products in category
- POST /api/billing/checkout — Stripe checkout
- POST /api/billing/portal — Stripe customer portal
- POST /api/billing/webhook — Stripe webhook
- POST /api/billing/btcpay/checkout — BTCPay Lightning checkout
- POST /api/billing/btcpay/webhook — BTCPay webhook
- GET /api/billing/history — transaction history
- GET /api/friends — list friends
- POST /api/friends/invite — create invite
- POST /api/friends/accept — accept invite
- GET /api/health — health check

**Phase 2+**
- Email preferences, admin, promos, referrals (see FUTURE_WORK_TODO.md)

## Data Migration

### CWG (56 collections) — Analysis: `CWG/MBS_DATABASE_MIGRATION_PLAN.md`
- Bucket 1 → `mbs_platform`: User identity, Stripe, consent, sessions, Nostr/LNURL, GDPR
- Bucket 2 → `inner_lab` with `cwg_*` prefix: 39 CWG-specific collections
- Bucket 3 → `inner_lab` with `il_*` prefix: Consciousness, personal history, check-ins, memories, wellness

### FlowState (7 collections) — Analysis: `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md`
- Bucket 1 → `mbs_platform`: User identity, Stripe, email preferences
- Bucket 2 → `inner_lab` with `yoga_*` prefix: Sessions, achievements, community flows
- Bucket 3 → shared: Friends/invites → platform, activity → IL level, health conditions/injuries → `il_*`

### Migration Rules
- Old databases stay untouched as backups (copy, not move)
- Only delete old DBs after weeks/months of verified operation

## Build Order (Final — 2026-03-25)
1. **MBS Platform** — add SSO + billing + entitlements to `MBS/` project (magicbusstudios.com)
2. **Inner Lab Middleware** — add il_* APIs + dashboard to `Innerlab/` project (innerlab.ai)
3. **CWG Migration** — shared data → il_*, product data → cwg_*, identity → mbs_platform
4. **FlowState Migration** — same pattern
5. **New Inner Lab modules** — born into the architecture
6. **Inner Lab Frontend enhancements** — daily briefing, cross-module insights (when enough data)

## Folder Structure

```
MBSPlatform/                                    ← THIS REPO (think tank, no code)
│
├── platform-instructions-for-mbs/              ← Copied to MBS/platform-instructions/
│   ├── CLAUDE.md                               ← Layer 1 build instructions
│   └── README.md                               ← Agent intro
│
├── platform-instructions-for-innerlab/         ← Copied to Innerlab/platform-instructions/
│   ├── CLAUDE.md                               ← Layer 2 build instructions
│   └── README.md                               ← Agent intro
│
├── platform-instructions-for-cwg/              ← Copied to CWG/platform-instructions/
│   ├── PLATFORM_MIGRATION.md                   ← CWG migration steps
│   └── README.md                               ← Agent intro
│
├── platform-instructions-for-yogaghost/        ← Copied to YogaGhost/platform-instructions/
│   ├── PLATFORM_MIGRATION.md                   ← FlowState migration steps
│   └── README.md                               ← Agent intro
│
├── marketing-docs/                             ← Source of truth — synced to Desktop/Marketing/Overview/
│   ├── MagicBusStudios_Brand_And_Company.md    ← Master: company + brand + all products + pricing
│   ├── InnerLab_Product_Brief.md               ← Inner Lab deep dive (11 modules, framework)
│   ├── ConversationsWithGod_Product_Brief.md   ← CWG deep dive (21 guides, features, pricing)
│   └── TheArcade_Marketing_Brief.md            ← Arcade deep dive (5 games, campaigns)
│
├── archive/                                    ← Old reference (absorbed)
│   ├── ChatGPT-architecture/
│   └── MBS_Platform_Technical_Architecture.docx
│
├── CLAUDE.md                                   ← This file
├── SESSION_HANDOFF.md                          ← Session continuity
├── CHANGELOG.md                                ← Decision history
├── CURRENT_STATUS.md                           ← Current state
├── FUTURE_WORK_TODO.md                         ← Prioritized tasks
└── .gitignore
```

### How this repo pushes instructions outward:

**To project folders (already copied):**
- `platform-instructions-for-mbs/` → copied to `MBS/platform-instructions/` — agent reads and builds Layer 1
- `platform-instructions-for-innerlab/` → copied to `Innerlab/platform-instructions/` — agent reads and builds Layer 2
- `platform-instructions-for-cwg/` → copied to `CWG/platform-instructions/` — agent reads and migrates CWG
- `platform-instructions-for-yogaghost/` → copied to `YogaGhost/platform-instructions/` — agent reads and migrates FlowState

Each project's CLAUDE.md has been updated with a note to check `platform-instructions/` before starting work.

**To Marketing folder:**
- `marketing-docs/` contents are copied to `Desktop/Marketing/Overview/`
- Marketing folder has its own README.md and folder structure per product line
- Marketing agent points to `Desktop/Marketing/` and reads `README.md` first

### When you make changes:
1. Update files in this repo (the think tank)
2. Push to GitHub
3. Re-copy the relevant `platform-instructions-for-*` folder to the target project
4. Open Claude Code in that project — the agent picks up the new instructions

## Marketing Folder Sync
When ANY file in `marketing-docs/` changes in this repo:
1. Copy ALL `.md` files from `marketing-docs/` to `Desktop/Marketing/Overview/` (overwrite)
2. Update `Desktop/Marketing/README.md` if the file listing changed
3. Tell the user what was synced

- The Marketing folder at `Desktop/Marketing/` is NOT a git repo — just copy files directly.
- Marketing folder structure:
  - `Overview/` — product briefs (synced from this repo)
  - `Magic Bus Studios/` — MBS brand marketing (social, campaigns)
  - `Inner Lab/` — IL product line marketing
  - `Inner Lab/Conversations with God/` — CWG campaigns, social, guides, emails
- No separate Arcade or Studio Works folders — marketed under MBS brand.

## What NOT to Do
- Do NOT write code in this repo — this is architecture/planning only
- Do NOT modify existing product backends (CWG, FlowState) until after platform + middleware are built
- Do NOT build Stripe into CWG or any product directly — all billing through MBS Platform
- Do NOT hardcode product slugs — config or database
