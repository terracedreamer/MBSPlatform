# PHASE 2 REPORT — Inner Lab Middleware + Dashboard

**Date:** 2026-03-26
**Project:** Innerlab (innerlab.ai)
**Branch:** main
**Phase:** Layer 2 — Inner Lab Middleware + Dashboard
**Last Updated:** 2026-03-26 (post-audit fixes)

---

## 0. Post-Build Audit Fixes (2026-03-26)

After the initial build, a full audit was run against the platform spec. The following bugs were found and fixed:

| # | Issue | Severity | Fix |
|---|-------|----------|-----|
| 1 | **Encryption route double-nesting** — `exportRouter` was mounted at both `/api/export` and `/api/encryption`, causing `/api/encryption/encryption/keys` instead of `/api/encryption/keys` | Bug (404s) | Split encryption endpoints into `server/routes/encryption.js`, mounted separately |
| 2 | **Unused `Navigate` import** in `ProtectedRoute.jsx` — would break CI build with `CI=true` (ESLint treats unused imports as errors) | Build breaker | Removed unused import |
| 3 | **`?refresh=true` cache bust race condition** — first `useEffect` stripped `refresh` from URL before second `useEffect` could read it, so entitlement cache was never busted | Bug (billing upgrades wouldn't reflect) | Moved refresh param reading and cache bust into first `useEffect`, before URL cleanup |
| 4 | **Entitlement loading flash** — `ProtectedRoute` showed `UpsellPage` briefly while entitlement fetch was in-flight (entitlement was `null`, not `false`) | UX bug | Added check for `entitlement === null` to show spinner instead of upsell |
| 5 | **Module URL double-protocol** — `DashboardPage` template used `https://${mod.url}` but `mod.url` already includes `https://`, producing `https://https://...` | Bug (broken links) | Added protocol check before prepending `https://` |
| 6 | **Missing canonical tags on 4 dashboard pages** — `DashboardPage`, `ConsciousnessPage`, `MemoriesPage`, `ActivityPage` were missing `path` prop on `<SEO>`, causing canonical to be `https://innerlab.aiundefined` | SEO bug | Added correct `path` prop to all 4 pages |

### Files Created
| File | Purpose |
|------|---------|
| `server/routes/encryption.js` | Dedicated encryption router (split from export.js) |

### Files Modified
| File | Change |
|------|--------|
| `server/routes/export.js` | Removed encryption endpoints (moved to encryption.js) |
| `server/index.js` | Import + mount `encryptionRouter` at `/api/encryption` |
| `src/components/ProtectedRoute.jsx` | Removed unused `Navigate` import; added entitlement loading state |
| `src/contexts/AuthContext.jsx` | Fixed `?refresh=true` cache bust race condition |
| `src/pages/DashboardPage.jsx` | Fixed module URL protocol; added SEO `path` prop |
| `src/pages/ConsciousnessPage.jsx` | Added SEO `path="/consciousness"` |
| `src/pages/MemoriesPage.jsx` | Added SEO `path="/memories"` |
| `src/pages/ActivityPage.jsx` | Added SEO `path="/activity"` |

### SEO Canonical Status
- **No hardcoded canonical in `index.html`** ✅ (not vulnerable to the CWG bug)
- **All 11 pages now have per-page canonical tags** via `<SEO path="..." />` → `https://innerlab.ai/exact-path`
- Marketing pages already had `path` props; dashboard pages were missing them (now fixed)

---

## 1. What Was Built

### Backend (server/)

| File | Purpose |
|------|---------|
| `server/package.json` | Express server dependencies (mongoose, jsonwebtoken, cors, helmet, etc.) |
| `server/index.js` | Express entry point — mounts all routes, connects to MongoDB, rate limiting |
| `server/config/logger.js` | Winston logger (never console.log) |
| `server/config/database.js` | MongoDB/Mongoose connection using `MONGO_URL` + `DB_NAME` |
| `server/config/modules.js` | Module catalog (11 modules, configurable, not hardcoded) |
| `server/middleware/requireAuth.js` | JWT validation middleware (verifies MBS Platform tokens) |
| `server/models/CheckIn.js` | `il_check_ins` — mood/energy/stress/intention |
| `server/models/ConsciousnessProfile.js` | `il_consciousness_profiles` — archetype/orientation |
| `server/models/ConsciousnessSnapshot.js` | `il_consciousness_snapshots` — historical changes |
| `server/models/PersonalHistory.js` | `il_personal_histories` — user life story |
| `server/models/UserMemory.js` | `il_user_memories` — cross-module memories with sharing |
| `server/models/WellnessProfile.js` | `il_user_wellness_profiles` — health/injuries/goals |
| `server/models/ActivityFeed.js` | `il_activity_feed` — cross-module events |
| `server/models/BlockchainAnchor.js` | `il_blockchain_anchors` — OpenTimestamps proofs |
| `server/models/SyncBackup.js` | `il_sync_backups` — local-first sync |
| `server/models/AnalyticsEvent.js` | `il_analytics_events` — TTL 90 days |
| `server/models/Notification.js` | `il_notifications` — unified notifications |
| `server/routes/checkins.js` | POST/GET check-in endpoints |
| `server/routes/consciousness.js` | GET/PUT consciousness profile + snapshots |
| `server/routes/personalHistory.js` | GET/PUT personal history |
| `server/routes/memories.js` | CRUD memories + share/unshare |
| `server/routes/activity.js` | POST/GET activity feed |
| `server/routes/export.js` | Data export endpoint |
| `server/routes/encryption.js` | Encryption key metadata + setup (placeholder) |
| `server/routes/forms.js` | Contact/subscribe/waitlist via SendGrid |
| `server/scripts/setupValidation.js` | MongoDB JSON Schema validation setup |

### Frontend (src/)

| File | Purpose |
|------|---------|
| `src/contexts/AuthContext.jsx` | Auth provider — JWT from localStorage, entitlement check |
| `src/components/ProtectedRoute.jsx` | Auth gate — redirects to MBS login or shows upsell |
| `src/utils/dashboardApi.js` | Authenticated API client for all dashboard endpoints |
| `src/pages/DashboardPage.jsx` | Main dashboard — check-in widget, module launcher, activity |
| `src/pages/ConsciousnessPage.jsx` | Consciousness profile viewer/editor + snapshots |
| `src/pages/MemoriesPage.jsx` | Memory manager with sharing toggles + create form |
| `src/pages/ActivityPage.jsx` | Full cross-module activity feed with filters |

### Modified Files

| File | Change |
|------|--------|
| `src/App.jsx` | Added 4 auth-gated routes (/dashboard, /consciousness, /memories, /activity) |
| `src/main.jsx` | Wrapped app in AuthProvider |
| `src/components/Layout.jsx` | Added Dashboard nav link + user avatar/logout for authenticated users |
| `.env.example` | Added all backend env vars |

### Infrastructure

| File | Purpose |
|------|---------|
| `Dockerfile.server` | Backend Docker container (node:22-alpine) |
| `.env.example` | Complete env var documentation |

---

## 2. What Changed From the Plan

| Spec | Actual | Why |
|------|--------|-----|
| `server/` already exists | Created from scratch | No server/ directory existed — it was never built |
| Forms use shared MBS backend | Added forms.js to this server too | Platform instructions expect forms in this server |
| Daily Briefing endpoint | Placeholder in dashboard UI only | Spec says "deferred until enough data" |
| Cross-Module Insights endpoint | Not built | Spec says "built LATER when 2-3 modules have data" |
| `GET /api/today` | Not built | Deferred per spec |
| `GET /api/insights` | Not built | Deferred per spec |
| Encryption endpoints | Placeholder responses | Client-side encryption requires frontend crypto implementation |
| `il_consciousness_snapshots` schema | Added `snapshot_reason` field | Not in original spec but needed to track why snapshot was taken |
| `il_blockchain_anchors` schema | Designed from scratch | Only collection name was in spec, no schema |
| `il_sync_backups` schema | Designed from scratch | Only collection name was in spec, no schema |
| `il_analytics_events` schema | Designed from scratch | Only TTL mentioned in spec |
| `il_notifications` schema | Designed from scratch | Only collection name was in spec |

---

## 3. Env Vars Required

| Variable | Required | Example | Notes |
|----------|----------|---------|-------|
| `MONGO_URL` | Yes | `mongodb://localhost:27017` | Same as all MBS apps |
| `DB_NAME` | Yes | `inner_lab` | Shared with all IL modules |
| `JWT_SECRET` | Yes | `your-shared-secret` | Must match MBS Platform |
| `PORT` | No | `3001` | Defaults to 3001 |
| `CORS_ORIGINS` | No | `https://innerlab.ai` | Comma-separated |
| `PLATFORM_URL` | No | `https://magicbusstudios.com` | For entitlement checks |
| `SENDGRID_API_KEY` | No | `SG.xxx` | For form emails |
| `FROM_EMAIL` | No | `noreply@magicbusstudios.com` | SendGrid sender |
| `TO_EMAIL` | No | `support@magicbusstudios.com` | Form recipient |
| `VITE_API_URL` | Build-time | `https://api.innerlab.ai/api` | Frontend API base URL |
| `VITE_FORM_SOURCE` | Build-time | `innerlab` | Form source tag |
| `VITE_PLATFORM_URL` | Build-time | `https://magicbusstudios.com` | Platform URL for auth |

---

## 4. Database Collections

| Mongoose Model | MongoDB Collection | Indexes |
|---|---|---|
| CheckIn | `il_check_ins` | `user_id`, `(user_id, created_at)` |
| ConsciousnessProfile | `il_consciousness_profiles` | `user_id` (unique) |
| ConsciousnessSnapshot | `il_consciousness_snapshots` | `(user_id, created_at)` |
| PersonalHistory | `il_personal_histories` | `user_id` (unique) |
| UserMemory | `il_user_memories` | `(user_id, source_module)`, `(user_id, shared)`, `expires_at` (TTL) |
| WellnessProfile | `il_user_wellness_profiles` | `user_id` (unique) |
| ActivityFeed | `il_activity_feed` | `(user_id, created_at)` |
| BlockchainAnchor | `il_blockchain_anchors` | `(user_id, collection_name)` |
| SyncBackup | `il_sync_backups` | `(user_id, device_id)` (unique) |
| AnalyticsEvent | `il_analytics_events` | `created_at` (TTL 90 days), `(user_id, event_type)` |
| Notification | `il_notifications` | `(user_id, read, created_at)` |

---

## 5. API Routes

| Method | Path | Auth? | Description |
|--------|------|-------|-------------|
| GET | `/health` | No | Health check |
| POST | `/api/contact` | No | Contact form (SendGrid) |
| POST | `/api/subscribe` | No | Newsletter signup (SendGrid) |
| POST | `/api/waitlist` | No | Waitlist signup (SendGrid) |
| POST | `/api/check-in` | Yes | Create check-in (mood/energy/stress/intention) |
| GET | `/api/check-in/latest` | Yes | Most recent check-in |
| GET | `/api/check-in/history` | Yes | Check-in history (query: from, to, limit) |
| GET | `/api/consciousness` | Yes | Get consciousness profile |
| PUT | `/api/consciousness` | Yes | Update consciousness profile (creates snapshot) |
| GET | `/api/consciousness/snapshots` | Yes | Historical snapshots |
| GET | `/api/personal-history` | Yes | Get personal history |
| PUT | `/api/personal-history` | Yes | Update personal history (upsert) |
| GET | `/api/memories` | Yes | Get memories (query: source_module, memory_type, limit) |
| GET | `/api/memories/shared` | Yes | Get all shared memories |
| POST | `/api/memories` | Yes | Create a memory |
| PUT | `/api/memories/:id/share` | Yes | Share a memory across Inner Lab |
| PUT | `/api/memories/:id/unshare` | Yes | Revoke memory sharing |
| POST | `/api/activity` | Yes | Log activity event |
| GET | `/api/activity/feed` | Yes | Activity feed (query: limit, source_module, before) |
| POST | `/api/export` | Yes | Generate data export with SHA-256 integrity hash |
| GET | `/api/encryption/keys` | Yes | Encryption key metadata (placeholder) |
| POST | `/api/encryption/setup` | Yes | Initialize encryption (placeholder) |

---

## 6. Frontend Routes

| Path | Component | Auth-Gated? | Description |
|------|-----------|-------------|-------------|
| `/` | Home | No | Marketing landing page |
| `/inner-lab` | → Redirects to `/` | No | Legacy redirect |
| `/modules` | Modules | No | Module catalog |
| `/waitlist` | Waitlist | No | Waitlist form |
| `/subscribe` | Subscribe | No | Newsletter signup |
| `/about` | About | No | Studio info |
| `/contact` | Contact | No | Contact form |
| `/dashboard` | DashboardPage | **Yes** | Main dashboard |
| `/consciousness` | ConsciousnessPage | **Yes** | Consciousness profile |
| `/memories` | MemoriesPage | **Yes** | Memory manager |
| `/activity` | ActivityPage | **Yes** | Activity feed |
| `*` | NotFound | No | 404 page |

---

## 7. il_* Schema Contracts (As Implemented)

### il_check_ins
```
user_id: String (required, indexed)
source_module: String (required, default: "dashboard")
mood: Number (required, 1-10)
energy: Number (required, 1-10)
stress: Number (required, 1-10)
intention: String (optional, max 500)
notes: String (optional, max 2000)
created_at: Date (default: now)
```

### il_consciousness_profiles
```
user_id: String (required, unique)
archetype: String (default: "")
orientation: String (default: "")
assessment_data: Mixed/Object (default: {})
source_module: String (required, default: "dashboard")
created_at: Date (default: now)
updated_at: Date (auto-updated)
```

### il_consciousness_snapshots
```
user_id: String (required, indexed)
archetype: String (required)
orientation: String (default: "")
assessment_data: Mixed/Object (default: {})
source_module: String (required)
snapshot_reason: String (default: "profile_update")
created_at: Date (default: now)
```

### il_personal_histories
```
user_id: String (required, unique)
content: Mixed/Object (required, default: {})
source_module: String (required, default: "dashboard")
created_at: Date (default: now)
updated_at: Date (auto-updated)
```

### il_user_memories
```
user_id: String (required, indexed)
source_module: String (required)
memory_type: String (required, enum: "fact"|"preference"|"emotional_state"|"insight")
content: String (required, max 5000)
confidence: Number (0-1, default: 0.8)
shared: Boolean (required, default: false)
shared_at: Date (null if not shared)
created_at: Date (default: now)
updated_at: Date (auto-updated)
expires_at: Date (optional, TTL index)
```

### il_user_wellness_profiles
```
user_id: String (required, unique)
health_conditions: [String] (default: [])
injuries: [String] (default: [])
goals: [String] (default: [])
source_module: String (required, default: "dashboard")
created_at: Date (default: now)
updated_at: Date (auto-updated)
```

### il_activity_feed
```
user_id: String (required, indexed)
source_module: String (required)
action: String (required, max 200)
title: String (required, max 500)
description: String (optional, max 2000)
metadata: Mixed/Object (default: {})
created_at: Date (default: now)
```

### il_blockchain_anchors
```
user_id: String (required, indexed)
collection_name: String (required)
document_id: String (required)
hash: String (required)
timestamp_proof: Mixed (default: null)
status: String (enum: "pending"|"anchored"|"failed", default: "pending")
created_at: Date (default: now)
```

### il_sync_backups
```
user_id: String (required)
device_id: String (required)
last_sync: Date (default: now)
data_hash: String (default: "")
status: String (enum: "synced"|"pending"|"conflict", default: "pending")
created_at: Date (default: now)
updated_at: Date (default: now)
```
**Unique index on (user_id, device_id)**

### il_analytics_events
```
user_id: String (required, indexed)
event_type: String (required)
source_module: String (default: "dashboard")
metadata: Mixed/Object (default: {})
created_at: Date (default: now)
```
**TTL index: auto-deletes after 90 days**

### il_notifications
```
user_id: String (required, indexed)
type: String (required)
title: String (required)
message: String (default: "")
source_module: String (default: "system")
read: Boolean (default: false)
read_at: Date (default: null)
action_url: String (default: null)
created_at: Date (default: now)
```

---

## 8. JWT Validation

- **Header format:** `Authorization: Bearer <token>`
- **Algorithm:** Default jsonwebtoken verification (HS256 assumed, matching MBS Platform)
- **Secret:** `JWT_SECRET` env var (must match MBS Platform's secret)
- **Payload fields extracted:** `userId`, `email`, `name`, `avatar`, `isAdmin`, `iat`, `exp`
- **userId mapping:** JWT uses `userId` (camelCase) → all il_* documents use `user_id: req.user.userId` (snake_case String)
- **Expiry handling:** Returns `401` with "Token expired" message
- **Missing secret:** Returns `500` with "Server configuration error"
- **Invalid token:** Returns `401` with "Invalid token"

---

## 9. Assumptions Made

1. **No server/ existed** — the spec referenced "existing Express server" but none was present. Created from scratch.
2. **Forms stay external AND local** — forms are handled in this server's routes AND the frontend can still point to api.magicbusstudios.com via `VITE_API_URL`. Both paths work.
3. **MongoDB JSON Schema validation uses `validationLevel: "moderate"` and `validationAction: "warn"`** — safety net, not gatekeeper. Mongoose schemas are the primary defense.
4. **No WellnessProfile API routes** — WellnessProfile model exists but no HTTP endpoints were specified. FlowState/BreathArc will write directly via shared DB.
5. **Consciousness profile update auto-creates a snapshot** — the spec didn't specify when snapshots are taken, so we snapshot on every PUT to /api/consciousness.
6. **DashboardPage module launcher reads from `src/data/modules.js`** — uses the existing frontend module catalog, not a backend endpoint.
7. **Entitlement check calls MBS Platform at `PLATFORM_URL/api/entitlements/category/innerlab`** — cached for 5 minutes in localStorage.
8. **Token extraction from URL** — `?token=<JWT>` is extracted on page load and stored in `localStorage("mbs_token")`, matching MBS Platform convention.
9. **`?refresh=true`** URL param busts entitlement cache, matching MBS Platform convention.
10. **Default port 3001** — avoids conflict with MBS Platform (3002) and common dev ports.

---

## 10. Known Gaps

1. **Daily Briefing (GET /api/today)** — Deferred. Placeholder shown in dashboard UI.
2. **Cross-Module Insights (GET /api/insights)** — Deferred. Needs 2-3 modules with data.
3. **Client-side encryption** — Encryption endpoints return placeholder responses. Full AES-256-GCM implementation requires frontend crypto work.
4. **Blockchain anchoring** — Model exists but no API routes or OpenTimestamps integration built.
5. **Sync backup endpoints** — Model exists but no API routes built. Waiting for local-first architecture decisions.
6. **Notification endpoints** — Model exists but no API routes built. Needs notification delivery mechanism (push/email/in-app).
7. **Analytics event endpoints** — Model exists but no API routes built. Could be added when module analytics are needed.
8. **No real Stripe/billing** — Entitlement check assumes MBS Platform billing is functional. Currently returns mock data since billing page has placeholder pricing.
9. **Wellness profile routes** — No HTTP endpoints. Modules write directly to il_user_wellness_profiles via shared DB.

---

## 11. Gotchas for Downstream Phases

### For Phase 3 (CWG Migration)
- **user_id is always a String** (not ObjectId). When CWG writes to il_* collections, use `user.userId.toString()` or equivalent.
- **il_user_memories requires `shared: Boolean`** — CWG should set `shared: false` by default. Users opt in via the dashboard.
- **memory_type enum is strict**: `"fact"`, `"preference"`, `"emotional_state"`, `"insight"`. CWG must map its memory categories to these types.
- **source_module must be `"cwg"`** — used for filtering in the dashboard.
- **il_check_ins mood/energy/stress are 1-10** — if CWG uses a different scale, normalize before writing.
- **il_consciousness_profiles has a unique index on user_id** — use `findOneAndUpdate` with `upsert: true`, not `create()`.
- **Personal history `content` field is a flexible Mixed/Object** — CWG can store its dual formats (narrative + structured) here.

### For Phase 4 (FlowState Migration)
- **il_user_wellness_profiles** — FlowState should write health conditions, injuries, and goals here. Unique on user_id, use upsert.
- **il_activity_feed** — use `source_module: "yoga"` or `"flowstate"` (decide on one and be consistent).
- **il_check_ins** — FlowState can write check-ins with `source_module: "yoga"`.
- All il_* date fields use JavaScript `Date` objects (Mongoose handles conversion from Motor's datetime).

### General
- All il_* collections are in the `inner_lab` database (DB_NAME env var).
- MongoDB-level JSON Schema validation is set to `warn` mode — documents that fail validation are still inserted but logged.
- `updated_at` fields are auto-set via Mongoose pre-save/pre-findOneAndUpdate hooks.
- `expires_at` on il_user_memories uses a MongoDB TTL index — documents with a past `expires_at` are auto-deleted by MongoDB.

---

## 12. Testing Commands

### Start Server
```bash
cd server && npm start
# or for development with auto-reload:
cd server && npm run dev
```

### Health Check
```bash
curl http://localhost:3001/health
# Expected: {"status":"ok","service":"innerlab-middleware"}
```

### Create Check-In (requires JWT)
```bash
curl -X POST http://localhost:3001/api/check-in \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"mood":7,"energy":6,"stress":4,"intention":"Focus on clarity"}'
# Expected: {"success":true,"checkIn":{...}}
```

### Get Latest Check-In
```bash
curl http://localhost:3001/api/check-in/latest \
  -H "Authorization: Bearer <JWT_TOKEN>"
# Expected: {"success":true,"checkIn":{...}}
```

### Get Consciousness Profile
```bash
curl http://localhost:3001/api/consciousness \
  -H "Authorization: Bearer <JWT_TOKEN>"
# Expected: {"success":true,"profile":null} (empty initially)
```

### Update Consciousness Profile
```bash
curl -X PUT http://localhost:3001/api/consciousness \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"archetype":"The Seeker","orientation":"Contemplative"}'
# Expected: {"success":true,"profile":{...}}
```

### Create Memory
```bash
curl -X POST http://localhost:3001/api/memories \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"source_module":"dashboard","memory_type":"insight","content":"User prefers morning meditation","shared":false}'
# Expected: {"success":true,"memory":{...}}
```

### Share Memory
```bash
curl -X PUT http://localhost:3001/api/memories/<MEMORY_ID>/share \
  -H "Authorization: Bearer <JWT_TOKEN>"
# Expected: {"success":true,"memory":{..., "shared":true}}
```

### Log Activity
```bash
curl -X POST http://localhost:3001/api/activity \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"source_module":"dashboard","action":"check_in","title":"Completed daily check-in"}'
# Expected: {"success":true,"event":{...}}
```

### Get Activity Feed
```bash
curl http://localhost:3001/api/activity/feed?limit=10 \
  -H "Authorization: Bearer <JWT_TOKEN>"
# Expected: {"success":true,"events":[...]}
```

### Export Data
```bash
curl -X POST http://localhost:3001/api/export \
  -H "Authorization: Bearer <JWT_TOKEN>"
# Expected: {"success":true,"export":{...},"integrity":{"algorithm":"sha256","hash":"..."}}
```

### Contact Form (no auth)
```bash
curl -X POST http://localhost:3001/api/contact \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test","message":"Hello","reason":"General Inquiry"}'
# Expected: {"success":true,"message":"Message sent successfully"}
```

### Setup MongoDB Validation
```bash
# No longer needed manually — runs automatically on server startup.
# Can still be run standalone if needed:
cd server && node scripts/setupValidation.js
# Expected: ✓ Validation applied to il_check_ins, etc.
```

### Build Frontend
```bash
npm run build
# Expected: ✓ built in ~1s, zero errors
```

---

## 13. Deployment Status (Live Verification — 2026-03-26)

### Infrastructure
| Service | Domain | Status |
|---------|--------|--------|
| **Frontend (Innerlab F)** | `innerlab.ai` | ✅ Running |
| **Backend (Innerlab B)** | `api.innerlab.ai` | ✅ Running |
| **DNS** | `api.innerlab.ai → 72.61.69.223` | ✅ Resolves correctly |

### Live Endpoint Verification
| Test | Result |
|------|--------|
| `GET https://api.innerlab.ai/health` | ✅ `{"status":"ok","service":"innerlab-middleware"}` |
| `GET https://innerlab.ai/` | ✅ Homepage renders correctly |
| `GET https://innerlab.ai/modules` | ✅ 11 module cards, filters, 2 live |
| `GET https://innerlab.ai/dashboard` | ✅ Redirects to MBS Platform login (auth gate works) |
| Redirect URL format | ✅ `magicbusstudios.com/login?redirect=https://innerlab.ai/dashboard` |

### Coolify Configuration (Backend — Innerlab B)
- **Build Pack**: Dockerfile
- **Dockerfile Location**: `/Dockerfile.server`
- **Base Directory**: `/`
- **Port**: 3001
- **Domains**: `https://api.innerlab.ai` (was incorrectly `.com` — fixed by user)

---

## 14. Cross-Project Blocker: MBS Platform `/login` Route Missing

### What We Found
When visiting `innerlab.ai/dashboard` without a JWT:
1. Inner Lab's `ProtectedRoute` correctly detects no token ✅
2. Redirects to `magicbusstudios.com/login?redirect=https://innerlab.ai/dashboard` ✅
3. **MBS Platform shows a blank dark page** ❌

### Console Errors on MBS Platform
```
[WARNING] No routes matched location "/login?redirect=https://innerlab.ai/dashboard"
[EXCEPTION] Error: Uncaught TypeError: Cannot read properties of undefined (reading 'merchant')
```

### Root Cause
The MBS Platform (magicbusstudios.com) React Router does not have a `/login` route defined. The frontend app loads (title shows "MagicBusStudios — Tools for Inner Growth") but renders nothing because no route matches.

### Impact
- **Inner Lab Phase 2 is fully functional** — the redirect to MBS Platform works correctly
- **End-to-end auth flow cannot be tested** until MBS Platform has a working `/login` page that:
  1. Shows Google SSO + email/password login
  2. After successful login, issues a JWT
  3. Redirects back to the `redirect` URL with `?token=<JWT>` appended
- **Dashboard, consciousness, memories, and activity pages** cannot be reached by any user until this is resolved
- **Entitlement check** (`GET /api/entitlements/category/innerlab`) also depends on MBS Platform — untestable until login works

### Action Required (MBS Platform team)
1. Build `/login` route on `magicbusstudios.com` with Google SSO button
2. After successful auth, redirect to `redirect` URL param with `?token=<JWT>`
3. Build `/api/entitlements/category/:category` endpoint if not already done
4. Inner Lab will automatically work once these are live — no code changes needed on Inner Lab side

---

## 15. Remaining Known Issues (Non-Blocking)

| # | Issue | Severity | Notes |
|---|-------|----------|-------|
| 1 | `dashboardApi.js` has no 401 interceptor | Low | Expired tokens show generic error toasts instead of auto-redirecting to login. Works but not ideal UX. |
| 2 | `setupValidation.js` covers 4 of 11 collections | Low | Only `il_check_ins`, `il_user_memories`, `il_activity_feed`, `il_consciousness_profiles` have JSON Schema. Others rely on Mongoose-only validation. |
| 3 | Waitlist form missing `usage_frequency` field | Low | CLAUDE.md Section 12 specifies it in the payload shape, but it's not in the form UI. Not user-facing yet. |
| 4 | No `HEALTHCHECK` in `Dockerfile.server` | Low | Coolify still monitors via its own health checks. |
| 5 | `VITE_PLATFORM_URL` not in `.env.example` | Low | Used by AuthContext and ProtectedRoute with hardcoded fallback to `https://magicbusstudios.com`. Works but undocumented for local dev. |

---

## 16. Git History (This Phase)

| Commit | Message |
|--------|---------|
| `26655ca` | feat: add Inner Lab Middleware — Express server, Mongoose models, dashboard pages |
| `4009fa7` | fix: post-audit fixes — route bug, auth race condition, SEO canonicals |
| `bf6104a` | chore: auto-run MongoDB validation on server startup |

**Branch:** `main` (only branch — auto-deploys on push)
**All code committed and pushed.** No uncommitted local changes.

---

## 17. Phase 2 Completion Checklist

### Code ✅
- [x] JWT validation middleware (verifies MBS Platform tokens)
- [x] 11 Mongoose models for all il_* collections
- [x] API routes: check-in CRUD, consciousness CRUD, personal history CRUD, memories with sharing, activity feed, export, encryption placeholders
- [x] MongoDB JSON Schema validation (auto-runs on startup)
- [x] Dashboard pages: dashboard, consciousness, memories, activity (all auth-gated)
- [x] AuthContext with token extraction, entitlement check, 5-min cache, refresh bust
- [x] ProtectedRoute with auth gate + upsell fallback
- [x] Marketing pages unchanged and working
- [x] SEO canonical tags on all 11 pages
- [x] Frontend build: zero errors
- [x] All audit bugs fixed

### Deployment ✅
- [x] Backend container running on Coolify (`api.innerlab.ai`)
- [x] Frontend container running on Coolify (`innerlab.ai`)
- [x] Health endpoint responding
- [x] DNS resolving correctly

### Blocked by MBS Platform ⏳
- [ ] `/login` route on magicbusstudios.com (blank page — no React route)
- [ ] End-to-end auth flow test (login → JWT → redirect → dashboard)
- [ ] Entitlement endpoint test (`/api/entitlements/category/innerlab`)
- [ ] Dashboard page visual verification with real user data
