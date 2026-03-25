# Inner Lab Middleware + Dashboard — Layer 2

## What This Project Becomes
This project is `Innerlab/` — the innerlab.ai website. It currently serves as a **frontend-only marketing site** (no backend). It is being upgraded to ALSO serve as the **Inner Lab Middleware** (shared data layer) AND the **Inner Lab Dashboard** (unified experience for subscribers).

**After the upgrade:**
- `innerlab.ai` (public) — marketing pages (existing, unchanged)
- `innerlab.ai/dashboard` — daily briefing, consciousness profile, memories, activity feed (NEW — requires Inner Lab All Access)
- `innerlab.ai/modules` — module launcher (existing marketing → becomes functional)
- `innerlab.ai/api/...` — all Inner Lab middleware API endpoints (NEW)

The MBS Platform (magicbusstudios.com) knows WHO the user is and WHAT they've paid for.
This project knows the user's **consciousness, state, memories, and cross-module insights**.

Each Inner Lab module (CWG, FlowState, BreathArc, StarMap, etc.) is a separate container with its own backend. They all connect to the same `inner_lab` database. This middleware owns the shared `il_*` collections and provides cross-module intelligence.

## What This Project Does NOT Do
- Does NOT handle login or auth — MBS Platform (magicbusstudios.com) does that
- Does NOT handle billing — MBS Platform does that
- Does NOT replace individual module backends (CWG, FlowState keep their own backends)
- Does NOT own module-prefixed collections (cwg_*, yoga_*) — modules own their own data

## Stack
- **Frontend**: React (Vite) — existing marketing pages + NEW dashboard/consciousness/memories pages
- **Backend**: Express (NEW — `server/` folder to be created) — il_* APIs. Forms continue using the existing MBS form handler at api.magicbusstudios.com (no need to duplicate SendGrid setup here).
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
il_daily_check_ins          — Aggregated daily check-in data
il_user_memories            — Cross-session AI memory with opt-in sharing
il_shared_memories          — Memories user has consented to share across modules
il_activity_feed            — Cross-module activity events
il_blockchain_anchors       — OpenTimestamps data integrity proofs
il_sync_backups             — Local-first storage sync infrastructure
il_analytics_events         — User event tracking (TTL: 90 days)
il_notifications            — Unified notifications across Inner Lab modules
```

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

## User Memory System (Critical Design)

### Privacy Model: Option C — User Opt-In Sharing
- Each module creates memories in its own namespace by default (e.g., CWG writes to `il_user_memories` with `source: "cwg"`)
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

## API Endpoints

### Check-In
- `POST /check-in` — Store mood, energy, stress, intention (any module can call this)
- `GET /check-in/latest` — Get user's most recent check-in
- `GET /check-in/history` — Check-in history with date range

### Consciousness Profile
- `GET /consciousness` — Get user's consciousness profile
- `PUT /consciousness` — Update consciousness profile (from assessment)
- `GET /consciousness/snapshots` — Historical snapshots

### Personal History
- `GET /personal-history` — Get user's life story
- `PUT /personal-history` — Update personal history

### User Memories
- `GET /memories` — Get memories for current module (filtered by source + shared)
- `POST /memories` — Create a memory (from any module)
- `PUT /memories/:id/share` — User opts to share a memory across Inner Lab
- `PUT /memories/:id/unshare` — User revokes sharing
- `GET /memories/shared` — Get all shared memories (for cross-module use)

### Daily Briefing (Future)
- `GET /today` — Synthesized daily briefing from all module data
- Reads from: check-ins, memories, module activity, consciousness profile

### Cross-Module Insights (Future)
- `GET /insights` — Patterns detected across modules
- Example: "You've been exploring grief in CWG and doing calming breathwork — here's what we notice..."

### Activity Feed
- `POST /activity` — Log an activity event (any module calls this)
- `GET /activity/feed` — Cross-module activity feed for Inner Lab dashboard

### Encryption & Data Export
- `POST /export` — Generate encrypted data export across all Inner Lab modules
- `GET /encryption/keys` — Get user's encryption key metadata
- `POST /encryption/setup` — Initialize client-side encryption

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
| MONGODB_URI | (same as all MBS apps) | Shared MongoDB instance |
| DB_NAME | `inner_lab` | Same DB all IL modules use |
| JWT_SECRET | (same as MBS Platform) | To verify platform-issued JWTs |
| PORT | TBD | Not 3002 (that's MBS Platform) |
| CORS_ORIGINS | All Inner Lab frontend domains | CWG, FlowState, innerlab.ai, etc. |
| MBS_PLATFORM_URL | https://platform.magicbusstudios.com | For user lookups if needed |

## When to Build This

**Build immediately after MBS Platform (Layer 1).** This middleware must exist BEFORE the CWG and FlowState migrations so that shared data (consciousness profiles, check-ins, personal history, user memories, wellness profiles) can be written directly to `il_*` tables from day one. Without this, shared data would temporarily live as `cwg_*` or `yoga_*` and require a painful double-migration later.

**Build order:**
1. MBS Platform (Layer 1) — SSO + billing
2. **This middleware (Layer 2)** — shared il_* collections + APIs
3. CWG migration — writes shared data to il_* via this middleware
4. FlowState migration — same pattern
5. Inner Lab **frontend** (innerlab.ai dashboard) — built LATER when 2-3 modules have enough data

**What comes later (NOT this project):** The Inner Lab frontend/dashboard at innerlab.ai. That's a separate frontend project built when there are enough modules and data to power a daily briefing and cross-module insights.

## Relationship to Other Projects

| Project | What it does | Database |
|---------|-------------|----------|
| **MBS Platform** | SSO, billing, entitlements, friends, payments | `mbs_platform` |
| **Inner Lab Middleware** (this) | Shared consciousness, memories, insights, encryption | `inner_lab` (il_* collections) |
| **CWG Backend** | Spiritual guidance, chat, journaling | `inner_lab` (cwg_* collections) |
| **FlowState Backend** | Yoga, breathwork, meditation | `inner_lab` (yoga_* collections) |
| **BreathArc Backend** (future) | Guided breathwork | `inner_lab` (breath_* collections) |
| **Inner Lab Frontend** (future) | Unified dashboard at innerlab.ai | No DB (calls middleware + modules) |

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

## What NOT to Do
- Do NOT handle login or auth — MBS Platform does that
- Do NOT handle billing — MBS Platform does that
- Do NOT write to module-prefixed collections (cwg_*, yoga_*) — modules own their own data
- Do NOT hardcode module slugs — use config
- Do NOT build this until 2-3 modules exist and need cross-module data
- Do NOT merge with MBS Platform — this is a separate service

## Coding Rules
Follow all conventions from the global CLAUDE.md:
- Logger (never console.log)
- `{ success: true/false }` response format
- Input validation on all routes
- Rate limiting on API routes
- Document all env vars in `.env.example`
