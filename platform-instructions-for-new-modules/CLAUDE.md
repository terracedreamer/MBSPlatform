# Inner Lab Module — Starter Template

> **How to use this file**: Copy this entire folder into your new module's project folder. Rename this file or merge it into your project's CLAUDE.md. Then add a section at the top describing what YOUR specific module does (name, purpose, features). Everything below is the shared context every Inner Lab module needs.

---

## You Are Building an Inner Lab Module

You are building a module within the **Inner Lab** ecosystem — a unified system for inner growth by Magic Bus Studios. Your module is one of up to 12 modules that share a database and user context.

**Your module does NOT handle:**
- Login/authentication — the Inner Lab login page at innerlab.ai/auth/login handles this (which calls MBS Platform auth APIs)
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
│  Handles: SSO (Google + Email/Password + Nostr + LNURL +     │
│           2FA/TOTP), billing (Stripe + BTCPay), entitlements  │
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
bonds_connections        — Bonds relationship data
bonds_reflections        — Bonds relationship reflections
lifemap_chapters         — LifeMap life chapters
astro_charts             — StarMap chart data
astro_transits           — StarMap transit calculations
```

**Choose a short prefix** (e.g., `bonds_`, `lifemap_`, `astro_`, `tarot_`, `dream_`, `ritual_`, `quest_`, `nexus_`). Never use `cwg_`, `yoga_`, or `il_` — those are taken.

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

### Module Domain Convention

**All new Inner Lab modules deploy to `{slug}.innerlab.ai`** with backend at `api.{slug}.innerlab.ai`.

Examples: `rituals.innerlab.ai`, `bonds.innerlab.ai`, `starmap.innerlab.ai`

Historical exceptions (pre-convention):
- CWG: `conversationswithgod.ai` (own domain)
- FlowState: `yoga.magicbusstudios.com`

Standalone products (Arcade, Studio Works) still use `*.magicbusstudios.com`.

**⚠️ IMPORTANT — don't confuse module domain with platform domain:**
- Your module's own domain → `{slug}.innerlab.ai` / `api.{slug}.innerlab.ai`
- `VITE_PLATFORM_URL` → `https://magicbusstudios.com` (subscribe/billing pages — this is MBS Platform, NOT your module)
- `PLATFORM_URL` → `https://api.magicbusstudios.com` (backend entitlement API — this is MBS Platform, NOT your module)

The `*.innerlab.ai` convention is about where YOUR MODULE lives. MBS Platform still lives at `magicbusstudios.com`. Do NOT change `VITE_PLATFORM_URL` or `PLATFORM_URL` to `innerlab.ai`.

### Login Flow
```
User visits {slug}.innerlab.ai
  → Clicks "Login"
  → Redirect to: innerlab.ai/auth/login?redirect=https://{slug}.innerlab.ai
  → User authenticates (Google SSO / Email+Password / Nostr / LNURL)
  → MBS Platform redirects back: https://{slug}.innerlab.ai?token={JWT}
  → Your frontend stores JWT, sends in Authorization header
  → Your backend validates JWT on every request
```

### Checking Entitlements

**Read `~/.claude/reference/entitlement-integration.md` for full patterns (frontend + backend).**

Before giving access to features, check with the platform:
```javascript
// Call from your backend
const response = await fetch(`${PLATFORM_URL}/api/entitlements/${YOUR_SLUG}`, {
  headers: { Authorization: `Bearer ${userToken}` }
});
const { hasAccess, isPremium, reason } = await response.json();
// hasAccess: true/false — does the user have ANY access?
// isPremium: true/false — is it premium or free tier?
// reason: "free_tier", "product_pass", "category_access", "mbs_all_access", "none"
```

Three states to handle:
- `hasAccess: false` → redirect to `magicbusstudios.com/subscribe/innerlab` (all IL modules use this dedicated page)
- `hasAccess: true, isPremium: false` → allow access, enforce module-defined free tier limits
- `hasAccess: true, isPremium: true` → full access, no limits

**MBS does NOT pass limits.** Your module defines its own free vs premium limits (e.g., daily caps, feature locks). Cache entitlement for 5 minutes, support `?refresh=true` to bust cache.

---

## Environment Variables

| Variable | Value | Notes |
|----------|-------|-------|
| MONGO_URL | (same as all MBS apps) | Shared MongoDB on Coolify |
| DB_NAME | `inner_lab` | Same DB as all Inner Lab modules |
| JWT_PUBLIC_KEY | PEM format (same as all services) | RS256 primary verification. Coolify: check "Is Multiline?" |
| JWT_SECRET | (same as MBS Platform) | HS256 fallback (dual-mode still active). Both keys required. |
| PORT | (unique per module) | Don't conflict with other modules |
| CORS_ORIGINS | `https://{slug}.innerlab.ai,https://www.{slug}.innerlab.ai` | Must include both www and non-www |
| PLATFORM_URL | `https://api.magicbusstudios.com` | For entitlement checks (backend-to-backend) |
| PRODUCT_SLUG | Your module's slug | e.g., `rituals`, `starmap` |
| SERVICE_NAME | `{slug}-api` | Used by Winston logger |

Optional (if your module uses AI):
| OPENAI_API_KEY | (shared MBS key) | Behind a service layer, never in frontend |
| OPENAI_MODEL | `gpt-4o-mini` | Default model |

---

## Project Structure (Recommended)

Follow the standard MBS two-container pattern. Most modules use a **TypeScript monorepo** with npm workspaces:
```
your-module/
├── frontend/               # React frontend (Vite + TypeScript)
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   ├── store/          # Zustand for local state
│   │   ├── hooks/
│   │   ├── lib/api.ts      # API client with JWT header
│   │   └── ...
│   ├── package.json        # workspace: "frontend"
│   └── tsconfig.json
├── backend/                # Express backend (TypeScript)
│   ├── src/
│   │   ├── server.ts       # Express entry
│   │   ├── routes/         # Module-specific routes
│   │   ├── middleware/      # requireAuth (JWT validation)
│   │   ├── services/       # AI service, module logic
│   │   └── config/
│   ├── package.json        # workspace: "backend"
│   └── tsconfig.json
├── Dockerfile.frontend     # Multi-stage: build → nginx
├── Dockerfile.backend      # Multi-stage: build (tsc) → runtime
├── nginx.conf
├── CLAUDE.md               # THIS FILE (customized for your module)
└── package.json            # Root: npm workspaces ["frontend", "backend"]
```

If using plain JavaScript instead of TypeScript, the structure is simpler (no `src/` subfolder, no tsconfig, no build step for backend).

---

## Dockerfile Gotchas (TypeScript Monorepo on Coolify)

> **Every TypeScript monorepo module has hit the same 3 deployment failures.** Read this section carefully before writing Dockerfiles.

### Gotcha 1: `npm ci --prefix backend` fails — lockfile not found

**Why**: npm workspaces keep `package-lock.json` at the monorepo **root**, not inside `backend/`. The `--prefix` flag looks for a lockfile inside `backend/` and fails.

**Fix**: Copy the root lockfile and use workspace install:
```dockerfile
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
RUN npm ci --workspace=backend
```

Or install everything from root (simpler but larger image):
```dockerfile
COPY package.json package-lock.json ./
COPY backend/ ./backend/
RUN npm ci
```

### Gotcha 2: `tsc: not found` — Coolify strips devDependencies

**Why**: Coolify injects `NODE_ENV=production` as a Docker build arg. When `NODE_ENV=production`, `npm install` and `npm ci` skip devDependencies — but `typescript` is a devDependency. The `tsc` compiler is never installed.

**Fix**: Override NODE_ENV during the **build stage** (the runtime stage should still be production):
```dockerfile
# Build stage — need devDeps for tsc
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
RUN NODE_ENV=development npm ci --workspace=backend
COPY backend/ ./backend/
RUN npx tsc -p backend/tsconfig.json

# Runtime stage — production only
FROM node:22-alpine
WORKDIR /app
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
RUN npm ci --workspace=backend --omit=dev
COPY --from=builder /app/backend/dist ./backend/dist
CMD ["node", "backend/dist/src/server.js"]
```

### Gotcha 3: `Cannot find module dist/server.js` — wrong output path

**Why**: If `tsconfig.json` has `rootDir: "."` (the default), TypeScript preserves the folder structure in output. So `backend/src/server.ts` compiles to `backend/dist/src/server.js`, NOT `backend/dist/server.js`.

**Fix** (choose one):
- **Option A** — Use the correct path in CMD: `CMD ["node", "backend/dist/src/server.js"]`
- **Option B** — Create a `tsconfig.build.json` with `rootDir: "src"` so output goes directly to `dist/server.js`:
  ```json
  {
    "extends": "./tsconfig.json",
    "compilerOptions": { "rootDir": "src" },
    "include": ["src"]
  }
  ```
  Then compile with: `RUN npx tsc -p backend/tsconfig.build.json`

### Complete Dockerfile.backend Example

```dockerfile
# ---- Build Stage ----
FROM node:22-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
COPY backend/package.json ./backend/

# Force development to install devDeps (typescript, etc.)
RUN NODE_ENV=development npm ci --workspace=backend

COPY backend/ ./backend/
RUN npx tsc -p backend/tsconfig.json

# ---- Runtime Stage ----
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production

COPY package.json package-lock.json ./
COPY backend/package.json ./backend/

# Production deps only
RUN npm ci --workspace=backend --omit=dev

COPY --from=builder /app/backend/dist ./backend/dist

EXPOSE 4000
USER node
HEALTHCHECK --interval=30s --timeout=5s CMD wget -q --spider http://localhost:4000/health || exit 1
CMD ["node", "backend/dist/src/server.js"]
```

**Adjust the CMD path** based on your tsconfig's `rootDir` setting (see Gotcha 3).

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
- Do NOT create a `users` collection — user identity lives in `mbs_platform`. If you DID build local auth before SSO integration, see the "Legacy User Collision" section in `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` — you must handle the case where existing local user `_id` values don't match the platform-assigned `userId`.
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
| Bonds | `bonds` | `bonds_*` | Coming Soon | Relationship mapping |
| LifeMap | `lifemap` | `lifemap_*` | Coming Soon | Life timeline & chapters |
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
- `GET https://api.magicbusstudios.com/api/entitlements/{your_slug}` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }` — cache for 5 minutes

### GDPR
- MBS Platform cascade delete reaches into `inner_lab` database. All your {prefix}_* and il_* collections must use `user_id` field consistently (not `userId` or `user`).
