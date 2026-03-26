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
- Auth handled by MBS Platform at magicbusstudios.com (Google SSO + Nostr + LNURL)
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
- Login button redirects to: `magicbusstudios.com/auth/login?redirect=yoga.magicbusstudios.com&brand=innerlab`
- After login, MBS Platform redirects back with `?token={JWT}`
- Store JWT, send in Authorization header

### Step 2: Remove Billing
- Remove FlowState's Stripe routes and webhook handler
- Remove Stripe env vars (STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, STRIPE_PRICE_*)
- Upgrade buttons redirect to MBS Platform billing page
- Check access via: `GET https://magicbusstudios.com/api/entitlements/flowstate`

### Step 3: Migrate Data
Since FlowState has 0 real users, this is less critical than CWG. But follow the same pattern:

**User identity fields → `mbs_platform.users`:**
- email, name, picture, googleId
- stripeCustomerId, subscription object
- emailPreferences (streakReminders, weeklyDigest, marketing)
- deviceId (deprecated — 90-day sunset)

**Product data → `inner_lab` with `yoga_` prefix:**
- `yoga_user_profiles` — onboarded, settings, favorites, joinedDate, customFlows, dailyStreak, lastSessionDate, streakSaverUsed, activeCategory, breathworkFavorites, breathworkPersonalBests, breathworkSessions (embedded), breathworkAchievements (embedded), activePrograms (embedded), completedPrograms, meditationSessions (embedded), meditationFavorites, flowSessions (embedded)
- `yoga_sessions` (from `sessions` collection) — yoga practice sessions with TTL
- `yoga_achievements` (from `achievements` collection) — earned badges
- `yoga_community_flows` (from `communityFlows`) — published user flows
- `yoga_activity` (from `activity`) — social activity feed, add `product: "flowstate"` field

**Shared data → `inner_lab` with `il_` prefix:**
- `il_user_wellness_profiles` — extract healthConditions + injuries from users doc
- Goals → partially shared (currently FlowState-specific strings like 'balance', 'flexibility')

**Platform-level → `mbs_platform`:**
- `mbs_friends` (from `friends`) — add source_product field
- `mbs_invites` (from `invites`) — add product field

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

## What NOT to Do
- Do NOT delete the `yogaghost` database — it's the backup
- Do NOT modify user data during migration — copy only
- Do NOT run migration until MBS Platform and Inner Lab Middleware are built and tested
- Do NOT keep standalone auth/billing code after migration
