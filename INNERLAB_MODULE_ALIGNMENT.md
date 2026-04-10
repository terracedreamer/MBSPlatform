# Inner Lab Module — Visual & Standards Alignment Prompt

> **Purpose**: Give this prompt to any agent working on an Inner Lab module to bring it up to production standards. This covers visual alignment, entitlement gating, backend standards, GDPR compliance, and deployment readiness.
>
> **How to use**: Copy this entire prompt into a new Claude Code session for the module. Replace `{MODULE_NAME}`, `{module_slug}`, and `{prefix}` with the module's actual values.

---

## Visual & Standards Alignment — Match Inner Lab Production Quality

This module needs to be brought up to the same visual and code standards as the live Inner Lab dashboard (innerlab.ai) and other production MBS applications.

### Reference: Read these first

**Inner Lab dashboard** (the target quality to match):
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\index.css` — design tokens, Tailwind theme, glass-morphism values
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\components\ui\` — PageShell, PageHeader, Card, EmptyState, Skeleton
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\components\DashboardLayout.jsx` — layout pattern
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\components\Sidebar.jsx` — nav pattern
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\lib\dashboardApi.js` — API client pattern (centralized fetch with auth headers)
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\lib\motion.js` — animation presets (if it exists)

**CWG** (gold standard live module — read for complete working example):
- `C:\Users\1984a\OneDrive\Desktop\Codes\CWG\frontend\src\` — full module frontend structure

**This module**:
- Read this module's CLAUDE.md for module-specific context
- Read this module's SESSION_HANDOFF.md for current state

**Global references** (read before entitlement/GDPR work):
- `~/.claude/reference/entitlement-integration.md` — entitlement gating patterns (MANDATORY)
- `~/.claude/reference/innerlab-data.md` — collection prefixes, sharing rules
- `~/.claude/reference/data-sovereignty.md` — GDPR deletion scoping

---

### Phase A: Visual Alignment

#### 1. Migrate from plain CSS to Tailwind

- Replace `frontend/src/styles/global.css` with Tailwind setup (`@import "tailwindcss"` + `@theme` block)
- Install `tailwindcss` + `@tailwindcss/vite` as devDeps, update `vite.config.ts`
- Module can have ONE accent color for identity, but base chrome must match IL dashboard

**Concrete design tokens** (copy from IL dashboard `index.css`):

```
Backgrounds:   #050505 (dark-950), #0a0a0a (dark-900), #111111 (dark-850), #1a1a1a (dark-800)
Borders:        rgba(255,255,255,0.06) (default), rgba(255,255,255,0.1) (hover)
Text:           #e5e5e5 (primary), #aaaaaa (secondary), #666666 (muted)
Accent:         teal #14b8a6, sky #0ea5e9
Glass panels:   bg-white/[0.03], backdrop-blur-xl, border border-white/[0.06]
Glass hover:    bg-white/[0.06], border-white/[0.1]
Glow effects:   box-shadow: 0 0 40px rgba(20,184,166,0.08)
Scrollbar:      6px wide, #333 thumb, transparent track
```

#### 2. Add shared UI component patterns

Use `.tsx` if the module is TypeScript, `.jsx` if JavaScript. Match whichever the module already uses.

- Create `components/ui/PageShell` — consistent page wrapper with entrance animation (`py-6 px-4 md:px-8 max-w-3xl mx-auto space-y-6`)
- Create `components/ui/PageHeader` — icon + title + description + optional actions
- Create `components/ui/Card` — glass card with variants: default, accent, elevated, subtle
- Create `components/ui/EmptyState` — decorative glow + title + description + CTA
- Create `components/ui/Skeleton` — shimmer loading placeholders (replace all "Loading..." text)
- Add `ErrorBoundary` wrapping the main content outlet

**shadcn/ui**: Available in the MBS stack but IL dashboard uses custom glass components. Use shadcn/ui for utility components (Dialog, Dropdown, Tooltip) if needed, but primary cards/panels should be custom glass components matching IL.

#### 3. API client & data fetching

**Do NOT install React Query.** IL dashboard uses a centralized API client with plain fetch — match that pattern for consistency.

Create a centralized API client (e.g., `src/lib/api.js`):
```javascript
const API_URL = import.meta.env.VITE_BACKEND_URL || "";

async function apiFetch(path, options = {}) {
  const token = localStorage.getItem("mbs_token");
  const res = await fetch(`${API_URL}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });
  if (res.status === 401) {
    // Token expired — redirect to login
    localStorage.removeItem("mbs_token");
    localStorage.removeItem("mbs_user");
    window.location.href = `https://innerlab.ai/auth/login?redirect=${encodeURIComponent(window.location.origin)}`;
    return;
  }
  const data = await res.json();
  if (!res.ok) throw new Error(data.message || "Request failed");
  return data;
}

export const api = {
  get: (path) => apiFetch(path),
  post: (path, body) => apiFetch(path, { method: "POST", body: JSON.stringify(body) }),
  put: (path, body) => apiFetch(path, { method: "PUT", body: JSON.stringify(body) }),
  delete: (path) => apiFetch(path, { method: "DELETE" }),
};
```

All pages use this client — never raw `fetch` with manual auth headers.

#### 4. Animation presets

Use consistent Framer Motion presets across all pages:
```javascript
// src/lib/motion.js
export const fadeUp = {
  initial: { opacity: 0, y: 8 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.25, ease: [0.22, 1, 0.36, 1] },
};

export const stagger = {
  animate: { transition: { staggerChildren: 0.04 } },
};

export const fadeIn = {
  initial: { opacity: 0 },
  animate: { opacity: 1 },
  transition: { duration: 0.2 },
};
```

If IL dashboard has a `src/lib/motion.js`, copy those exact values instead.

#### 5. Polish every page

- Skeleton loading states (not text spinners)
- Empty states with helpful CTAs (not just "No data" text)
- Error states with retry buttons
- Hover micro-interactions on cards and buttons (scale, glow, border highlight)
- Consistent spacing (8px grid)
- Mobile responsive at 640px, 768px, 1024px breakpoints
- Custom scrollbar (thin, dark, matching theme — see design tokens above)
- Use `(array || []).map()` for all array renders — never bare `.map()` on potentially null data
- NEVER use `alert()`, `prompt()`, or `confirm()` — use Sonner toasts or modal components

**Toast pattern**:
```javascript
import { toast } from "sonner";
toast.success("Saved successfully");
toast.error("Failed to save — please try again");
```

**Form inputs** — consistent styling:
```
bg-white/[0.05] border border-white/[0.08] rounded-lg px-3 py-2 text-white text-sm
placeholder-white/20 focus:border-teal-500/40 focus:outline-none
```

#### 6. Typography alignment

- Headings: Space Grotesk (600/700 weight)
- Body: DM Sans (400/500)
- Display/accent: Instrument Serif italic
- Mono (if code): JetBrains Mono
- Load via Google Fonts with `display=swap`

#### 7. Navigation pattern

- Sidebar on desktop (collapsible, with icon-only compact mode)
- Bottom nav or slide-out drawer on mobile
- Active route highlighting with teal accent
- Module identity (icon + name) at top of sidebar

---

### Phase B: Auth & Entitlement Integration

#### 8. Auth & login flow

This module does NOT handle login itself — it validates JWTs issued by MBS Platform.

**Login redirect flow** (cross-domain):
```
1. User visits {module_domain} → no token in localStorage
2. Redirect to: https://innerlab.ai/auth/login?redirect={module_domain}
3. User authenticates (Google SSO / Email+Password / Nostr / LNURL)
4. MBS Platform issues JWT, redirects to: {module_domain}?token={JWT}
5. Module frontend detects ?token= param:
   - Extract token from URL
   - Store: localStorage.setItem("mbs_token", token)
   - Decode payload for user info: localStorage.setItem("mbs_user", JSON.stringify(decoded))
   - Clean URL: window.history.replaceState({}, "", window.location.pathname)
6. All subsequent API calls include: Authorization: Bearer {token}
```

**Token extraction** (put in App.tsx/App.jsx on mount):
```javascript
useEffect(() => {
  const params = new URLSearchParams(window.location.search);
  const token = params.get("token");
  if (token) {
    localStorage.setItem("mbs_token", token);
    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      localStorage.setItem("mbs_user", JSON.stringify(payload));
    } catch {}
    window.history.replaceState({}, "", window.location.pathname);
  }
}, []);
```

- Google SSO button ABOVE email/password option
- MBS branding link in footer

#### 9. Entitlement integration (free vs premium vs unsubscribed)

**Read `~/.claude/reference/entitlement-integration.md` before implementing this section.**

Every module must handle three user states:

| State | API Response | What to do |
|-------|-------------|------------|
| **Not registered** | `{ hasAccess: false }` | Redirect to `https://magicbusstudios.com/subscribe/innerlab` |
| **Free tier** | `{ hasAccess: true, isPremium: false }` | Allow access. Enforce this module's own free tier limits. |
| **Premium** | `{ hasAccess: true, isPremium: true }` | Full access. No limits. |

**Backend** (server-to-server): Call MBS Platform to check access:
```
GET ${PLATFORM_URL}/api/entitlements/{module_slug}
// PLATFORM_URL = https://api.magicbusstudios.com (backend env var)
Authorization: Bearer <user's JWT>
```

**Frontend** (browser): Call via the nginx proxy:
```
GET ${VITE_PLATFORM_URL}/api/entitlements/{module_slug}
// VITE_PLATFORM_URL = https://magicbusstudios.com (nginx proxies /api/* to backend)
Authorization: Bearer <mbs_token from localStorage>
```

Response (same from either path): `{ hasAccess: true/false, isPremium: true/false, reason: "free_tier|product_pass|category_access|mbs_all_access" }`

**Frontend**: AuthContext must expose `hasAccess` and `isPremium`:
- No access → redirect to `https://magicbusstudios.com/subscribe/innerlab`
- Free tier → show full UI, enforce module-specific limits
- Premium → no limits

**Critical rules**:
- MBS does NOT pass any limit numbers — only `hasAccess` and `isPremium` booleans
- Each module defines its OWN free vs premium limits (e.g., messages/day, saved items, AI calls/day). Work with the module owner to define these.
- Cache entitlement response for 5 minutes. Support `?refresh=true` query param to bust cache (MBS redirects back with this after subscribe/upgrade).
- Admin users (`isAdmin: true` in JWT) always get full access — skip entitlement checks.
- Use `effectivePremium = true` pattern during development — hardcode to true until MBS premium enforcement is live, so the module works without gating. Leave a clear `// TODO: remove effectivePremium when MBS free tier is live` comment.

**Downgrade handling**: When a premium user cancels, MBS auto-downgrades them to `free_tier`. The next entitlement check returns `isPremium: false`. Do NOT redirect to subscribe — they still have access. Lock premium features, keep free features working. Optionally show: "Your premium access has ended. You still have free access."

---

### Phase C: Backend Standards & GDPR

#### 10. Backend standards

- **Response format**: `{ success: true, ...data }` or `{ success: false, message: "..." }`
- **Logging**: Winston logger with `SERVICE_NAME` env var (NEVER `console.log`)
- **`req.user` shape**: `{ userId, email, name, avatar, isAdmin }` — always use `userId` (never `id` or `_id`)
- **Route pattern**: authenticate → validate → business logic → respond
- **AI calls**: Always behind a service layer — API keys (OPENAI_API_KEY) never in frontend
- **Input validation**: express-validator on all routes
- **Rate limiting**: express-rate-limit on API routes
- **CORS**: `cors()` BEFORE `helmet()` in middleware order. Include both `www` and non-`www` in `CORS_ORIGINS`.
- **Trust proxy**: `app.set("trust proxy", 1)` as the FIRST thing after `const app = express()` — BEFORE any middleware. Required for Coolify/Traefik deployment. Without it, express-rate-limit crashes with `ERR_ERL_UNEXPECTED_X_FORWARDED_FOR` on every request.

#### 11. GDPR compliance

- Module MUST expose: `DELETE /api/user-data` (protected by auth middleware)
- This endpoint deletes ALL data for the authenticated user from THIS module's database only
- For module-prefixed collections (`{prefix}_*`): delete all documents where `user_id` matches
- For shared `il_*` collections: delete entries WHERE `source_module = "{module_slug}"` AND `user_id` matches
- Do NOT delete identity singletons (`il_consciousness_profiles`, `il_personal_histories`, `il_consciousness_snapshots`, `il_user_wellness_profiles`, `il_birth_profiles`) — these are shared across all modules and only deleted at category-level or higher
- Returns `{ success: true, message: "All {MODULE_NAME} data deleted" }`
- MBS Platform calls this endpoint during category-level and full-account deletion cascades

#### 12. Inner Lab data layer

- **Database**: `inner_lab` (shared with all IL modules). Set `DB_NAME=inner_lab`.
- **Module collections**: use `{prefix}_*` naming (e.g., `dream_entries`, `bonds_connections`)
- **Shared IL collections**: `il_check_ins`, `il_user_memories`, `il_activity_feed`, `il_reflections` — you can READ all `il_*` and WRITE to these four with `source_module: "{module_slug}"`
- **NEVER write** to another module's prefix (`cwg_*`, `yoga_*`, etc.)
- **NEVER write** to identity singletons (`il_consciousness_profiles`, etc.) — only Inner Lab Middleware writes these
- **Journal entries**: `il_reflections` is the ONLY journal store — do NOT create module-local journal collections
- **User memories**: Write to `il_user_memories` with `shared: false` (private to this module until user opts in)
- **userId mapping**: JWT uses `userId` (camelCase) but all `il_*` documents use `user_id` (snake_case). Map: `user_id: req.user.userId`
- Read `~/.claude/reference/innerlab-data.md` for full collection prefix rules and sharing toggle details

#### 13. Testing

- Backend tests required. Express: Jest + supertest (or Vitest for TypeScript modules).
- Test auth flow: valid token, invalid token, expired token, missing token.
- Test GDPR endpoint: `DELETE /api/user-data` — verify it deletes correct data and returns success.
- Test entitlement-gated routes if applicable.
- **Jest + ESM on Windows**: `npx --node-options=--experimental-vm-modules jest`

---

### Phase D: Deployment Readiness

#### 14. Environment variables (canonical names)

**PLATFORM_URL dual-use pattern** (important — these are intentionally different):
- **Backend** `PLATFORM_URL` = `https://api.magicbusstudios.com` — server-to-server API calls (entitlement checks, etc.)
- **Frontend** `VITE_PLATFORM_URL` = `https://magicbusstudios.com` — used for BOTH API calls (nginx proxies `/api/*` to the backend) AND user-facing links (`/subscribe/*`, `/auth/*`). The frontend nginx config on magicbusstudios.com proxies `/api/*` requests to the backend, so both API calls and UI links work from the same origin.

**Backend** (runtime):

| Variable | Value | Notes |
|----------|-------|-------|
| `MONGO_URL` | (shared MongoDB connection) | NEVER `MONGODB_URI` or `MONGO_URI` |
| `DB_NAME` | `inner_lab` | Same DB as all Inner Lab modules |
| `JWT_SECRET` | (same as MBS Platform) | Must match for JWT validation |
| `JWT_PUBLIC_KEY` | (MBS Platform public key) | RS256 verification. PEM format. |
| `PORT` | (unique per module) | Default 4000 for new IL modules |
| `CORS_ORIGINS` | Frontend domain(s) | Include both `www` and non-`www` |
| `PLATFORM_URL` | `https://api.magicbusstudios.com` | Server-to-server entitlement checks. NEVER `MBS_PLATFORM_URL`. |
| `PRODUCT_SLUG` | `{module_slug}` | Must match what's registered in MBS Platform |
| `SERVICE_NAME` | `{module_slug}` | For Winston logger |
| `OPENAI_API_KEY` | (shared MBS key) | Only if module uses AI. Behind service layer. |
| `FROM_EMAIL` | `noreply@magicbusstudios.com` | Only if module sends email via SendGrid |

**Frontend** (Docker build args — NOT runtime):

| Variable | Value | Notes |
|----------|-------|-------|
| `VITE_BACKEND_URL` | `https://api.{module_slug}.magicbusstudios.com` | Module's own backend API URL |
| `VITE_PLATFORM_URL` | `https://magicbusstudios.com` | MBS frontend — for API calls (via nginx proxy) AND user-facing links (/subscribe, /auth) |
| `VITE_PRODUCT_SLUG` | `{module_slug}` | For branded login |
| `VITE_GOOGLE_CLIENT_ID` | (Google OAuth client ID) | The `.apps.googleusercontent.com` value |

#### 15. Platform registration

- This module must be registered in MBS Platform — either in `MBS/server/config/products.js` or via the admin dashboard at `magicbusstudios.com/admin` → Products tab
- The product slug (`PRODUCT_SLUG`) must match exactly — this is how entitlement checks and SSO work
- If the module isn't registered, SSO redirects won't work and entitlement checks will return errors

#### 16. Docker & deployment

- **Docker**: 2 containers (frontend nginx + backend Express). Dockerfiles in project root (`Dockerfile.backend`, `Dockerfile.frontend`).
- Non-root user in both containers
- HEALTHCHECK in backend Dockerfile
- `VITE_*` env vars are Docker **build args** in Coolify (not runtime)
- Add `www → non-www` redirect in `nginx.conf`
- `trust proxy` set before middleware (see section 10)
- Read `~/.claude/reference/deployment-quirks.md` for any module-specific exceptions

### What to keep (do NOT change)

- Sonner dark glass toasts (NEVER react-hot-toast)
- Framer Motion page transitions
- react-helmet-async for SEO
- lucide-react icons
- The module's unique content, routes, and business logic — DO NOT change functionality

---

### Process

Work through the phases in order. Each phase should compile and run before moving to the next.

**Phase A — Visual alignment:**
1. Read the IL dashboard reference files listed above
2. Migrate to Tailwind, implement design tokens, UI components, API client, polish
3. Verify: `npm run build` (frontend) succeeds, app renders correctly

**Phase B — Auth & entitlement:**
4. Read `~/.claude/reference/entitlement-integration.md`
5. Implement auth redirect flow, entitlement checking, free/premium gating
6. Verify: login flow works, entitlement states handled

**Phase C — Backend standards & GDPR:**
7. Align backend response format, logging, validation, trust proxy, CORS order
8. Implement `DELETE /api/user-data` endpoint
9. Run `npm run test` — all tests pass

**Phase D — Deployment readiness:**
10. Verify env vars, Dockerfiles, nginx config
11. Run `npm run build` (both workspaces) to verify production build
12. Check `git branch` — if `test` or `dev` branches exist, push there (NEVER push to `main` if lower branches exist)
13. Commit with: `feat: align visual standards with Inner Lab dashboard`
