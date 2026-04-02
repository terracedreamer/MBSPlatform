# Inner Lab Module Building Guide

**Version**: 1.0
**Date**: April 2, 2026
**Purpose**: Complete guide for building new Inner Lab modules AND enhancing the Inner Lab dashboard (innerlab.ai). Includes guardrails, operational setup, deployment patterns, and lessons learned. Written for an AI agent (Codex) that has never worked on this codebase.

---

## Table of Contents

1. [Before You Start](#1-before-you-start)
2. [Architecture You Must Understand](#2-architecture-you-must-understand)
3. [Authentication — How It Works](#3-authentication--how-it-works)
4. [Database — Rules and Patterns](#4-database--rules-and-patterns)
5. [Building a New Module](#5-building-a-new-module)
6. [Enhancing the Inner Lab Dashboard](#6-enhancing-the-inner-lab-dashboard)
7. [Deployment Setup](#7-deployment-setup)
8. [Coding Standards](#8-coding-standards)
9. [Guardrails — Things That Will Break If You Ignore Them](#9-guardrails--things-that-will-break-if-you-ignore-them)
10. [Lessons Learned from Building 17 Apps](#10-lessons-learned-from-building-17-apps)
11. [Reference Files](#11-reference-files)

---

## 1. Before You Start

### Read These Documents First
1. `MBS_Platform_Technical_Architecture.md` — Full architecture reference
2. `MBS_Database_Schema_Reference.md` — All MongoDB schemas
3. The project's own `CLAUDE.md` — Project-specific instructions override everything else

### What You Are NOT Building
- **Authentication backend** — MBS Platform (magicbusstudios.com) owns ALL auth. Your module has login PAGES that redirect to MBS Platform, or uses JWT validation middleware. You never issue tokens.
- **Billing/payments** — MBS Platform owns Stripe + BTCPay. Your module checks entitlements, it doesn't process payments.
- **User identity storage** — The `users` collection lives in `mbs_platform` database. Your module does NOT create a `users` collection.

---

## 2. Architecture You Must Understand

```
┌──────────────────────────────────────────────────────────────┐
│  LAYER 1: MBS Platform (magicbusstudios.com)                  │
│  Database: mbs_platform                                       │
│  Owns: Users, Auth, Billing, Entitlements, Friends            │
│  Signs JWTs with RS256 private key                            │
└──────────────┬────────────────────────────────────────────────┘
               │ JWT (RS256 signed, verified with public key)
┌──────────────▼────────────────────────────────────────────────┐
│  LAYER 2: Inner Lab Middleware (innerlab.ai)                   │
│  Database: inner_lab (shared with ALL modules below)           │
│  Owns: il_* shared collections                                │
│  Provides: Dashboard, consciousness, memories, check-ins      │
│  Frontend: innerlab.ai (marketing + auth pages + dashboard)   │
│  Backend: api.innerlab.ai                                     │
└──────────────┬────────────────────────────────────────────────┘
               │ Same inner_lab database
┌──────────────▼────────────────────────────────────────────────┐
│  YOUR MODULE (separate containers, same inner_lab database)    │
│  Owns: {prefix}_* collections                                 │
│  Reads: il_* shared collections for personalization            │
│  Writes: il_check_ins, il_user_memories, il_activity_feed     │
└───────────────────────────────────────────────────────────────┘
```

### Key Insight
All Inner Lab modules share the `inner_lab` database. They are separate Docker containers with their own backends, but they all connect to the same MongoDB database. Data isolation is achieved through **collection prefixes**, not separate databases.

---

## 3. Authentication — How It Works

### The Login Flow (End to End)
```
1. User visits your-module.magicbusstudios.com (or innerlab.ai)
2. User clicks "Sign In"
3. Redirect to: innerlab.ai/auth/login?redirect={your_domain}
   (Inner Lab modules use innerlab.ai login page, not magicbusstudios.com)
4. User authenticates (Google SSO / Email+Password / Nostr / LNURL)
5. MBS Platform issues JWT (RS256 signed)
6. Redirect back: {your_domain}?token={JWT}
7. Your frontend extracts token from URL, stores in localStorage as "mbs_token"
8. Your frontend sends token as: Authorization: Bearer {JWT}
9. Your backend verifies JWT with JWT_PUBLIC_KEY (RS256) or JWT_SECRET (HS256 fallback)
```

### JWT Payload (Exact Shape)
```json
{
  "userId": "67890abc...",  // String (ObjectId.toString()) — THIS IS THE UNIVERSAL USER ID
  "email": "user@example.com",  // String or null (Nostr/LNURL users have no email)
  "name": "User Name",
  "avatar": "https://..." ,  // String or null
  "isAdmin": false,
  "iat": 1711234567,
  "exp": 1711839367
}
```

### Auth Middleware for Express (Node.js) Modules
```javascript
const jwt = require('jsonwebtoken');

// Parse PEM key from env (Coolify may escape newlines)
const parsePemKey = (raw) => {
  if (!raw) return null;
  let key = raw;
  while (key.includes('\\n')) key = key.replace(/\\n/g, '\n');
  return key.trim();
};

function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split('Bearer ')[1];
  if (!token) return res.status(401).json({ success: false, message: 'No token' });

  try {
    // Try RS256 first (recommended)
    const publicKey = parsePemKey(process.env.JWT_PUBLIC_KEY);
    if (publicKey) {
      try {
        req.user = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
        return next();
      } catch (e) {
        if (e.name === 'TokenExpiredError') throw e;
        // Fall through to HS256
      }
    }
    // Fallback: HS256 with shared secret
    req.user = jwt.verify(token, process.env.JWT_SECRET, { algorithms: ['HS256'] });
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ success: false, message: 'Token expired' });
    }
    return res.status(401).json({ success: false, message: 'Invalid token' });
  }
}
```

### Auth Middleware for FastAPI (Python) Modules
```python
import jwt
from fastapi import Depends, HTTPException, Request

def get_public_key():
    raw = os.getenv("JWT_PUBLIC_KEY", "")
    return raw.replace("\\n", "\n").strip() if raw else None

def get_current_user(request: Request):
    auth = request.headers.get("Authorization", "")
    if not auth.startswith("Bearer "):
        raise HTTPException(401, "No token")
    token = auth.split(" ", 1)[1]

    public_key = get_public_key()
    if public_key:
        try:
            return jwt.decode(token, public_key, algorithms=["RS256"])
        except jwt.ExpiredSignatureError:
            raise HTTPException(401, "Token expired")
        except jwt.InvalidTokenError:
            pass  # Fall through to HS256

    try:
        return jwt.decode(token, os.getenv("JWT_SECRET"), algorithms=["HS256"])
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")
```

### Checking Entitlements
```javascript
// Call from your backend to check if user has premium access
const response = await fetch(
  `${process.env.PLATFORM_URL}/api/entitlements/${process.env.PRODUCT_SLUG}`,
  { headers: { Authorization: `Bearer ${userToken}` } }
);
const { hasAccess, reason } = await response.json();
// reason: "free_tier" | "product_pass" | "category_access" | "mbs_all_access" | "no_subscription"
```

**Cache entitlement responses** for 5 minutes in-memory (keyed by `userId:productSlug`).

**Fail-open**: If the platform API is unreachable, default to allowing access (free_tier). This prevents platform downtime from breaking every product.

### Premium Gating Pattern (effectivePremium)
Currently, no product enforces premium gating. Use this pattern to wire it up but keep everything accessible:
```javascript
// In your route handler:
const effectivePremium = true; // Toggle to false when platform enables gating
if (!effectivePremium && !hasAccess) {
  return res.status(403).json({ success: false, message: 'Premium required' });
}
```

---

## 4. Database — Rules and Patterns

### Connection
All Inner Lab modules connect to the SAME database:
```
MONGO_URL=mongodb://user:pass@host:27017/?authSource=admin
DB_NAME=inner_lab
```

### Your Collections
Choose a short, unique prefix. Create your collections under this prefix:
```
breath_sessions        — BreathArc
breath_tracks
breath_preferences
astro_charts           — StarMap
astro_transits
```

**Taken prefixes**: `cwg_`, `yoga_`, `il_` — never use these.

### Writing to Shared il_* Collections
Your module CAN and SHOULD write to these shared collections when appropriate:

| Collection | When to Write | Example |
|-----------|--------------|---------|
| `il_check_ins` | When your module captures mood/energy data | After a meditation session |
| `il_user_memories` | When your AI learns a fact about the user | "User is exploring grief" |
| `il_activity_feed` | When a significant event happens | "Completed 30-min breathwork session" |

**Always set `source_module` to your module's slug** (e.g., "breatharc").

### Reading Shared il_* Collections
You CAN read from all il_* collections to personalize your module:
- Read `il_consciousness_profiles` to tailor AI responses to user's archetype
- Read `il_user_wellness_profiles` for health/injury awareness
- Read `il_user_memories` WHERE `shared: true` for cross-module context
- **NEVER read memories where `shared: false` and `source_module` is not yours**

### user_id Mapping (CRITICAL)
```
JWT field:     userId    (camelCase, String)
Database field: user_id  (snake_case, String)

// In your code:
const user_id = req.user.userId;  // Maps JWT userId to DB user_id
```

ALL il_* collections use `user_id` (snake_case, String). This is non-negotiable — the GDPR cascade depends on this field name to find and delete user data.

---

## 5. Building a New Module

### Step 1: Register with MBS Platform
Your module MUST be added to `MBS/server/config/products.js`:
```javascript
{
  slug: "breatharc",
  name: "BreathArc",
  category: "innerlab",
  freeTier: true,
  freeTierLimits: { sessionsPerDay: 3 },
  premiumFeatures: ["unlimited_sessions", "custom_patterns", "session_history"],
  stripePricePrefix: "[IL]",
  url: "https://breatharc.magicbusstudios.com",  // or null if not deployed
  apiUrl: "https://api.breatharc.magicbusstudios.com",
}
```
Without this, entitlement checks and SSO redirects will not work.

### Step 2: Project Structure
```
your-module/
├── src/                    # React frontend (Vite)
│   ├── pages/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   │   └── api.js          # Axios/fetch with JWT header
│   └── index.jsx
├── server/
│   ├── index.js            # Express entry
│   ├── routes/             # Module-specific routes
│   ├── middleware/
│   │   └── auth.js         # JWT validation (copy pattern from Section 3)
│   ├── models/             # Mongoose models for {prefix}_* collections
│   ├── services/           # AI service layer, business logic
│   ├── utils/
│   │   └── logger.js       # Winston logger
│   └── config/
│       └── database.js     # MongoDB connection
├── Dockerfile              # Frontend (multi-stage: build + nginx)
├── Dockerfile.server       # Backend (Node.js or Python)
│   OR server/Dockerfile
├── nginx.conf              # SPA routing
├── CLAUDE.md               # Project instructions
├── .env.example            # Document ALL env vars
└── package.json
```

### Step 3: Required Endpoints
Every module MUST implement:

| Endpoint | Purpose | Notes |
|----------|---------|-------|
| `GET /health` or `GET /api/health` | Health check | Returns `{ status: "ok" }` |
| `DELETE /api/user-data` | GDPR data deletion | Delete ALL user data from your {prefix}_* collections. Auth-gated. |

### Step 4: Frontend Auth Integration
```javascript
// On app load, check for ?token= parameter (SSO redirect callback)
useEffect(() => {
  const params = new URLSearchParams(window.location.search);
  const token = params.get('token');
  if (token) {
    localStorage.setItem('mbs_token', token);
    // Fetch user profile
    fetch(`${PLATFORM_URL}/api/auth/me`, {
      headers: { Authorization: `Bearer ${token}` }
    }).then(r => r.json()).then(data => {
      localStorage.setItem('mbs_user', JSON.stringify(data.user));
    });
    // Clean URL
    window.history.replaceState({}, '', window.location.pathname);
  }
}, []);

// Login redirect (when user clicks "Sign In")
const loginUrl = `https://innerlab.ai/auth/login?redirect=${encodeURIComponent(window.location.origin)}`;
window.location.href = loginUrl;
```

### Step 5: Environment Variables
| Variable | Purpose | Required |
|----------|---------|----------|
| `MONGO_URL` | MongoDB connection string | Yes |
| `DB_NAME` | `inner_lab` | Yes |
| `JWT_PUBLIC_KEY` | RSA public key for JWT verification (PEM) | Yes |
| `JWT_SECRET` | HS256 fallback secret | Yes |
| `PORT` | Server port | Yes |
| `CORS_ORIGINS` | Your frontend domain(s) | Yes |
| `PLATFORM_URL` | `https://magicbusstudios.com` | Yes |
| `PRODUCT_SLUG` | Your module's slug | Yes |
| `SERVICE_NAME` | For Winston logger | Recommended |
| `OPENAI_API_KEY` | If your module uses AI | Conditional |
| `VITE_BACKEND_URL` | Backend URL (frontend build arg) | Yes |
| `VITE_PLATFORM_URL` | `https://magicbusstudios.com` (frontend build arg) | Yes |
| `VITE_GOOGLE_CLIENT_ID` | Google OAuth client ID (frontend build arg) | Yes |

---

## 6. Enhancing the Inner Lab Dashboard

### Current State (April 2026)
The Inner Lab dashboard at innerlab.ai is live with:
- **Auth pages**: Login, signup, forgot/reset password (calling MBS Platform APIs)
- **Dashboard page**: Module launcher, check-in widget, activity feed
- **Consciousness page**: Profile viewer/editor
- **Memories page**: Memory list with sharing controls
- **Activity page**: Cross-module activity feed
- **Backend**: Express at api.innerlab.ai with all il_* API routes

### What the Dashboard Should Become
Inner Lab is the **star** — users should log in to innerlab.ai and get a unified view of their inner growth journey across all modules. Currently, it only has data from CWG (the only module that writes to il_* collections). As more modules come online, the dashboard becomes more powerful.

### Dashboard Enhancement Priorities
1. **Daily Briefing** — Synthesized summary from check-ins, memories, module activity. Even with only CWG data, this can surface patterns like "You've had 3 conversations about grief this week."
2. **Cross-Module Insights** — Pattern detection across modules. Initially simple (activity frequency, mood trends), becoming smarter as data grows.
3. **Module Launcher Enhancement** — Show per-module stats (last session, streak, recent activity) rather than just links.
4. **Consciousness Profile Deepening** — Integration with CWG assessment data, timeline of archetype evolution.
5. **Unified Search** — Search across memories, journal entries, activity feed.

### Dashboard Architecture
```
innerlab.ai (frontend)
  ├── Marketing pages (public)
  ├── Auth pages (call MBS Platform APIs at magicbusstudios.com)
  └── Dashboard pages (call Inner Lab middleware at api.innerlab.ai)

api.innerlab.ai (backend)
  ├── il_* CRUD routes (check-ins, consciousness, memories, activity, etc.)
  ├── Forms routes (contact, subscribe, waitlist → SendGrid)
  └── GDPR route (DELETE /api/user-data)
```

### Key Env Vars for Dashboard
```
VITE_BACKEND_URL = https://api.innerlab.ai        # Dashboard API calls
VITE_API_URL = https://api.magicbusstudios.com     # Form submissions
VITE_PLATFORM_URL = https://magicbusstudios.com    # Auth API calls
VITE_GOOGLE_CLIENT_ID = {google_client_id}         # Google SSO on auth pages
```

---

## 7. Deployment Setup

### Where Things Run
Everything is deployed on **Coolify** (self-hosted VPS). Coolify manages Docker containers with Traefik as the reverse proxy.

### Container Pattern
Most apps deploy as **2 containers**:
1. **Frontend**: Multi-stage Docker build (Node.js builds Vite app → nginx serves static files)
2. **Backend**: Node.js (Express) or Python (FastAPI) container

Some simpler apps use a **single container** (Express serves both API + frontend static files).

### Deployment Flow
```
1. Push code to GitHub (branch: main, test, or development — depends on app)
2. Coolify detects push via webhook
3. Coolify builds container(s) using Dockerfile or Nixpacks
4. Traefik routes traffic to the new container
```

### Coolify Specifics
- **Build args** (VITE_*) must be set as Coolify build args, NOT runtime env vars. Vite bakes them in at build time.
- **Runtime env vars** (MONGO_URL, JWT_SECRET, etc.) are set as regular env vars in Coolify.
- **Multiline env vars** (JWT_PUBLIC_KEY, JWT_PRIVATE_KEY): Must check "Is Multiline?" in Coolify.
- **WebSocket apps**: Must enable WebSocket support in Coolify proxy settings.

### Domain Pattern
```
Frontend:  {slug}.magicbusstudios.com     (preferred)
           {slug}.innerlab.ai              (future for IL modules)
Backend:   api.{slug}.magicbusstudios.com  (preferred — dot-separated)
```

### Dockerfile Template (Frontend)
```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
ARG VITE_BACKEND_URL
ARG VITE_PLATFORM_URL
ARG VITE_GOOGLE_CLIENT_ID
ARG VITE_PRODUCT_SLUG
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Dockerfile Template (Backend — Express)
```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 5000
HEALTHCHECK --interval=30s --timeout=5s CMD wget -q --spider http://localhost:5000/health || exit 1
USER node
CMD ["node", "index.js"]
```

### nginx.conf Template (SPA)
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 8. Coding Standards

### Backend (Express — Node.js)
- **Logger**: Winston — NEVER use `console.log`. Import from `utils/logger.js`.
- **Response format**: Always `{ success: true, ...data }` or `{ success: false, message: "..." }`
- **Route pattern**: `authenticate → validate → business logic → respond`
- **Error handler**: Centralized middleware. Never catch errors in individual routes unless you have route-specific recovery.
- **Input validation**: On ALL routes. Use express-validator or manual validation.
- **Rate limiting**: On ALL API routes.
- **CORS**: Must come BEFORE helmet() middleware.
- **AI calls**: Always behind a service layer. API keys NEVER in frontend code.

### Backend (FastAPI — Python)
- **Logger**: Python logging module with structured output.
- **Response format**: Same `{ success: true/false }` pattern via response helpers.
- **Auth**: Via dependency injection (`Depends(get_current_user)`).
- **Database**: `get_database()` is NOT async. Motor for async DB operations.
- **DB fields**: `snake_case` in MongoDB. camelCase middleware converts for frontend.
- **Dates**: Always timezone-aware (`datetime.now(timezone.utc)`, never `datetime.utcnow()`).
- **Always**: `python -m py_compile` before committing.

### Frontend (React)
- **Build tool**: Vite (never CRA). File extensions: `.jsx` for files with JSX.
- **Styling**: Tailwind CSS (default). Exception: some projects use CSS Modules.
- **Toasts**: Sonner (`import { toast } from 'sonner'`). NEVER react-hot-toast.
- **Array safety**: Always `(array || []).map()` to prevent undefined crashes.
- **Auth pages**: Google SSO button ABOVE email/password form.
- **Mobile-first**: All layouts responsive.
- **Code splitting**: Use `React.lazy()` for route-level splitting.
- **Fonts**: Space Grotesk (headings), DM Sans (body), via @fontsource-variable.
- **Dark theme**: `--bg-primary: #0f0f13`, glass card panels, Framer Motion animations.
- **Animations**: framer-motion ONLY — never CSS @keyframes for entrance/exit/hover effects.

---

## 9. Guardrails — Things That Will Break If You Ignore Them

### WILL CAUSE AUTH FAILURES
1. **Never create your own auth routes** (login, signup, token issuance). All auth goes through MBS Platform.
2. **JWT_PUBLIC_KEY must have real newlines**, not literal `\n`. The `parsePemKey` helper handles Coolify's escaping.
3. **JWT_SECRET must be identical across ALL services**. One mismatch = 401 for every user.
4. **`authSource=admin` in MONGO_URL** when using root MongoDB credentials. Without it, auth fails silently.

### WILL CAUSE DATA CORRUPTION
5. **Never write to another module's prefix** (e.g., a BreathArc module writing to `cwg_*` collections).
6. **Always use `user_id` (snake_case, String)** in il_* collections. Using `userId` (camelCase) or ObjectId will break GDPR cascade deletion.
7. **Never create a `users` collection** in the `inner_lab` database. User identity lives only in `mbs_platform`.

### WILL CAUSE DEPLOYMENT FAILURES
8. **VITE_* env vars are build-time only** — they must be Docker build args in Coolify, NOT runtime env vars.
9. **Coolify env vars must be on separate lines** — concatenated vars (e.g., `JWT_SECRET=xxxLOG_LEVEL=INFO`) cause silent failures.
10. **CORS_ORIGINS must include both www and non-www domains** and must be registered BEFORE helmet() middleware.

### WILL CAUSE GDPR VIOLATIONS
11. **Every module MUST implement `DELETE /api/user-data`** — called by MBS Platform during account deletion cascade.
12. **ConsentAuditLog is NEVER deleted** — it lives in mbs_platform and is anonymized, not deleted.
13. **Use `user_id` field consistently** across ALL collections — the cascade deletes by this field.

### WILL CAUSE CONFUSION
14. **Never say "calm"** in marketing copy — brand guideline.
15. **Never say "Operating System"** — Inner Lab is a "system" or "platform", not an OS.
16. **Never call Inner Lab a "suite of apps"** — it's "one unified system" with modules.
17. **Never expose technical terms to users**: "SSO", "middleware", "platform", "API", "JWT".

---

## 10. Lessons Learned from Building 17 Apps

### Auth Lessons
- **Legacy user collision**: When migrating existing apps to SSO, users who signed up locally have different `_id` values than their MBS Platform `userId`. Fix: email-based lookup → delete old record → recreate with platform ID → update all related collections.
- **RS256 rollout**: Done in phases. Phase 1: dual-mode (RS256 first, HS256 fallback). Phase 2 removal attempted but reverted because not all services had redeployed. HS256 fallback is still active.
- **Google One Tap**: Requires the domain to be added to Google Cloud Console's authorized JavaScript origins. Forgot this = login silently fails.

### Database Lessons
- **MongoDB transactions require replica set**: TaskTracker's legacy user migration uses transactions. If running on standalone MongoDB, transactions fail. Self-hosted MongoDB must be configured as a replica set.
- **`authSource=admin` is required** when using root credentials. Without it, connection succeeds but queries return empty results.
- **TTL indexes**: Used for auto-expiring documents (auth challenges, analytics events). MongoDB checks every 60 seconds — don't rely on instant expiry.

### Deployment Lessons
- **Coolify Docker networks are separate per service**: Internal Docker hostnames (e.g., `backend:3001`) DON'T resolve between services. Always use public domains for inter-service communication.
- **Traefik may strip `/api` prefix**: If routes return 404, check whether Traefik is stripping the prefix. Mount routes without `/api` if so.
- **Frontend healthcheck can kill containers**: Coolify's default health probe may not reach nginx in all configurations. Disable frontend healthcheck if container keeps restarting.
- **OneDrive dehydration**: Working on repos synced by OneDrive can dehydrate `.git/objects/pack` files. Fix: `attrib -U +P .git /S /D` on Windows.

### Frontend Lessons
- **Vite requires `.jsx` extension** for files containing JSX syntax. Vite's esbuild pipeline doesn't transform JSX in `.js` files. This was discovered during the TaskTracker CRA→Vite migration (Session 14).
- **`VITE_BACKEND_URL` must NOT include `/api`** if Traefik strips the prefix.
- **Lenis smooth scroll** prevents screenshot tools from settling — use `preview_inspect` instead of `preview_screenshot` for verification.

### Python-Specific Lessons
- **PyJWT vs jwt**: The import is `import jwt` but the package is `PyJWT`. Installing `jwt` (different package) causes cryptic import errors.
- **camelCase middleware**: Must exclude auth endpoints — stripping `Set-Cookie` headers breaks login.
- **`model_config = SettingsConfigDict(env_file=".env")`** — NOT the deprecated `Config` inner class.

---

## 11. Reference Files

### Architecture Documents (this folder)
- `MBS_Platform_Technical_Architecture.md` — Complete technical architecture
- `MBS_Platform_Overview.md` — Non-technical overview
- `MBS_Database_Schema_Reference.md` — All MongoDB schemas
- `Inner_Lab_Module_Building_Guide.md` — This file

### Global Configuration
- `~/.claude/CLAUDE.md` — Global coding standards for all MBS projects
- `~/.claude/rules/env-standards.md` — Canonical env var names
- `~/.claude/rules/deployment-quirks.md` — Per-product deployment exceptions
- `~/.claude/rules/innerlab-data.md` — Cross-module data sharing rules
- `~/.claude/rules/data-sovereignty.md` — GDPR and data ownership principles

### Build Instructions (in MBSPlatform repo)
- `platform-instructions-for-innerlab/CLAUDE.md` — Inner Lab middleware build spec
- `platform-instructions-for-new-modules/CLAUDE.md` — New module starter template
- `platform-instructions-for-mbs/CLAUDE.md` — MBS Platform build spec

### Live Code References
- `MBS/server/middleware/auth.js` — The canonical auth middleware (RS256 + HS256 fallback)
- `MBS/server/config/products.js` — Product catalog (all 22 products)
- `Innerlab/server/models/` — All il_* Mongoose models (source of truth for schemas)
- `Innerlab/server/routes/` — All Inner Lab middleware API routes
