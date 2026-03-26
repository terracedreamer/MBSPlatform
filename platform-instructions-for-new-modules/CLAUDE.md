# Inner Lab Module — Starter Template

> **How to use this file**: Copy this entire folder into your new module's project folder. Rename this file or merge it into your project's CLAUDE.md. Then add a section at the top describing what YOUR specific module does (name, purpose, features). Everything below is the shared context every Inner Lab module needs.

---

## You Are Building an Inner Lab Module

You are building a module within the **Inner Lab** ecosystem — a unified system for inner growth by Magic Bus Studios. Your module is one of up to 11 modules that share a database and user context.

**Your module does NOT handle:**
- Login/authentication — the MBS Platform at magicbusstudios.com handles this
- Billing/payments — the MBS Platform handles Stripe + BTCPay
- User identity (email, name, avatar) — lives in `mbs_platform` database
- Shared consciousness data — the Inner Lab Middleware at innerlab.ai handles this

**Your module DOES handle:**
- Its own product-specific features and logic
- Its own prefixed collections in the `inner_lab` database
- Reading shared `il_*` data to personalize the user experience
- Writing to shared `il_*` data when the module generates insights about the user

---

## Architecture You Must Know

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 1: MBS Platform (magicbusstudios.com)                  │
│  Database: mbs_platform                                       │
│  Handles: SSO (Google + Nostr + LNURL), billing (Stripe +    │
│           BTCPay), entitlements, friends, invites             │
│  Your module calls: GET /api/entitlements/{your_slug}         │
│  JWT_SECRET: must match the platform's secret                 │
└──────────────────────┬───────────────────────────────────────┘
                       │ JWT validation
┌──────────────────────▼───────────────────────────────────────┐
│  Layer 2: Inner Lab Middleware (innerlab.ai)                  │
│  Database: inner_lab (SAME database your module uses)         │
│  Owns: il_* shared collections                               │
│  APIs: consciousness, memories, check-ins, wellness, feed    │
│  Your module can call these APIs or read il_* directly        │
└──────────────────────┬───────────────────────────────────────┘
                       │ same database
┌──────────────────────▼───────────────────────────────────────┐
│  YOUR MODULE (separate container, same inner_lab database)    │
│  Owns: {prefix}_* collections (e.g., breath_*, astro_*)      │
│  Reads: il_* shared collections                              │
│  Writes: il_* when generating user insights                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Database: `inner_lab`

**Your module connects to the same database as ALL other Inner Lab modules.** Use `DB_NAME=inner_lab`.

### Your Collections (you own these)
Name all your collections with your module's prefix. Examples:
```
breath_sessions          — BreathArc session data
breath_tracks            — BreathArc saved tracks
breath_preferences       — BreathArc user preferences
astro_charts             — StarMap chart data
astro_transits           — StarMap transit calculations
```

**Choose a short prefix** (e.g., `breath_`, `astro_`, `tarot_`, `dream_`, `ritual_`, `quest_`, `nexus_`). Never use `cwg_`, `yoga_`, or `il_` — those are taken.

### Shared Collections (owned by Inner Lab Middleware — you can read and contribute)
```
il_consciousness_profiles   — User's spiritual assessment and archetype
il_consciousness_snapshots  — Historical profile changes
il_personal_histories       — User's life story for AI personalization
il_check_ins                — Mood, energy, stress, intention (any module can write)
il_user_memories            — AI-extracted facts about the user (shared: false = private, shared: true = cross-module)
il_user_wellness_profiles   — Health conditions, injuries, goals
il_activity_feed            — Cross-module activity events
il_blockchain_anchors       — OpenTimestamps data integrity proofs
il_sync_backups             — Local-first storage sync infrastructure
il_notifications            — Cross-module notifications
il_analytics_events         — User event tracking (90-day TTL)
```

### userId Mapping
The JWT uses `userId` (camelCase) but all database documents use `user_id` (snake_case). When writing to any collection, map: `user_id: req.user.userId`.

### Database Rules
- **Write** to your own prefix (`{prefix}_*`) freely
- **Write** to `il_check_ins` when your module captures mood/energy data
- **Write** to `il_user_memories` when your AI extracts facts about the user (set `source_module: "{your_slug}"`, `shared: false`)
- **Write** to `il_activity_feed` when significant events happen (session complete, achievement earned, etc.)
- **Read** from `il_*` to personalize your module (e.g., read consciousness profile to tailor recommendations)
- **NEVER write** to another module's prefix (`cwg_*`, `yoga_*`)

### User Memory Privacy Model
When your module's AI learns something about the user:
1. Write to `il_user_memories` with `source_module: "{your_slug}"` and `shared: false`
2. Only YOUR module can see memories with `shared: false` and `source_module: "{your_slug}"`
3. The user can opt-in to share memories across Inner Lab modules (sets `shared: true`)
4. When `shared: true`, ALL Inner Lab modules can read that memory
5. Never read another module's unshared memories

Memory document structure:
```javascript
{
  _id: ObjectId,
  user_id: String,           // platform user ID (from JWT)
  source_module: String,     // your module slug
  memory_type: String,       // "fact", "preference", "emotional_state", "insight"
  content: String,           // what the AI learned
  confidence: Number,        // 0-1, how certain the AI is about this memory
  shared: Boolean,           // false = private to your module, true = visible to all IL modules
  shared_at: DateTime,       // when user opted to share (null if not shared)
  created_at: DateTime,
  updated_at: DateTime,
  expires_at: DateTime       // optional TTL for transient memories like mood
}
```

---

## Authentication

Your module does NOT handle login. It validates JWTs issued by the MBS Platform.

### JWT Validation Middleware
```javascript
const jwt = require('jsonwebtoken');

async function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split('Bearer ')[1];
  if (!token) return res.status(401).json({ success: false, message: 'No token' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // { userId, email, name, ... }
    next();
  } catch (err) {
    return res.status(401).json({ success: false, message: 'Invalid token' });
  }
}
```

### Login Flow
```
User visits your-module.magicbusstudios.com (or your-module.innerlab.ai)
  → Clicks "Login"
  → Redirect to: magicbusstudios.com/auth/login?redirect={your_domain}&brand=innerlab
  → User authenticates (Google SSO / Nostr / LNURL)
  → MBS Platform redirects back: {your_domain}?token={JWT}
  → Your frontend stores JWT, sends in Authorization header
  → Your backend validates JWT on every request
```

### Checking Entitlements
Before giving access to premium features, check with the platform:
```javascript
// Call from your backend
const response = await fetch(`${PLATFORM_URL}/api/entitlements/${YOUR_SLUG}`, {
  headers: { Authorization: `Bearer ${userToken}` }
});
const { hasAccess, reason } = await response.json();
// hasAccess: true/false
// reason: "free_tier", "product_pass", "category_access", "mbs_all_access", "no_subscription"
```

If `hasAccess: false` and `reason: "no_subscription"`:
- Redirect user to `magicbusstudios.com/billing?product={YOUR_SLUG}&brand=innerlab`

---

## Environment Variables

| Variable | Value | Notes |
|----------|-------|-------|
| MONGO_URL | (same as all MBS apps) | Shared MongoDB on Coolify |
| DB_NAME | `inner_lab` | Same DB as all Inner Lab modules |
| JWT_SECRET | (same as MBS Platform) | Must match for JWT validation |
| PORT | (unique per module) | Don't conflict with other modules |
| CORS_ORIGINS | Your frontend domain(s) | |
| PLATFORM_URL | `https://magicbusstudios.com` | For entitlement checks |
| PRODUCT_SLUG | Your module's slug | e.g., `breatharc`, `starmap` |

Optional (if your module uses AI):
| OPENAI_API_KEY | (shared MBS key) | Behind a service layer, never in frontend |

---

## Project Structure (Recommended)

Follow the standard MBS two-container pattern:
```
your-module/
├── src/                    # React frontend (Vite)
│   ├── pages/
│   ├── components/
│   ├── store/              # Zustand for local state
│   ├── hooks/
│   ├── lib/api.js          # API client with JWT header
│   └── ...
├── server/
│   ├── index.js            # Express entry
│   ├── routes/             # Module-specific routes
│   ├── middleware/          # requireAuth (JWT validation)
│   ├── services/           # AI service, module logic
│   └── config/
├── Dockerfile
├── nginx.conf
├── CLAUDE.md               # THIS FILE (customized for your module)
└── package.json
```

---

## Coding Rules (From MBS Global Conventions)
- Use `logger` for all logging — never `console.log`
- Response format: `{ success: true, ...data }` or `{ success: false, message: "..." }`
- Input validation on all routes
- Rate limiting on API routes
- AI calls behind a service layer — API keys never in frontend
- AI responses: always structured JSON, not free text. Always have a fallback.
- Frontend: React functional components, mobile-first, lazy-load pages
- Styling: Tailwind CSS (default) or CSS Modules — check if project has preference
- Dark aesthetic, glass card UI — match Inner Lab design language
- All user_id fields reference the MBS Platform user ID (from JWT `userId` field)

---

## What NOT to Do
- Do NOT build your own login/signup — use MBS Platform SSO
- Do NOT build your own payment/billing — use MBS Platform
- Do NOT create a `users` collection — user identity lives in `mbs_platform`
- Do NOT write to other modules' prefixed collections (`cwg_*`, `yoga_*`)
- Do NOT read another module's unshared memories
- Do NOT hardcode your product slug — use env var `PRODUCT_SLUG`
- Do NOT duplicate the Inner Lab Middleware APIs — call them or read il_* directly

---

## Existing Inner Lab Modules (For Reference)

| Module | Slug | Prefix | Status | Notes |
|--------|------|--------|--------|-------|
| Conversations With God | `cwg` | `cwg_*` | Active | 56 collections, ~10 users, FastAPI (Python) |
| FlowState | `flowstate` | `yoga_*` | Active | 7 collections, Express (Node.js) |
| BreathArc | `breatharc` | `breath_*` | Coming Soon | |
| StarMap | `starmap` | `astro_*` | Coming Soon | |
| AstroCompass | `astrocompass` | `astrocart_*` | Coming Soon | |
| Arcana | `arcana` | `tarot_*` | Coming Soon | |
| Archetypes | `archetypes` | `archetype_*` | Coming Soon | |
| DreamLens | `dreamlens` | `dream_*` | Coming Soon | |
| Rituals | `rituals` | `ritual_*` | Coming Soon | |
| InnerQuest | `innerquest` | `quest_*` | Coming Soon | |
| Nexus | `nexus` | `nexus_*` | Coming Soon | |

---

## Phase 1 Learnings (Added by Orchestrator — 2026-03-26)

### JWT Details (as actually implemented)
- Header: `Authorization: Bearer <token>`
- Payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- `userId` is a **string** (ObjectId.toString()). Cast if needed: `new mongoose.Types.ObjectId(req.user.userId)`
- Token stored in `localStorage.getItem("mbs_token")`, user in `localStorage.getItem("mbs_user")`
- Token arrives via `?token=<JWT>` URL param after login redirect — extract, store, replaceState

### Entitlement Check
- `GET https://magicbusstudios.com/api/entitlements/{your_slug}` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }` — cache for 5 minutes

### GDPR
- MBS Platform cascade delete reaches into `inner_lab` database. All your {prefix}_* and il_* collections must use `user_id` field consistently (not `userId` or `user`).
