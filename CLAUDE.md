# MBS Platform — Architecture & Planning Repository

## Purpose of This Repo
**This is NOT a deployable project.** This is the **architecture think tank** for the MBS Platform ecosystem. It contains:
- Reference CLAUDE.md files that get copied into the actual project folders for building
- Migration plans for CWG (28 collections) and FlowState (7 collections)
- Phase reports and audit records

**Architecture Docs and Brand Overview have moved to `Desktop/Marketing/`.** That folder is now the single source of truth for all marketing briefs and architecture reference documents. Do NOT recreate these folders here.

**No code is built here.** The actual platform code is built inside:
- `MBS/` folder (magicbusstudios.com) — gets the MBS Platform backend + frontend additions
- `Innerlab/` folder (innerlab.ai) — gets the Inner Lab Middleware backend + frontend additions

## How This Repo Is Used

```
MBSPlatform/ (THIS REPO — think tank, no code)
│
├── platform-instructions-for-mbs/CLAUDE.md
│     ↓ Copy into MBS/platform-instructions/
│     MBS/ adds: SSO, billing, entitlements, login page, account settings
│     Deploys to: magicbusstudios.com (2 containers: frontend + backend)
│
├── platform-instructions-for-innerlab/CLAUDE.md
│     ↓ Copy into Innerlab/platform-instructions/
│     Innerlab/ adds: il_* APIs, consciousness, memories, dashboard
│     Deploys to: innerlab.ai (2 containers: frontend + backend)
│
├── platform-instructions-for-cwg/ → CWG/platform-instructions/ (migration)
├── platform-instructions-for-yogaghost/ → YogaGhost/platform-instructions/ (migration)
├── platform-instructions-for-standalone-products/ → each Arcade/SW project (SSO migration)
├── platform-instructions-for-new-modules/ → new IL module projects (starter kit)
│
├── archive/, session docs
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
| innerlab.ai | Marketing + login/signup + dashboard + consciousness + memories + feed | Form handler + il_* middleware APIs | `inner_lab` | `Innerlab/` |

**These are the same Coolify deployments that already exist** — we're adding backend functionality to existing projects, not creating new containers.

## Auth: Four Methods
- Google SSO (primary — existing across MBS)
- Email/Password (traditional signup + login, email verification, password reset, optional 2FA/TOTP with backup codes)
- Nostr authentication (challenge → signature verification)
- LNURL-Auth (Lightning wallet login)

Auth API routes live on MBS Platform (magicbusstudios.com). Login/signup PAGES exist on both magicbusstudios.com and innerlab.ai (each with its own branding). Both call the same MBS Platform auth endpoints. Inner Lab modules redirect to innerlab.ai/auth/login. Standalone products redirect to magicbusstudios.com/auth/login.

**JWT Signing**: RS256 asymmetric (Session 11). MBS Platform signs with private key (`JWT_PRIVATE_KEY`), all 15 child apps verify with public key (`JWT_PUBLIC_KEY`). Dual-mode: RS256 first, HS256 fallback active (Phase 2 removal attempted Session 12, reverted). `JWT_SECRET` still required on all services. Public key available at `GET /api/auth/public-key`.

## Payments: Dual System
- Stripe (subscriptions + one-time) — primary
- BTCPay / Lightning (sats) — alternative

Both handled at Layer 1. No per-product payment handling.

## Login Pages (Two Sites)

| Site | Login URL | Signup URL | Branding | Used by |
|---|---|---|---|---|
| magicbusstudios.com | /auth/login | /auth/signup | MBS branding (or `?brand=innerlab` for IL) | Arcade, Studio Works, direct MBS visitors |
| innerlab.ai | /auth/login | /auth/signup | Inner Lab branding | CWG, FlowState, all Inner Lab modules, innerlab.ai visitors |

Both login pages call the SAME MBS Platform auth API endpoints. Both support all 4 auth methods (Google SSO, email/password, Nostr, LNURL) + 2FA. The user account is identical regardless of where they signed up.

## Entitlement Types
- `product_pass` — access to one specific product (one-time or subscription)
- `category_access` — access to all products in a category (Inner Lab / Arcade / Studio Works)
- `mbs_all_access` — access to everything

## Product Catalog
### Inner Lab (12 modules)
`cwg` (Active), `flowstate` (Active), `bonds`, `lifemap`, `starmap`, `astrocompass`, `arcana`, `archetypes`, `dreamlens`, `rituals`, `innerquest`, `nexus` (all Coming Soon)

### The Arcade (5 games)
`brokenchain`, `mindhacker`, `triviaroast`, `whisperinghouse`, `fakeartist`

### Studio Works (6 tools)
`wildlens`, `lazychef`, `tasktracker`, `tutor`, `smartcart`, `moviepicker`

**Do NOT hardcode slugs** — store in config or database.

## Core Data Models (mbs_platform DB)
- **User**: email, name, avatar, password_hash?, google_id?, nostr_npub?, lnurl_linking_key?, auth_methods [String] (e.g. ["email","google","nostr","lnurl"]), email_verified, email_verification_token?, password_reset_token?, totp_enabled, totp_secret?, totp_backup_codes?, preferred_language, preferences {}, consent_preferences {}, is_admin, stripe_customer_id?, referral_code?, referred_by?, referral_count, created_at, updated_at, last_login
- **Entitlement**: user_id, category, type, product, status, stripe_subscription_id?, purchased_at, expires_at?, trial_ends_at?
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
- DELETE /api/auth/account — GDPR full account delete (see Data Deletion section below)
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

**Phase 1 Addendum (all deployed)**
- POST /api/auth/signup — email/password registration
- POST /api/auth/login — email/password login
- POST /api/auth/forgot-password — send reset email
- POST /api/auth/reset-password — reset with token
- POST /api/auth/2fa/setup — enable TOTP 2FA
- POST /api/auth/2fa/verify — verify TOTP code
- POST /api/auth/refresh — refresh JWT
- GET /api/auth/email-preferences — get email preferences
- PUT /api/auth/email-preferences — update email preferences
- GET /api/auth/unsubscribe/:token — one-click unsubscribe
- GET /api/admin/stats — admin dashboard stats
- GET /api/admin/users — admin user list
- POST /api/promos — create promo code
- POST /api/promos/validate — validate promo code
- POST /api/referrals — create referral
- GET /api/referrals/stats — referral stats

## Data Deletion (Three-Level Architecture)

"Delete my data" means different things depending on where the user triggers it:

| Level | Where | What gets deleted | What stays |
|-------|-------|-------------------|-----------|
| **App-level** | Settings within each app (e.g., WildLens Settings → "Delete my data") | Only that app's data in its own database | MBS account, entitlements, all other apps' data |
| **Category-level** | magicbusstudios.com/settings | All data from all apps in one category (e.g., all Inner Lab, all Arcade, or all Studio Works) | MBS account, other categories' data |
| **Full account** | magicbusstudios.com/settings | Everything — user record, entitlements, transactions, data across ALL apps | Nothing |

**Rules:**
- A standalone app's "Delete my data" button ONLY calls its own `DELETE /api/user-data` endpoint. It does NOT touch the MBS Platform user account or any other app.
- Only the MBS Platform (magicbusstudios.com) has authority to do cross-product or full account deletion.
- Category-level deletion: MBS Platform calls `DELETE /api/user-data` on each app in the category.
- Full account deletion: MBS Platform calls every product's delete endpoint, then deletes the user record from `mbs_platform`.

**Each standalone app must implement:** `DELETE /api/user-data` — deletes all data for the authenticated user from that app's database only.

## Data Migration

### CWG (28 collections) — Analysis: `CWG/MBS_DATABASE_MIGRATION_PLAN.md`
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

## Build Order (All 5 Phases Complete — 2026-03-28)
1. **Phase 1: MBS Platform** — SSO + billing + entitlements at magicbusstudios.com ✅ DONE
2. **Phase 2: Inner Lab Middleware** — il_* APIs + dashboard + auth pages at innerlab.ai ✅ DONE
3. **Phase 3: CWG Migration** — data (20 users, 28 collections) + backend/frontend refactor ✅ DONE (on `test` branch)
4. **Phase 4: FlowState Migration** — backend/frontend refactor, 0 users ✅ DONE (live on production)
5. **Phase 5: Standalone Products** — all 11 Arcade + Studio Works apps SSO migrated ✅ DONE (all verified live)

### Future Build Work
6. **Premium feature gating** — entitlement infrastructure wired, enforcement TBD per product
7. **New Inner Lab modules** — born into the architecture using `platform-instructions-for-new-modules/`
8. **Inner Lab Frontend enhancements** — daily briefing, cross-module insights (when enough data)

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
├── platform-instructions-for-standalone-products/ ← Copied to each Arcade/Studio Works project
│   ├── PLATFORM_MIGRATION.md                   ← SSO migration steps (simpler than IL modules)
│   └── README.md                               ← Agent intro
│
├── platform-instructions-for-new-modules/      ← Starter kit for new Inner Lab modules
│   ├── CLAUDE.md                               ← Full module template with architecture context
│   └── README.md                               ← How to use
│
├── (MOVED) Brand Overview/                     ← Now lives at Desktop/Marketing/Brand Overview/
├── (MOVED) Architecture Docs/                  ← Now lives at Desktop/Marketing/Architecture Docs/
│
├── audits/                                      ← Compliance and brand audits
│   ├── 2026-03-29-compliance-audit.md           ← 17-project standards audit
│   ├── 2026-03-29-arcade-brand-audit.md         ← Arcade Brand DNA audit
│   └── 2026-03-30-platform-review.md            ← SSO/entitlements/GDPR/billing review
│
├── archive/                                     ← Old reference + historical reports
│   ├── PHASE_REPORTS_CONSOLIDATED.md            ← All 9 phase reports (Phases 1-5, March 2026)
│   ├── ChatGPT-architecture/
│   └── MBS_Platform_Technical_Architecture.docx
│
├── CLAUDE.md                                   ← This file
├── SESSION_HANDOFF.md                          ← Session continuity
├── CHANGELOG.md                                ← Decision history
├── CURRENT_STATUS.md                           ← Current state
├── FUTURE_WORK_TODO.md                         ← Prioritized tasks
├── ORCHESTRATION_GUIDE.md                      ← Build workflow + paste-ready prompts
└── .gitignore
```

### How this repo pushes instructions outward:

**To project folders (already copied):**
- `platform-instructions-for-mbs/` → copied to `MBS/platform-instructions/` — agent reads and builds Layer 1
- `platform-instructions-for-innerlab/` → copied to `Innerlab/platform-instructions/` — agent reads and builds Layer 2
- `platform-instructions-for-cwg/` → copied to `CWG/platform-instructions/` — agent reads and migrates CWG
- `platform-instructions-for-yogaghost/` → copied to `YogaGhost/platform-instructions/` — agent reads and migrates FlowState
- `platform-instructions-for-standalone-products/` → copied to each Arcade game and Studio Works app as `platform-instructions/` — agent reads and does SSO migration
- `platform-instructions-for-new-modules/` → copied to new Inner Lab module projects as `platform-instructions/` — agent reads and builds from template

Each project's CLAUDE.md has been updated with a note to check `platform-instructions/` before starting work.

**Marketing & Architecture Docs (single source of truth):**
- All brand briefs and architecture docs live at `Desktop/Marketing/`
- `Desktop/Marketing/Brand Overview/` — product briefs, pricing reference
- `Desktop/Marketing/Architecture Docs/` — technical architecture, database schemas, module building guide, dashboard vision
- Marketing agent points to `Desktop/Marketing/` and reads `_INSTRUCTIONS.md` first
- These folders were previously duplicated here in MBSPlatform — that duplication was removed Session 14 (April 2, 2026)

### When you make changes:
1. Update architecture/brand files directly in `Desktop/Marketing/` (the single source of truth)
2. Update platform-instructions files in this repo, push to GitHub
3. Re-copy the relevant `platform-instructions-for-*` folder to the target project
4. Open Claude Code in that project — the agent picks up the new instructions

## What NOT to Do
- Do NOT write code in this repo — this is architecture/planning only
- Do NOT build auth into any product directly — all auth through MBS Platform SSO
- Do NOT build Stripe into CWG or any product directly — all billing through MBS Platform
- Do NOT hardcode product slugs — config or database
- Do NOT modify another module's prefixed collections (e.g., CWG should never write to yoga_*)
- Do NOT assume email exists on all users — Nostr/LNURL users are pseudonymous

---

## Env Var Standards (Canonical Names)

This repo is architecture-only (no code), but when writing instructions for child apps, always use canonical env var names:

| Legacy Name | Canonical Name |
|-------------|---------------|
| `MONGODB_URI` / `MONGO_URI` | `MONGO_URL` |
| `JWT_SECRET_KEY` / `SECRET_KEY` | `JWT_SECRET` |
| `CORS_ORIGIN` / `CLIENT_URL` / `ALLOWED_ORIGINS` | `CORS_ORIGINS` |
| `MONGODB_DATABASE` | `DB_NAME` |
| `MBS_PLATFORM_URL` / `PLATFORM_API_URL` | `PLATFORM_URL` |
| `SENDGRID_FROM_EMAIL` | `FROM_EMAIL` |

**RS256 JWT Keys** (added Session 11):
| Env Var | Where | Notes |
|---------|-------|-------|
| `JWT_PRIVATE_KEY` | MBS Backend ONLY | PEM format. Coolify: must check "Is Multiline?" |
| `JWT_PUBLIC_KEY` | ALL 15 services | PEM format. Coolify: must check "Is Multiline?" |

Full reference: `~/.claude/rules/env-standards.md`

## GDPR — MBS Platform Role

MBS Platform is Layer 1 and **orchestrates** the data deletion cascade. It does NOT expose `DELETE /api/user-data` itself — instead, it **calls** each child app's `DELETE /api/user-data` endpoint during category-level and full-account deletion.

GDPR status by product:
| Status | Products |
|--------|----------|
| Implemented (deployed) | MBS Platform (cascade + deletion UI), Lazy Chef, SmartCart, TaskTracker, AI Tutor, BrokenChain, MindHacker, WildLens, Movie Picker, Whispering House, Fake Artist (stub), Trivia Roast (stub), Inner Lab, CWG, FlowState |
| Not needed | Consciousness (no user accounts — email capture only) |

## Testing

This repo has no code to test. For child app testing standards:
- **Express apps**: Jest + supertest. Always test auth flow (valid/invalid/expired tokens, SSO auto-creation).
- **FastAPI apps**: pytest + httpx AsyncClient.
- **Reference project**: Lazy Chef is the gold standard for test coverage.

## Skill References

When working in this architecture repo, these skills provide context for writing instructions:
- `express-backend-patterns` — Express error handler, Winston logger, response helpers
- `fastapi-architecture` — FastAPI project structure, Motor async DB, auth dependency injection
- `frontend-api-client` — Axios client with SSO interceptors, ProtectedRoute
- `mbs-platform-sso` — JWT verification, user ID sync, entitlement checks
- `coolify-deployment` — Dockerfiles, nginx config, env vars, branch mapping
- `mongodb-shared-cluster` — DB-per-app naming, Mongoose/Motor patterns

## Design Tokens

Standard across all MBS products:
- **Fonts**: Space Grotesk (headings), DM Sans (body), Instrument Serif (italic accent), JetBrains Mono (terminal)
- **Colors**: MBS = Cyan/Teal, Inner Lab = Teal-500 `#14b8a6` + Sky-500 `#0ea5e9`, CWG = Blue `#3B82F6` + Purple `#7C3AED`
- **UI**: Dark aesthetic, glass card panels, Framer Motion animations, floating particles

## Backend Standardization

All child apps built from MBS Platform instructions must follow:
- **`req.user` shape**: `{ userId, email, name, avatar }` — standardize on `userId` (never `id` or `_id`)
- **Response format**: `{ success: true, ...data }` or `{ success: false, message: "..." }`
- **Logging**: Winston logger (NEVER `console.log`). Service name via `SERVICE_NAME` env var.
- **Toasts**: Sonner (NEVER react-hot-toast)
- **Route pattern**: authenticate -> validate -> business logic -> respond
- **AI calls**: Always behind a service layer — API keys never in frontend

## Architecture Reference

Full platform architecture document: `~/Desktop/Marketing/Architecture Docs/MBS_Platform_Technical_Architecture.md`

---

## Pending Work
Check `FUTURE_WORK_TODO.md` for pending standards compliance work and backlog items.
