# Inner Lab Middleware + Dashboard — Layer 2

## What This Project Becomes
This project is `Innerlab/` — the innerlab.ai website. It currently has a **frontend + backend** (Express with SendGrid form handler, deployed on Coolify). It is being upgraded to ALSO serve as the **Inner Lab Middleware** (shared data layer) AND the **Inner Lab Dashboard** (unified experience for subscribers).

**After the upgrade:**
- `innerlab.ai` (public) — marketing pages (existing, unchanged)
- `innerlab.ai/auth/login` — Inner Lab branded login page (NEW — calls MBS Platform auth APIs)
- `innerlab.ai/auth/signup` — Inner Lab branded signup page (NEW — calls MBS Platform auth APIs)
- `innerlab.ai/dashboard` — daily briefing, consciousness profile, memories, activity feed (NEW — requires Inner Lab All Access)
- `innerlab.ai/modules` — module launcher (existing marketing → becomes functional)
- `innerlab.ai/api/...` — all Inner Lab middleware API endpoints (NEW)

The MBS Platform (magicbusstudios.com) knows WHO the user is and WHAT they've paid for.
This project knows the user's **consciousness, state, memories, and cross-module insights**.

Each Inner Lab module (CWG, FlowState, BreathArc, StarMap, etc.) is a separate container with its own backend. They all connect to the same `inner_lab` database. This middleware owns the shared `il_*` collections and provides cross-module intelligence.

## What This Project Does NOT Do
- Does NOT handle auth backend — MBS Platform (magicbusstudios.com) has all auth API routes. This project has login/signup PAGES that call MBS Platform APIs.
- Does NOT handle billing — MBS Platform does that
- Does NOT replace individual module backends (CWG, FlowState keep their own backends)
- Does NOT own module-prefixed collections (cwg_*, yoga_*) — modules own their own data

## Stack
- **Frontend**: React (Vite) — existing marketing pages + NEW dashboard/consciousness/memories pages
- **Backend**: Express (EXISTING — `server/` folder already exists with SendGrid form handler) — add il_* APIs alongside existing form endpoints.
- **Database**: MongoDB — `DB_NAME: inner_lab` (same database all IL modules connect to)
- **Auth**: Validates JWTs issued by MBS Platform (does NOT handle login itself)
- **Deployment**: Coolify — 2 containers (frontend + backend), same pattern as all MBS apps

## Stack Deviations from Global
- **Auth**: No login/auth of its own — validates MBS Platform JWTs via middleware
- **No Stripe** — billing is handled by MBS Platform
- **Shares database** with all Inner Lab module backends — they all use `DB_NAME=inner_lab`
- **Frontend has both public and authenticated routes** — marketing is public, dashboard requires auth

## Architecture Context

```
┌─────────────────────────────────────────────────────────┐
│  MBS Platform (Layer 1)                                  │
│  mbs_platform DB                                         │
│  SSO + billing + entitlements + friends + invites        │
│  Auth: Google SSO + Nostr + LNURL                       │
│  Payments: Stripe + BTCPay                              │
└────────────────────────┬────────────────────────────────┘
                         │ JWT validation
┌────────────────────────▼────────────────────────────────┐
│  Inner Lab Middleware (Layer 2) ← THIS PROJECT           │
│  inner_lab DB (shared with all modules below)            │
│                                                          │
│  Owns: il_* collections (shared data)                   │
│  Provides: consciousness API, memories API, check-in    │
│            API, daily briefing, cross-module insights,   │
│            activity feed, encryption/data export         │
│                                                          │
│  Other containers also connect to inner_lab DB:          │
│  ├── CWG Backend      → reads/writes cwg_* + reads il_* │
│  ├── FlowState Backend → reads/writes yoga_* + reads il_*│
│  ├── BreathArc Backend → reads/writes breath_* + reads il│
│  └── ...future modules                                   │
└──────────────────────────────────────────────────────────┘
```

## Database: `inner_lab`

All Inner Lab containers (this middleware + every module backend) connect to the SAME database. Collections are namespaced by prefix.

### Shared Collections (owned by this middleware — `il_*` prefix)

These are created when data needs to be shared across modules. They start empty and get populated as modules contribute.

```
il_consciousness_profiles   — Spiritual assessment, archetype type, orientation
il_consciousness_snapshots  — Historical profile changes over time
il_personal_histories       — User's life story for AI personalization
il_check_ins                — Daily mood/energy/stress/intention (any module can write)
il_user_memories            — Cross-session AI memory with opt-in sharing (shared: false = private to source module, shared: true = visible to all IL modules)
il_user_wellness_profiles   — Health conditions, injuries, goals (shared by FlowState + BreathArc)
il_activity_feed            — Cross-module activity events
il_blockchain_anchors       — OpenTimestamps data integrity proofs
il_sync_backups             — Local-first storage sync infrastructure
il_analytics_events         — User event tracking (TTL: 90 days)
il_notifications            — Unified notifications across Inner Lab modules
```

**Note on removed collections:**
- `il_daily_check_ins` (aggregated) — removed. Use `il_check_ins` with date queries instead of a separate aggregation collection. Simpler, no sync issues.
- `il_shared_memories` — removed. Shared memories are simply `il_user_memories` documents where `shared: true`. No need for a separate collection.

### Module Collections (owned by each module backend)

Each module backend owns its own prefixed collections. This middleware does NOT write to them — only reads (for cross-module intelligence).

```
CWG:       cwg_chat_sessions, cwg_messages, cwg_journal_entries, cwg_practices, ...
FlowState: yoga_sessions, yoga_achievements, yoga_user_profiles, yoga_community_flows, ...
BreathArc: breath_sessions, breath_tracks, breath_preferences, ... (future)
StarMap:   astro_charts, astro_transits, ... (future)
```

### Database Rule
- Modules **write** to their own prefix (`cwg_*`, `yoga_*`) and to shared `il_*` tables
- Modules **read** from `il_*` shared tables and their own prefix only
- This middleware **reads** from all prefixes (for cross-module insights) and **writes** only to `il_*`

## Schema Contracts for il_* Collections (CRITICAL)

Every module writing to shared il_* collections MUST validate documents against these schemas. Node modules use Mongoose schemas, Python modules use Pydantic models. After collections are created, apply MongoDB JSON Schema validation as a safety net.

### il_check_ins
```javascript
{
  _id: ObjectId,
  user_id: String,           // MBS Platform user ID (from JWT userId)
  source_module: String,     // "cwg", "yoga", "breatharc", etc.
  mood: Number,              // 1-10 scale
  energy: Number,            // 1-10 scale
  stress: Number,            // 1-10 scale
  intention: String,         // optional — user's intention for the day
  notes: String,             // optional — freeform notes
  created_at: DateTime
}
```

### il_consciousness_profiles
```javascript
{
  _id: ObjectId,
  user_id: String,           // unique per user
  archetype: String,         // primary archetype from assessment
  orientation: String,       // spiritual orientation
  assessment_data: Object,   // full assessment Q&A (flexible structure)
  source_module: String,     // which module last updated this
  created_at: DateTime,
  updated_at: DateTime
}
```

### il_personal_histories
```javascript
{
  _id: ObjectId,
  user_id: String,           // unique per user
  content: Object,           // structured personal history (flexible — normalized from CWG's dual formats)
  source_module: String,
  created_at: DateTime,
  updated_at: DateTime
}
```

### il_user_wellness_profiles
```javascript
{
  _id: ObjectId,
  user_id: String,           // unique per user
  health_conditions: [String], // e.g., ["back pain", "anxiety"]
  injuries: [String],          // e.g., ["left knee"]
  goals: [String],             // e.g., ["flexibility", "stress reduction"]
  source_module: String,       // which module last updated this
  created_at: DateTime,
  updated_at: DateTime
}
```

### il_activity_feed
```javascript
{
  _id: ObjectId,
  user_id: String,
  source_module: String,     // "cwg", "yoga", "breatharc", etc.
  action: String,            // "session_complete", "achievement_earned", "journal_entry", etc.
  title: String,             // human-readable title for the feed item
  description: String,       // optional — additional detail
  metadata: Object,          // optional — module-specific data
  created_at: DateTime
}
```

### MongoDB-Level Validation (Safety Net)
After creating il_* collections, apply JSON Schema validation:
```javascript
db.runCommand({ collMod: 'il_user_memories', validator: {
  $jsonSchema: {
    required: ['user_id', 'source_module', 'memory_type', 'content', 'shared', 'created_at'],
    properties: {
      confidence: { bsonType: 'double', minimum: 0, maximum: 1 }
    }
  }
}});
```
Apply similar validation to il_check_ins (mood/energy/stress: 1-10), il_activity_feed (required fields), etc.

### Schema Enforcement Rule
Every module MUST validate documents before writing to any il_* collection. Use Mongoose schemas (Node) or Pydantic models (Python) that match these contracts exactly. MongoDB-level validation is the backstop, not the primary defense.

## User Memory System (Critical Design)

### Privacy Model: Option C — User Opt-In Sharing
- Each module creates memories in its own namespace by default (e.g., CWG writes to `il_user_memories` with `source_module: "cwg"`)
- Memories are **private to the originating module** by default
- User can explicitly opt-in to share specific memories (or all memories from a module) across Inner Lab
- Shared memories get a `shared: true` flag and become readable by other modules
- **Why**: Inner Lab deals with deeply personal/spiritual data. User must control what crosses module boundaries.

### Memory Document Structure
```javascript
{
  _id: ObjectId,
  user_id: String,           // MBS Platform user ID
  source_module: String,     // "cwg", "yoga", "breatharc", etc.
  memory_type: String,       // "fact", "preference", "emotional_state", "insight"
  content: String,           // "User is processing grief over loss of parent"
  confidence: Number,        // 0-1, how certain the AI is about this memory
  shared: Boolean,           // false = private to source module, true = visible to all IL modules
  shared_at: DateTime,       // when user opted to share (null if not shared)
  created_at: DateTime,
  updated_at: DateTime,
  expires_at: DateTime       // optional TTL for transient memories like mood
}
```

## Data Write Paths (IMPORTANT — Read This First)

There are TWO ways data gets written to il_* collections:

1. **Direct DB writes** — Module backends (CWG, FlowState, etc.) share the `inner_lab` database and write directly to il_* collections via Mongoose/Motor. This is the primary write path for modules. No HTTP call needed.
2. **HTTP API** — The endpoints below are called by the **Inner Lab dashboard frontend** (innerlab.ai) only. Example: the dashboard's check-in widget calls `POST /api/check-in`.

**Rule:** Modules write to il_* via shared DB. The dashboard frontend writes via these HTTP endpoints. Both paths write to the same collections. Keep validation logic consistent — if you add validation to the HTTP endpoint, document the same rules for direct DB writers.

**CORS_ORIGINS** only needs to include `innerlab.ai` because module backends don't call these HTTP endpoints.

### userId Mapping
The JWT uses `userId` (camelCase) but all il_* database documents use `user_id` (snake_case). When writing to any il_* collection, map: `user_id: req.user.userId`.

## API Endpoints

All endpoints are mounted at `/api/` prefix (e.g., `POST /api/check-in`).

### Check-In
- `POST /api/check-in` — Store mood, energy, stress, intention (dashboard frontend calls this)
- `GET /api/check-in/latest` — Get user's most recent check-in
- `GET /api/check-in/history` — Check-in history with date range

### Consciousness Profile
- `GET /api/consciousness` — Get user's consciousness profile
- `PUT /api/consciousness` — Update consciousness profile (from assessment)
- `GET /api/consciousness/snapshots` — Historical snapshots

### Personal History
- `GET /api/personal-history` — Get user's life story
- `PUT /api/personal-history` — Update personal history

### User Memories
- `GET /api/memories` — Get memories for current module (filtered by source + shared)
- `POST /api/memories` — Create a memory (from any module)
- `PUT /api/memories/:id/share` — User opts to share a memory across Inner Lab
- `PUT /api/memories/:id/unshare` — User revokes sharing
- `GET /api/memories/shared` — Get all shared memories (for cross-module use)

### Daily Briefing (Future)
- `GET /api/today` — Synthesized daily briefing from all module data
- Reads from: check-ins, memories, module activity, consciousness profile

### Cross-Module Insights (Future)
- `GET /api/insights` — Patterns detected across modules
- Example: "You've been exploring grief in CWG and doing calming breathwork — here's what we notice..."

### Activity Feed
- `POST /api/activity` — Log an activity event (any module calls this)
- `GET /api/activity/feed` — Cross-module activity feed for Inner Lab dashboard

### Encryption & Data Export
- `POST /api/export` — Generate encrypted data export across all Inner Lab modules
- `GET /api/encryption/keys` — Get user's encryption key metadata
- `POST /api/encryption/setup` — Initialize client-side encryption

## Frontend Pages (Add to Existing Marketing Site)

The existing innerlab.ai React frontend has marketing pages. Add these authenticated pages:

### Auth Pages (NEW — call MBS Platform APIs, no local auth backend)
- `innerlab.ai/auth/login` — Inner Lab branded login page
  - Google SSO button on top ("Sign in with Google") — sends Google ID token to `POST ${VITE_PLATFORM_URL}/api/auth/google` with `{ idToken, source: "innerlab" }`. Returns `{ token, user }`. Store token as `mbs_token`, user as `mbs_user` in localStorage.
  - "or" divider
  - Nostr + Lightning (LNURL) buttons — call MBS Platform Nostr/LNURL challenge APIs
  - "or use email" divider
  - Email + Password form → `POST ${VITE_PLATFORM_URL}/api/auth/login` with `{ email, password }`. If response has `requires2FA: true`, show 6-digit TOTP input (+ "Use backup code" link) → `POST ${VITE_PLATFORM_URL}/api/auth/2fa/verify`
  - "Don't have an account? Create one" link → `/auth/signup`
  - "Forgot password?" link → calls `POST ${VITE_PLATFORM_URL}/api/auth/forgot-password`
  - After successful auth, redirect to `/dashboard` (or `?redirect=` param if present)
  - If user already logged in (valid `mbs_token`), auto-redirect to `/dashboard`
  - Styled with Inner Lab branding (dark theme, IL logo, colors from existing marketing pages)

- `innerlab.ai/auth/signup` — Inner Lab branded signup page
  - Google SSO button on top ("Sign up with Google") — same flow as login, auto-creates account
  - "or" divider
  - Signup form: Name, Email, Password (8+ chars), Confirm Password
  - Age confirmation checkbox (13+, 16+ in EU/EEA/UK)
  - Terms of Service + Privacy Policy acceptance checkbox (required)
  - "Create Account" button → `POST ${VITE_PLATFORM_URL}/api/auth/signup` with `{ name, email, password, age_confirmed: true, terms_accepted: true }`
  - "Already have an account? Sign in" link → `/auth/login`
  - After successful signup, redirect to `/dashboard` (or `?redirect=` param)
  - Same Inner Lab branding as login page

- `innerlab.ai/auth/reset-password` — Password reset page
  - Reads `?token=` from URL
  - Form: New Password + Confirm Password
  - Calls `POST ${VITE_PLATFORM_URL}/api/auth/reset-password` with `{ token, password }`
  - On success, redirect to `/auth/login?reset=true`

**Auth env vars (frontend build args):**
- `VITE_PLATFORM_URL` — `https://magicbusstudios.com` (all auth API calls go here)
- `VITE_GOOGLE_CLIENT_ID` — same Google OAuth client ID as MBS Platform

**Token handling on ANY page load:**
On app initialization (e.g., in App.jsx or a top-level AuthProvider), check for `?token=` query param. If present: extract it, store as `mbs_token`, fetch user profile from `GET ${VITE_PLATFORM_URL}/api/auth/me`, store as `mbs_user`, then `history.replaceState` to remove token from URL. This handles redirects back from MBS Platform for edge cases.

**Nav bar auth state:**
- Not logged in: show "Sign In" button → `/auth/login`
- Logged in: show user avatar/name + dropdown with "Dashboard", "Account" (links to MBS Platform account page), "Sign Out"
- Sign Out: clear `mbs_token` and `mbs_user` from localStorage, redirect to `/`

### Dashboard Pages
- `innerlab.ai/dashboard` — Main dashboard (requires auth + Inner Lab All Access entitlement)
  - Module launcher: links to all active modules with status indicators
  - Recent activity feed (from il_activity_feed)
  - Current consciousness snapshot summary
  - Quick check-in widget (mood, energy, stress, intention)
  - Daily briefing (placeholder/coming soon until enough data exists)
- `innerlab.ai/consciousness` — Consciousness profile viewer/editor
  - Shows archetype, orientation, assessment results
  - Historical snapshots timeline
- `innerlab.ai/memories` — User memories manager
  - List memories by module, with sharing toggle
  - Shared vs private filter
- `innerlab.ai/activity` — Full activity feed (cross-module)

### Auth-Gated Routing
- Marketing pages (/, /modules, /about) — public, no auth required
- Auth pages (/auth/login, /auth/signup, /auth/reset-password) — public, redirect to /dashboard if already logged in
- Dashboard pages (/dashboard, /consciousness, /memories, /activity) — require JWT + entitlement check
- If user is not logged in: redirect to /auth/login (NOT to MBS Platform — Inner Lab has its own login page)
- If user has no Inner Lab All Access: show upsell page linking to MBS Platform billing

### Entitlement Check
Dashboard requires `category_access: "innerlab"` or `mbs_all_access`. Check via:
```
GET magicbusstudios.com/api/entitlements/category/innerlab
```
If user has only a `product_pass` for one module, they can use that module directly but NOT the dashboard.

## Auth Middleware

This service does NOT handle login. It validates JWTs from the MBS Platform:

```javascript
// Every route uses this middleware
async function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split('Bearer ')[1];
  if (!token) return res.status(401).json({ success: false, message: 'No token' });

  try {
    // Verify JWT was issued by MBS Platform
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // { userId, email, name, ... }
    next();
  } catch (err) {
    return res.status(401).json({ success: false, message: 'Invalid token' });
  }
}
```

## Environment Variables

| Variable | Value | Notes |
|----------|-------|-------|
**NEW (add to Coolify):**
| MONGO_URL | (same as all MBS apps) | Shared MongoDB instance |
| DB_NAME | `inner_lab` | Same DB all IL modules use |
| JWT_SECRET | (same as MBS Platform) | To verify platform-issued JWTs |
| PORT | `3001` | Default for Inner Lab (MBS Platform defaults to 3001 locally too — Coolify assigns real ports in production) |
| CORS_ORIGINS | innerlab.ai frontend only | Dashboard frontend is the only HTTP caller |
| PLATFORM_URL | `https://magicbusstudios.com` | For entitlement checks |

**EXISTING (keep — form handler needs these):**
| SENDGRID_API_KEY | (existing) | Used by existing contact/subscribe/waitlist forms |
| FROM_EMAIL | (existing) | SendGrid sender |
| TO_EMAIL | (existing) | SendGrid recipient |

## When to Build This

**Build immediately after MBS Platform (Layer 1).** This middleware must exist BEFORE the CWG and FlowState migrations so that shared data (consciousness profiles, check-ins, personal history, user memories, wellness profiles) can be written directly to `il_*` tables from day one. Without this, shared data would temporarily live as `cwg_*` or `yoga_*` and require a painful double-migration later.

**Build order:**
1. MBS Platform (Layer 1) — SSO + billing
2. **This project (Layer 2)** — backend (il_* collections + APIs) + frontend (dashboard shell, consciousness profile, activity feed)
3. CWG migration — writes shared data to il_* via this middleware
4. FlowState migration — same pattern
5. Daily briefing + cross-module insights — built LATER when 2-3 modules have enough data to power them

**The Inner Lab frontend/dashboard** is built as part of THIS project (innerlab.ai). The dashboard pages (daily briefing, consciousness profile, memories, activity feed) are added to the existing React frontend alongside the marketing pages. Daily briefing and cross-module insights features are deferred until enough modules have data, but the dashboard shell, consciousness profile view, and activity feed should be built now.

## Relationship to Other Projects

| Project | What it does | Database |
|---------|-------------|----------|
| **MBS Platform** | SSO, billing, entitlements, friends, payments | `mbs_platform` |
| **Inner Lab Middleware** (this) | Shared consciousness, memories, insights, encryption | `inner_lab` (il_* collections) |
| **CWG Backend** | Spiritual guidance, chat, journaling | `inner_lab` (cwg_* collections) |
| **FlowState Backend** | Yoga, breathwork, meditation | `inner_lab` (yoga_* collections) |
| **BreathArc Backend** (future) | Guided breathwork | `inner_lab` (breath_* collections) |
| **Inner Lab Frontend** (this project) | Dashboard + marketing at innerlab.ai | No own DB (calls middleware APIs) |

## Product Catalog (Inner Lab Modules)

Do NOT hardcode these. Store in config so new modules can be added.

| Slug | Name | Status | Domain |
|------|------|--------|--------|
| cwg | Conversations With God | Active | conversationswithgod.ai |
| flowstate | FlowState | Active | yoga.magicbusstudios.com |
| breatharc | BreathArc | Coming Soon | TBD |
| starmap | StarMap | Coming Soon | TBD |
| astrocompass | AstroCompass | Coming Soon | TBD |
| arcana | Arcana | Coming Soon | TBD |
| archetypes | Archetypes | Coming Soon | TBD |
| dreamlens | DreamLens | Coming Soon | TBD |
| rituals | Rituals | Coming Soon | TBD |
| innerquest | InnerQuest | Coming Soon | TBD |
| nexus | Nexus | Coming Soon | TBD |

## Project Structure (After Upgrade)
```
Innerlab/
├── src/                    # React frontend
│   ├── pages/              # Existing marketing pages + NEW auth + dashboard pages
│   │   ├── HomePage.jsx         # existing — marketing landing
│   │   ├── ModulesPage.jsx      # existing — module catalog
│   │   ├── AuthLoginPage.jsx    # NEW — Inner Lab branded login (calls MBS Platform APIs)
│   │   ├── AuthSignupPage.jsx   # NEW — Inner Lab branded signup (calls MBS Platform APIs)
│   │   ├── AuthResetPasswordPage.jsx # NEW — password reset
│   │   ├── DashboardPage.jsx    # NEW — main dashboard (auth-gated)
│   │   ├── ConsciousnessPage.jsx # NEW — consciousness profile
│   │   ├── MemoriesPage.jsx     # NEW — user memories manager
│   │   └── ActivityPage.jsx     # NEW — cross-module activity feed
│   ├── components/
│   └── ...
├── server/
│   ├── index.js            # Express entry — EXISTING form handler + NEW il_* APIs
│   ├── routes/
│   │   ├── forms.js        # EXISTING — contact, subscribe, waitlist (SendGrid)
│   │   ├── consciousness.js # NEW — consciousness profile CRUD
│   │   ├── memories.js     # NEW — user memories CRUD + sharing
│   │   ├── checkins.js     # NEW — daily check-ins
│   │   ├── activity.js     # NEW — activity feed
│   │   └── export.js       # NEW — encryption & data export
│   ├── models/             # NEW — Mongoose models for il_* collections
│   ├── middleware/          # NEW — requireAuth (JWT validation)
│   ├── services/           # NEW — briefing engine, insight engine
│   └── config/             # NEW — module catalog
├── Dockerfile
├── nginx.conf
└── package.json
```

## What NOT to Do
- Do NOT build auth backend routes — MBS Platform owns all auth APIs. This project has login/signup PAGES that call MBS Platform APIs cross-origin.
- Do NOT handle billing — MBS Platform does that
- Do NOT write to module-prefixed collections (cwg_*, yoga_*) — modules own their own data
- Do NOT hardcode module slugs — use config
- Do NOT wait until multiple modules exist — build this middleware NOW (immediately after MBS Platform)
- Do NOT merge with MBS Platform — this is a separate service

## Coding Rules
Follow all conventions from the global CLAUDE.md:
- Logger (never console.log)
- `{ success: true/false }` response format
- Input validation on all routes
- Rate limiting on API routes
- Document all env vars in `.env.example`

---

## Completion Report (REQUIRED)

When you finish building, generate a file called `PHASE_2_REPORT.md` in the project root. The orchestrator will fetch it from here. The report must contain:

1. **What was built** — Every file created or modified, grouped by backend/frontend
2. **What changed from the plan** — Any deviations from this document. Did you rename anything? Skip anything? Add anything not in the spec? Why?
3. **Env vars required** — Complete list of every env var the code expects, with example values
4. **Database collections** — Exact name of every Mongoose model and its MongoDB collection name
5. **API routes** — Full table (method, path, auth required?, description)
6. **Frontend routes** — Every React route added (path, component, auth-gated?)
7. **il_* schema contracts** — Exact field names and types for each il_* collection as actually implemented
8. **JWT validation** — How your middleware validates tokens (header format, algorithm, fields extracted)
9. **Assumptions made** — Anything you had to decide that wasn't explicitly in the spec
10. **Known gaps** — Anything intentionally deferred or couldn't complete
11. **Gotchas for downstream phases** — Anything Phase 3 (CWG migration) or Phase 4 (FlowState migration) agents need to know — especially il_* collection field names if they differ from the spec, how modules should write to shared collections, or any middleware behavior they need to match
12. **Testing commands** — Exact curl commands or steps to verify each feature works

This report is critical — the orchestrator session (MBSPlatform repo) uses it to update instructions for all downstream phases before they start.

---

## Phase 1 Learnings (Added by Orchestrator — 2026-03-26)

These are real-world implementation details from the MBS Platform build that affect this phase:

### Env Var Name
- The MBS Platform uses **`MONGO_URL`** (not `MONGODB_URI`). Use `MONGO_URL` for consistency across all MBS apps.

### JWT Details (as actually implemented)
- Header: `Authorization: Bearer <token>`
- Payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- `userId` is a **string** (MongoDB ObjectId.toString()). If you need an ObjectId, cast it: `new mongoose.Types.ObjectId(req.user.userId)`
- Token expiry: 7 days, no refresh endpoint yet

### Frontend Token Storage
- Products store the JWT in `localStorage.getItem("mbs_token")`
- User profile in `localStorage.getItem("mbs_user")` — JSON: `{ id, email, name, avatar }`
- Token is passed via URL param `?token=<JWT>` after login redirect. Product frontends must extract it, store it, and remove from URL with `history.replaceState`.

### ConsentAuditLog.user_id is String
- The GDPR consent audit log stores `user_id` as a **String**, not ObjectId. This is because GDPR deletion replaces the user_id with a SHA-256 hash. If Phase 2 writes to this collection, use `.toString()`.

### GDPR Cascade
- The MBS Platform DELETE /api/auth/account connects to the `inner_lab` database and deletes all documents where `user_id` matches. All il_* collections MUST use `user_id` consistently (not `userId` or `user`).

### Category Values
- Categories are lowercase no-separator: `innerlab`, `arcade`, `studioworks` (not `inner_lab` or `Inner Lab`)

### Entitlement Response
- `GET /api/entitlements/:product` returns `{ success, hasAccess, reason }`
- Reason values: `mbs_all_access`, `category_access`, `product_pass`, `free_tier`, `no_subscription` (confirmed from live code)
- Products should cache entitlement responses for 5 minutes (in-memory, key: `userId:productSlug`)
- After billing success, redirect with `?refresh=true` to bust cache

### BTCPay
- BTCPay entitlements are **30-day non-recurring passes**. No auto-renew. User must repurchase.
- BTCPay webhook uses HMAC-SHA256 verification with `btcpay-sig` header and `BTCPAY_WEBHOOK_SECRET` env var.

### First Platform User
- Abhinav Gupta (`1984.abhinav@gmail.com`), platform ID: `69c53401fe8f1763b9046ae5`. This user exists ONLY in `mbs_platform` — no link to CWG or FlowState data yet.

### Billing Page Status
- Billing page is LIVE with real pricing (CWG $9.99/mo, IL All Access $19.99/mo, MBS All Access $29.99/mo) and Lightning payment toggle. CWG Stripe price IDs configured. IL/MBS bundle price IDs still need Stripe Dashboard setup.

---

## Phase 1 Learnings (SUPPLEMENT — added after Phase 1 build)

**Read this before starting Phase 2 work. These are confirmed facts from the live MBS Platform.**

### JWT Payload (confirmed from live code)
```javascript
{ userId: "ObjectId string", email: "..." || null, name: "...", avatar: "..." || null, isAdmin: false, iat, exp }
```
- **`userId` is camelCase, string** — use `req.user.userId` after verification
- **`email` can be null** — Nostr/LNURL users are pseudonymous. Do NOT assume email exists.
- JWT_EXPIRY defaults to `"7d"`, configurable via env var

### Auth Header
```
Authorization: Bearer <JWT>
```
Verify with: `jwt.verify(token, process.env.JWT_SECRET)` — HS256 shared secret.

### localStorage Key
Token stored as `mbs_token` — `localStorage.getItem("mbs_token")`

### user_id in il_* Collections
Store as `user_id` (snake_case) in all il_* MongoDB documents. The value is the `userId` string from the JWT (which is the ObjectId.toString() from mbs_platform.users). The GDPR cascade delete in MBS Platform searches inner_lab collections for documents matching `user_id`.

### Rate Limiting
MBS Platform has rate limiting: 100 req/15min general, 20 req/15min auth, 30 req/15min billing. If Inner Lab middleware calls the platform API (e.g., entitlement checks), implement retry with backoff on 429.

### Entitlement Check (confirmed format)
```
GET https://magicbusstudios.com/api/entitlements/{slug}
Authorization: Bearer <JWT>
→ { success: true, hasAccess: true/false, reason: "product_pass"|"category_access"|"mbs_all_access"|"free_tier"|"none" }
```
Dashboard pages should check entitlement for `category_access` on category `innerlab` to verify Inner Lab All Access.

### BTCPay Status (KNOWN ISSUE)
BTCPay API key has insufficient permissions (403). Lightning payments don't work until the BTCPay API key is regenerated with full store permissions. This does NOT block Phase 2 (no payments in IL middleware).

### Routes Deployed at MBS Platform
See `phase-reports/PHASE_1_LEARNINGS.md` for the complete 40-route table.
