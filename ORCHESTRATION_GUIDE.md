# MBS Platform — Orchestration Guide

**How to build the entire platform using Claude Code sessions as agents.**

Each phase is a Claude Code session pointing to a project folder. You paste the prompt, the agent reads `platform-instructions/` and builds. You are the orchestrator.

### Report-Back Workflow (IMPORTANT)

Each phase agent auto-generates a `PHASE_X_REPORT.md` when it finishes. **Before starting the next phase:**

1. Bring the report back to the orchestrator session (this repo — MBSPlatform)
2. The orchestrator reviews for anything that changes downstream phases
3. If needed, orchestrator updates platform-instructions and re-syncs to target folders
4. Then proceed to the next phase with confidence

This prevents Phase 1 decisions from silently breaking Phase 2-5 assumptions.

---

## Before You Start (Manual — One Time)

### 1. Generate JWT_SECRET
Run this anywhere:
```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```
Save the output. This ONE secret goes into EVERY Coolify service.

### 2. Gather Your Keys
You'll need these ready (don't paste them in chat — enter them in Coolify directly):
- Google OAuth: Client ID + Client Secret (from Google Cloud Console)
- Stripe: Secret Key + Webhook Secret (from Stripe Dashboard)
- BTCPay: URL + API Key + Store ID (from your BTCPay instance)
- SendGrid: API Key (already exists in MBS and Innerlab Coolify configs)

### 3. Verify platform-instructions are copied
Each project folder should already have `platform-instructions/` with the right files. If not:
```bash
cp -r MBSPlatform/platform-instructions-for-mbs/* MBS/platform-instructions/
cp -r MBSPlatform/platform-instructions-for-innerlab/* Innerlab/platform-instructions/
cp -r MBSPlatform/platform-instructions-for-cwg/* CWG/platform-instructions/
cp -r MBSPlatform/platform-instructions-for-yogaghost/* YogaGhost/platform-instructions/
```

---

## Phase 1: MBS Platform (Layer 1)

**Folder:** `MBS/`
**What gets built:** SSO, billing, entitlements, branded login, migration scripts
**Duration estimate:** Largest phase — the foundation for everything else

### Open Claude Code in MBS/ and paste:

```
Read platform-instructions/CLAUDE.md completely before doing anything.

You are adding the MBS Platform to this existing website. The existing marketing site and form handler must keep working.

Build Phase 1 (MVP) in this order:
1. MongoDB connection to mbs_platform database + all Mongoose models (User, Entitlement, Transaction, ActiveSession, NostrChallenge, LnurlChallenge, BtcpayInvoice, Friend, Invite, FeatureFlag, ConsentAuditLog, DataRequest, PushSubscription, Announcement, EmailPreference, Promotion, Referral, ActivityLog)
2. JWT issuance + validation middleware (requireAuth, requireAdmin)
3. Google SSO auth route (POST /api/auth/google) — create/find user, issue JWT
4. Branded login page (AuthLoginPage.jsx) — reads ?brand= and ?redirect= params
5. GET /api/auth/me, POST /api/auth/logout, DELETE /api/auth/account (GDPR cascade)
6. Entitlement routes — GET /api/entitlements, GET /api/entitlements/:product, GET /api/entitlements/category/:cat
7. Product catalog config (all 22 products with freeTier flags)
8. Stripe billing — POST /api/billing/checkout, POST /api/billing/portal, POST /api/billing/webhook
9. BTCPay billing — POST /api/billing/btcpay/checkout, POST /api/billing/btcpay/webhook
10. Friends routes — GET /api/friends, POST /api/friends/invite, POST /api/friends/accept
11. Frontend pages — BillingPage, AccountPage
12. Health check — GET /api/health
13. Open redirect protection on login redirects
14. Defensive route loading (try/catch on new route imports so marketing site survives bugs)

Defer to Phase 2: Nostr auth, LNURL auth, admin panel, email preferences, promos, referrals.

Do NOT touch existing marketing pages or form handler routes. They must keep working.
```

### After Phase 1 is built — Manual steps:

1. **Set env vars in Coolify** for MBS backend:
   - `DB_NAME=mbs_platform`
   - `JWT_SECRET=(your generated secret)`
   - `JWT_EXPIRY=7d`
   - `GOOGLE_CLIENT_ID=(your ID)`
   - `GOOGLE_CLIENT_SECRET=(your secret)`
   - `STRIPE_SECRET_KEY=(your key)`
   - `STRIPE_WEBHOOK_SECRET=(your webhook secret)`
   - `BTCPAY_URL=(your BTCPay URL)`
   - `BTCPAY_API_KEY=(your key)`
   - `BTCPAY_STORE_ID=(your store ID)`
   - `CORS_ORIGINS=(all 18 domains from platform-instructions)`

2. **Set VITE_BACKEND_URL as build arg** in Coolify frontend config

3. **Deploy** — backend first, verify /api/health, then frontend

4. **Test** — Visit magicbusstudios.com/auth/login, complete Google SSO flow

### Phase 1 is DONE when:
- You can log in via Google SSO and get redirected back with a JWT
- GET /api/entitlements/cwg returns `{ success: true, hasAccess: true, reason: "free_tier" }`
- Existing marketing pages still work
- Existing form handler still works

---

## Phase 2: Inner Lab Middleware + Dashboard (Layer 2)

**Folder:** `Innerlab/`
**What gets built:** il_* collections, APIs, dashboard frontend
**Depends on:** Phase 1 complete (needs JWT_SECRET to validate tokens)

### Open Claude Code in Innerlab/ and paste:

```
Read platform-instructions/CLAUDE.md completely before doing anything.

You are adding the Inner Lab Middleware to this existing website. The existing marketing site and form handler must keep working.

Build in this order:
1. Add Mongoose to the existing Express server. Connect to inner_lab database.
2. JWT validation middleware (requireAuth) — verifies MBS Platform tokens using JWT_SECRET
3. Create Mongoose models matching the schema contracts in the platform-instructions:
   - il_consciousness_profiles, il_consciousness_snapshots, il_personal_histories
   - il_check_ins, il_user_memories, il_user_wellness_profiles
   - il_activity_feed, il_blockchain_anchors, il_sync_backups
   - il_analytics_events, il_notifications
4. Apply MongoDB JSON Schema validation on il_user_memories and il_check_ins
5. API routes (all at /api/ prefix):
   - Consciousness: GET/PUT /api/consciousness, GET /api/consciousness/snapshots
   - Check-ins: POST/GET /api/check-in, GET /api/check-in/latest, GET /api/check-in/history
   - Memories: GET/POST /api/memories, PUT /api/memories/:id/share, PUT /api/memories/:id/unshare, GET /api/memories/shared
   - Personal history: GET/PUT /api/personal-history
   - Activity: POST /api/activity, GET /api/activity/feed
   - Export: POST /api/export, GET /api/encryption/keys, POST /api/encryption/setup
6. Frontend dashboard pages (auth-gated, require JWT + Inner Lab All Access entitlement):
   - DashboardPage (/dashboard) — module launcher, activity feed, check-in widget
   - ConsciousnessPage (/consciousness) — profile view/edit
   - MemoriesPage (/memories) — list, sharing toggles
   - ActivityPage (/activity) — cross-module feed
7. Auth-gated routing — public pages (/, /modules) stay public, dashboard pages require JWT
8. Entitlement check from frontend: call MBS Platform API for category_access: innerlab

Defer: Daily briefing engine, cross-module insights (need data from multiple modules first).

Do NOT touch existing marketing pages or form handler routes.
```

### After Phase 2 — Manual steps:

1. **Set env vars in Coolify** for Innerlab backend:
   - `DB_NAME=inner_lab`
   - `JWT_SECRET=(same secret as MBS Platform)`
   - `PLATFORM_URL=https://magicbusstudios.com`
   - `CORS_ORIGINS=https://innerlab.ai,https://www.innerlab.ai`
   - Keep existing SENDGRID vars

2. **Deploy and test**

### Phase 2 is DONE when:
- You can log in via MBS Platform, get redirected to innerlab.ai/dashboard
- POST /api/check-in creates a document in il_check_ins
- GET /api/consciousness returns empty profile (no data yet)
- Marketing pages still work

---

## Phase 3: CWG Migration

**Folder:** First `MBS/` (run script), then `CWG/`
**Depends on:** Phase 1 AND Phase 2 both complete

### Step A — Run migration script from MBS/

In your existing MBS/ Claude Code session (or reopen):

```
Read platform-instructions/CLAUDE.md, specifically the Migration Scripts section.
Also read the CWG migration doc at: ../MBSPlatform/platform-instructions-for-cwg/PLATFORM_MIGRATION.md

Build and run the CWG migration script in server/scripts/migrate-cwg.js:
1. Connect to conversations_with_god database (read-only)
2. Connect to mbs_platform database (write users, platform-level data)
3. Connect to inner_lab database (write cwg_* and il_* collections)
4. Copy users with field renames (picture→avatar) and defaults for missing fields
5. Use upsert on email to avoid duplicates
6. Copy 42 cwg_* collections to inner_lab
7. Copy 7 il_* collections (consciousness, histories, check-ins, memories, analytics, notifications)
8. Copy platform-level data (sessions, challenges, consent logs, etc.)
9. Log progress and counts for verification

Run the script once. Verify a few users and collections look right.
```

### Step B — Refactor CWG

Open Claude Code in `CWG/` and paste:

```
Read platform-instructions/PLATFORM_MIGRATION.md completely before doing anything.

CWG has been migrated to the MBS Platform. The data is already copied. Now refactor the app:

1. Update backend:
   - Change DB connection from conversations_with_god to inner_lab
   - Add JWT validation middleware (Python — use PyJWT, verify with JWT_SECRET, algorithm HS256)
   - Update all collection references to use cwg_* prefix
   - Remove all standalone auth routes (login, signup, password reset, TOTP)
   - Remove all Stripe routes and BTCPay routes
   - Use req.state.user["userId"] (from JWT) instead of local user IDs
   - Add platform_user_id field mapping where needed

2. Update frontend:
   - Remove login/signup pages
   - Login button redirects to: https://innerlab.ai/auth/login?redirect=https://conversationswithgod.ai
   - Handle ?token= on redirect back — extract, store, replaceState to remove from URL
   - Remove billing/pricing pages — upgrade buttons go to MBS Platform
   - Send JWT in Authorization: Bearer header on all API calls

3. Remove standalone auth/billing code completely — don't leave dead code.
```

### After CWG migration — Manual steps:

1. **Set env vars in Coolify** for CWG backend:
   - `DB_NAME=inner_lab`
   - `JWT_SECRET=(same secret)`
   - `PLATFORM_URL=https://magicbusstudios.com`
   - `PRODUCT_SLUG=cwg`
   - Remove old STRIPE and auth env vars

2. **Deploy and test** — Log in from CWG through MBS Platform, verify conversations and data are there

---

## Phase 4: FlowState Migration

**Folder:** First `MBS/` (run script), then `YogaGhost/`
**Depends on:** Phase 1 AND Phase 2 complete
**Can run in PARALLEL with Phase 3** (no dependency between CWG and FlowState)

### Step A — Run migration script from MBS/

Same pattern as CWG but simpler (7 collections, 0 users):

```
Build and run server/scripts/migrate-flowstate.js:
1. Connect to yogaghost database (read-only)
2. Copy to mbs_platform (users with renames: picture→avatar, googleId→google_id, stripeCustomerId→stripe_customer_id)
3. Copy to inner_lab: 5 yoga_* collections + il_user_wellness_profiles
4. Copy friends/invites to mbs_platform
5. FlowState has 0 users — this is mostly setting up the collection structure.
```

### Step B — Refactor FlowState

Open Claude Code in `YogaGhost/` and paste:

```
Read platform-instructions/PLATFORM_MIGRATION.md completely before doing anything.

Refactor FlowState to use MBS Platform auth:
1. Change DB from yogaghost to inner_lab
2. Add JWT validation middleware (Node — verify with JWT_SECRET)
3. Update collections to yoga_* prefix
4. Remove standalone auth routes
5. Remove Stripe routes
6. Remove LoginPage, PricingPage
7. Login redirects to: https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com
8. Handle ?token= on redirect back
9. Sunset deviceId routes (90-day window, then remove)
```

---

## Phase 5: Standalone Products (Arcade + Studio Works)

**Depends on:** Phase 1 complete (only needs MBS Platform)
**Can run ALL 11 in PARALLEL** — they're independent of each other

### For EACH of the 11 products:

1. Copy platform-instructions:
```bash
cp -r MBSPlatform/platform-instructions-for-standalone-products/* {PROJECT}/platform-instructions/
```

2. Open Claude Code in that project folder and paste:

```
Read platform-instructions/PLATFORM_MIGRATION.md completely.

This product is: {PRODUCT_NAME}
- Slug: {SLUG}
- Domain: {DOMAIN}
- Category: {CATEGORY}

Do the SSO migration:
1. Add JWT validation middleware (requireAuth)
2. Remove standalone auth (login/signup pages and routes)
3. Login button redirects to: https://magicbusstudios.com/auth/login?redirect=https://{DOMAIN}&brand=mbs
4. Handle ?token= on redirect back — extract, store, replaceState
5. Remove Stripe/billing if any — upgrade buttons go to MBS Platform
6. Check entitlement: GET https://magicbusstudios.com/api/entitlements/{SLUG}
7. Add JWT_SECRET and PLATFORM_URL to env vars
8. Keep everything else (own database, own features, own logic)
```

### Product List (fill in the prompt above for each):

| Product | Slug | Domain | Category |
|---------|------|--------|----------|
| Broken Chain | brokenchain | brokenchain.magicbusstudios.com | arcade |
| MindHacker | mindhacker | mindhacker.magicbusstudios.com | arcade |
| Trivia Roast | triviaroast | triviaroast.magicbusstudios.com | arcade |
| Whispering House | whisperinghouse | whisperinghouse.magicbusstudios.com | arcade |
| Fake Artist | fakeartist | fakeartist.magicbusstudios.com | arcade |
| WildLens | wildlens | wildlens.magicbusstudios.com | studioworks |
| Lazy Chef | lazychef | lazy-chef.magicbusstudios.com | studioworks |
| Task Tracker | tasktracker | tasktracker.magicbusstudios.com | studioworks |
| AI Tutor | tutor | tutor.magicbusstudios.com | studioworks |
| SmartCart | smartcart | smartcart.magicbusstudios.com | studioworks |
| Movie Picker | moviepicker | moviepicker.magicbusstudios.com | studioworks |

### After each — set env vars in Coolify:
- `JWT_SECRET=(same secret)`
- `PLATFORM_URL=https://magicbusstudios.com`
- `PRODUCT_SLUG={slug}`

---

## Parallel Execution Map

```
Week 1-2:  [====== Phase 1: MBS Platform ======]
Week 2-3:           [==== Phase 2: IL Middleware ====]
Week 3:                      [Phase 3: CWG] [Phase 4: FlowState]
Week 3+:   [Phase 5: All 11 standalone products — parallel]
           [BrokenChain] [MindHacker] [TriviaRoast] ...
```

Phase 5 can start as soon as Phase 1 is deployed — it doesn't need Phase 2, 3, or 4.

---

## Coolify Env Var Checklist (All Services)

Every backend service needs:
- `JWT_SECRET` — the SAME value everywhere

MBS Platform additionally needs:
- `DB_NAME=mbs_platform`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
- `BTCPAY_URL`, `BTCPAY_API_KEY`, `BTCPAY_STORE_ID`
- `CORS_ORIGINS=(all 18 domains)`

Inner Lab Middleware additionally needs:
- `DB_NAME=inner_lab`, `PLATFORM_URL=https://magicbusstudios.com`

CWG + FlowState additionally need:
- `DB_NAME=inner_lab`, `PLATFORM_URL`, `PRODUCT_SLUG`

Standalone products additionally need:
- `PLATFORM_URL=https://magicbusstudios.com`, `PRODUCT_SLUG`
- Keep their own `DB_NAME` and `MONGO_URL`
