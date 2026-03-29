# FlowState (YogaGhost) — MBS Platform Migration Instructions

## Prerequisites — Do NOT Start Until These Are Built
1. **MBS Platform (Layer 1)** at magicbusstudios.com must be live with SSO + entitlements API
2. **Inner Lab Middleware (Layer 2)** at innerlab.ai must be live with il_* collections created
3. Both must be tested and working before ANY FlowState migration begins

## What's Happening
FlowState is being migrated to use the centralized MBS Platform for auth/billing and the Inner Lab shared database for data. This document tells the FlowState agent what to do.

## Important Technical Notes
- **FlowState uses MongoDB ObjectId for _id** — this matches the MBS Platform, so ID mapping is simpler than CWG
- **FlowState uses native MongoDB driver (not Mongoose)** — may want to switch to Mongoose for consistency with platform, or keep native driver. Agent decides.
- **JWT_SECRET must match the MBS Platform** — use the same secret so JWTs issued by the platform validate in FlowState
- **Migration scripts run FROM the MBS/ project** (`MBS/server/scripts/`), not from FlowState. The FlowState agent should NOT build its own migration script — just prepare the backend for the new database structure.

## Current State
- FlowState has its own auth (Google SSO + email/password with scrypt hashing)
- FlowState has its own Stripe integration (not configured — missing env vars)
- FlowState database: `yogaghost` with 7 collections
- 0 real users (no production data to protect)
- Uses native MongoDB driver (not Mongoose)
- Zustand local storage + cloud sync pattern

## Target State
- Auth handled by MBS Platform at magicbusstudios.com (Google SSO + Email/Password + Nostr + LNURL + 2FA/TOTP)
- Login redirects to Inner Lab login page: `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com`
- Billing handled by MBS Platform (Stripe + BTCPay)
- User identity stored in `mbs_platform` database
- FlowState product data stored in `inner_lab` database with `yoga_` prefix
- Shared Inner Lab data stored in `inner_lab` database with `il_` prefix
- Old `yogaghost` database stays untouched as backup

## Migration Steps

### Step 1: Update Auth Flow
- Remove FlowState's standalone auth (login, signup, password reset, scrypt hashing)
- Remove `/auth/signup`, `/auth/login`, `/auth/logout` routes
- Add JWT verification middleware that validates tokens from MBS Platform
- Login button redirects to: `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com`
  - FlowState is an Inner Lab module, so login goes through Inner Lab login page (which calls MBS Platform auth APIs — supports Google SSO, email/password, Nostr, LNURL, 2FA)
- After login, user is redirected back with `?token={JWT}`
- Store JWT, send in Authorization header

### Step 2: Remove Billing
- Remove FlowState's Stripe routes and webhook handler
- Remove Stripe env vars (STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, STRIPE_PRICE_*)
- Upgrade buttons redirect to MBS Platform billing page
- Check access via: `GET https://api.magicbusstudios.com/api/entitlements/flowstate`

### Step 3: Migrate Data
Since FlowState has 0 real users, this is less critical than CWG. But follow the same pattern:

**User identity fields → `mbs_platform.users`:**
- email, name, picture → **rename to `avatar`** (platform model uses `avatar`)
- googleId → **rename to `google_id`** (platform uses snake_case)
- stripeCustomerId → **rename to `stripe_customer_id`** (platform uses snake_case)
- emailPreferences → **normalize to `consent_preferences`** structure:
  - `{ marketing_emails: emailPreferences.marketing, product_updates: true, billing_alerts: true, promotional_offers: true }`
  - FlowState's `streakReminders` and `weeklyDigest` are product-level preferences, NOT platform-level. Keep them in `yoga_user_profiles`.

**Fields NOT in FlowState but required by platform (set defaults):**
- nostr_npub: `null` (FlowState has no Nostr support)
- lnurl_linking_key: `null` (FlowState has no LNURL support)
- auth_methods: `["google"]` (FlowState only supports Google SSO currently)
- preferred_language: `"en"`
- preferences: `{}`
- is_admin: `false`
- referral_code: `null`
- referred_by: `null`
- referral_count: `0`
- created_at: use FlowState's `joinedDate` from user profile, or migration timestamp
- updated_at: migration timestamp
- last_login: `null`

**Do NOT migrate to platform:**
- subscription object (platform creates its own Entitlement records via Stripe)
- deviceId (deprecated — not a platform concept)

**Product data → `inner_lab` with `yoga_` prefix:**
- `yoga_user_profiles` — onboarded, settings, favorites, joinedDate, customFlows, dailyStreak, lastSessionDate, streakSaverUsed, activeCategory, breathworkFavorites, breathworkPersonalBests, breathworkSessions (embedded), breathworkAchievements (embedded), activePrograms (embedded), completedPrograms, meditationSessions (embedded), meditationFavorites, flowSessions (embedded)
- `yoga_sessions` (from `sessions` collection) — yoga practice sessions with TTL
- `yoga_achievements` (from `achievements` collection) — earned badges
- `yoga_community_flows` (from `communityFlows`) — published user flows
- `yoga_activity` (from `activity`) — social activity feed, add `product: "flowstate"` field

**Shared data → `inner_lab` with `il_` prefix:**
- `il_user_wellness_profiles` — extract healthConditions + injuries from users doc
- Goals → partially shared (currently FlowState-specific strings like 'balance', 'flexibility')

**Platform-level → `mbs_platform` database:**
- `friends` collection (from `friends`) — add `source_product: "flowstate"` field
- `invites` collection (from `invites`) — add `product: "flowstate"` field

### Step 4: Update FlowState Backend
- Change database connection from `yogaghost` to `inner_lab`
- Update all collection references to use `yoga_` prefix
- Read shared data from `il_*` collections (wellness profiles, check-ins, memories)
- Use `platformUserId` (from JWT) instead of local `_id`
- Remove auth routes, Stripe routes
- Remove `deviceId`-based sync routes (sunset after 90 days)
- Keep cloud sync but use platform user ID instead of local ID

### Step 5: Update FlowState Frontend
- Remove LoginPage — redirect to MBS Platform
- Remove PricingPage — redirect to MBS Platform
- Update `useAuthStore` to use MBS Platform JWT
- Remove `mbflow-device-id` localStorage key (deprecated)
- Update sync endpoints to use authenticated routes only

### Step 6: Handle Embedded Arrays
FlowState stores breathwork/meditation/flow data as embedded arrays in the `users` doc. During migration, decide:
- **Option A**: Keep embedded (simpler, current pattern works)
- **Option B**: Extract to separate `yoga_breathwork_sessions`, `yoga_meditation_sessions`, `yoga_flow_sessions` collections (better for scale)
- **Recommendation**: Keep embedded for now (0 users, no scale concern). Extract later if needed.

## deviceId Legacy Pattern
- `deviceId` was used for pre-auth device-based sync
- All routes now require JWT auth
- Keep deviceId-based routes for 90-day migration window
- After 90 days, remove deviceId routes and field entirely
- Do NOT create new deviceId entries

## User Dedup / Merge Logic
FlowState has 0 real users, so collision risk is zero. The migration script should still use upsert on email as a safety pattern — if a user somehow exists (from CWG migration), merge without overwriting existing values.

## What NOT to Do
- Do NOT delete the `yogaghost` database — it's the backup
- Do NOT modify user data during migration — copy only
- Do NOT run migration until MBS Platform and Inner Lab Middleware are built and tested
- Do NOT keep standalone auth/billing code after migration

---

## Completion Report (REQUIRED)

When you finish the migration and refactor, generate a file called `PHASE_4_REPORT.md` in the project root. The orchestrator will fetch it from here. The report must contain:

1. **What was built/changed** — Every file created, modified, or deleted, grouped by backend/frontend
2. **What changed from the plan** — Any deviations from this document. Why?
3. **Migration results** — How many users migrated, collections copied, any errors or skipped records
4. **Collection mapping** — Old collection name → new collection name (yoga_* and il_*) as actually created
5. **Field renames** — Every field that was renamed during migration (old → new)
6. **Env vars required** — Complete list for the refactored FlowState app
7. **Code removed** — List of removed auth/billing routes, pages, and files
8. **JWT integration** — How JWT middleware was implemented (library, header format, fields extracted)
9. **deviceId sunset** — How deviceId legacy routes were handled (kept for 90 days? removed?)
10. **Assumptions made** — Anything you had to decide that wasn't explicitly in the spec
11. **Known gaps** — Anything deferred or issues discovered
12. **Testing steps** — How to verify the migration worked and the refactored app functions correctly

This report is critical — the orchestrator session (MBSPlatform repo) uses it to track the migration.

---

## Phase 3B Learnings (Added by Orchestrator — 2026-03-27)

These are real-world lessons from the CWG migration that apply to FlowState:

### API URL: Use `api.magicbusstudios.com` NOT `magicbusstudios.com`
The MBS Platform API lives at `https://api.magicbusstudios.com` (the backend container). `https://magicbusstudios.com` is the frontend (nginx). Calling the frontend URL for API requests causes CORS errors. All entitlement checks, billing redirects, and any backend API calls must use `api.magicbusstudios.com`.

### After Deleting Auth Files — Grep for ALL Imports
When you delete auth routes and services (e.g., stripe_integration.js, auth_routes.js), other files may still import functions from them. A runtime crash will occur. **Before deploying, run:** `grep -r "require.*auth\|require.*stripe\|require.*btcpay\|import.*auth\|import.*stripe" server/` and remove all stale imports.

### Legacy Auth Routes — Redirect to Platform Login, Not Homepage
When removing `/login`, `/signup`, `/forgot-password` pages, DO NOT redirect those routes to `/` (homepage). Instead, redirect them to the platform login URL: `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com`. Users with bookmarks or cached links will otherwise land on a confusing homepage.

### BrowserRouter Ordering
Any React component that uses `useLocation()`, `useNavigate()`, or other React Router hooks MUST be nested INSIDE `<BrowserRouter>`, not wrapping it. If you have a wrapper component (e.g., SmoothScroll) that uses these hooks, it must be a child of BrowserRouter.

### User ID Resolution — CRITICAL for Migrated Modules
The JWT `userId` field contains the MBS Platform ObjectId (e.g., `69c53401fe8f1763b9046ae5`). But if data was migrated from the old database, documents may use the old module-specific `_id`. FlowState uses MongoDB ObjectId natively so this is LESS of a problem than CWG (which used UUIDs), but still:
- **If FlowState has 0 real users**: auto-provision new profiles with the platform user ID. No resolution layer needed.
- **If any users exist at migration time**: implement email-based resolution like CWG did — look up old profile by email from JWT, add `platform_user_id` field, cache the mapping in memory.
- **All new user_id fields in yoga_* collections must store the platform ObjectId string**, not a local ID.

### Entitlements Check — MUST Be Wired
CWG missed this during Phase 3B. FlowState MUST call `GET https://api.magicbusstudios.com/api/entitlements/flowstate` (with Bearer token) and enforce the result. Without it, there's no free/premium distinction. Cache the result for 5 minutes.

### Cross-Origin Login Redirect Already Works
Inner Lab's login page (commit `efebe36`) already supports cross-origin redirects to `*.magicbusstudios.com` domains. When FlowState redirects to `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com`, Inner Lab will:
1. Accept the redirect URL (it's in the trusted domain whitelist)
2. After login, redirect back with `?token={JWT}` appended
3. FlowState frontend extracts the token and stores it in localStorage
No changes needed on Inner Lab's side for this to work with FlowState.

### Coolify Env Vars — Separate Lines
When pasting env vars in Coolify, ensure each var is on its own line. A known CWG issue: `JWT_SECRET=xxxLOG_LEVEL=INFO` was pasted as one line, making JWT validation fail silently.

---

## Phase 1 Learnings (Added by Orchestrator — 2026-03-26)

These are real-world implementation details from the MBS Platform build that affect this migration:

### Env Var Name
- The MBS Platform uses **`MONGO_URL`** (not `MONGODB_URI`). Use `MONGO_URL` for consistency.

### JWT Details (as actually implemented)
- Header: `Authorization: Bearer <token>`
- Payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- `userId` is a **string** (ObjectId.toString())
- Node.js: use `jsonwebtoken` package, algorithm `HS256`, verify with shared `JWT_SECRET`

### Frontend Token Storage
- After login redirect, JWT arrives as `?token=<JWT>` in the URL
- FlowState frontend must: extract token, store as `localStorage.setItem("mbs_token", token)`, store user as `localStorage.setItem("mbs_user", JSON.stringify(user))`, then `history.replaceState` to remove from URL
- All API calls use header: `Authorization: Bearer ${localStorage.getItem("mbs_token")}`

### Entitlement Check
- `GET https://api.magicbusstudios.com/api/entitlements/flowstate` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }` — reason `free_tier` means free access
- Cache response for 5 minutes in-memory

### Category Values
- Categories are lowercase no-separator: `innerlab` (not `inner_lab`)

### GDPR
- MBS Platform cascade delete reaches into `inner_lab` database. All yoga_* and il_* collections must use `user_id` field consistently.

### Additional Phase 1 Learnings (updated after live testing)
- **email can be null** — Nostr/LNURL users have no email. Guard with `if (user.email)` everywhere.
- **Entitlement reason values**: `"product_pass"`, `"category_access"`, `"mbs_all_access"`, `"free_tier"`, `"no_subscription"` — confirmed from live code.
- **Rate limiting on platform API**: 100 req/15min. Cache entitlement checks for 5 min.
- **Billing page is live** at `magicbusstudios.com/billing` with CWG pricing and Lightning option. Redirect upgrade buttons there.
- **BTCPay status**: Currently 403 due to API key permissions. Will be fixed separately — does not block FlowState migration.
- **First platform user**: Abhinav Gupta (`1984.abhinav@gmail.com`), ID: `69c53401fe8f1763b9046ae5`.
