# CWG — Visual Polish & Entitlement Gating Prompt

> **Purpose**: Bring CWG up to visual parity with the Inner Lab dashboard and add free/premium entitlement gating. CWG is already migrated to MBS Platform SSO with GDPR and il_* integration — this prompt covers what's STILL MISSING.
>
> **What's already done (DO NOT redo)**: SSO auth (Phase 3), GDPR `DELETE /api/user-data` (works), il_* writes (reflections, check-ins, memories, activity feed, consciousness, personal histories), Sonner toasts, Framer Motion, Tailwind CSS, Vite build.

---

## IMPORTANT — Feature Removal First

**If CWG has a pending feature removal (7 features being stripped), do that BEFORE running this alignment prompt.** Otherwise you'll polish features that get deleted in the next session. Check with the owner or the CWG agent whether feature removal has been completed. If not, do feature removal first, then come back to this prompt.

## Scope — What CWG Needs

CWG is a **Python/FastAPI** backend + **React/Vite** frontend. It already has the most functionality of any IL module. This prompt targets two gaps:

1. **Visual polish** — align design tokens and UI patterns with IL dashboard
2. **Entitlement gating** — add `isPremium` check for free vs premium feature differentiation

**Do NOT change**: auth flow, GDPR endpoint, il_* data writes, backend API routes, core business logic, or the Python backend patterns.

---

### Reference: Read these first

- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\index.css` — IL dashboard design tokens (target palette)
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\components\ui\` — PageShell, Card, EmptyState, Skeleton patterns
- `C:\Users\1984a\OneDrive\Desktop\Codes\CWG\CURRENT_STATUS.md` — current state
- `C:\Users\1984a\OneDrive\Desktop\Codes\CWG\SESSION_HANDOFF.md` — latest session context
- `~/.claude/reference/entitlement-integration.md` — entitlement gating patterns

---

## Part 1: Visual Polish

CWG already uses Tailwind with a custom design token system in `frontend/tailwind.config.js` and `frontend/src/index.css`. The goal is to **align CWG's palette with IL dashboard** while keeping CWG's identity (blue #3B82F6 + purple #7C3AED as accent).

### 1.1 Design token alignment

Update `frontend/src/index.css` and/or `tailwind.config.js` so the base chrome matches IL:

```
Backgrounds:   #050505 (darkest), #0a0a0a, #111111, #1a1a1a (panels)
Borders:        rgba(255,255,255,0.06) (default), rgba(255,255,255,0.1) (hover)
Text:           #e5e5e5 (primary), #aaaaaa (secondary), #666666 (muted)
Glass panels:   bg-white/[0.03], backdrop-blur-xl, border border-white/[0.06]
Glass hover:    bg-white/[0.06], border-white/[0.1]
Scrollbar:      6px wide, #333 thumb, transparent track
```

**Keep CWG's accent colors** (blue/purple) for branding — only align the neutral chrome.

### 1.2 UI component patterns

CWG already has Radix UI primitives. Ensure these patterns exist (create if missing):

- **Skeleton loading** — shimmer placeholders on every data-fetching page (replace any "Loading..." text)
- **Empty states** — decorative glow + title + description + CTA (not just "No data")
- **Error states** — retry button on failures
- **Hover micro-interactions** — scale/glow on cards and buttons

### 1.3 Typography check

Verify fonts match the standard:
- Headings: Space Grotesk (600/700)
- Body: DM Sans (400/500)
- Display/accent: Instrument Serif italic
- Mono: JetBrains Mono

### 1.4 Frontend safety rules

Verify these are followed (fix any violations):
- `(array || []).map()` for all array renders — never bare `.map()`
- No `alert()`, `prompt()`, or `confirm()` — Sonner toasts or modals only
- Mobile responsive at 640px, 768px, 1024px breakpoints

### 1.5 API client pattern

CWG has `frontend/src/utils/api.js`. Verify it has:
- Centralized auth header injection (`Authorization: Bearer {token}`)
- 401 handling (clear token, redirect to `innerlab.ai/auth/login`)
- Consistent error handling

If the API client is scattered across components instead of centralized, consolidate it.

---

## Part 2: Entitlement Gating (Free vs Premium)

CWG already has an entitlement check in `backend/routers/profile_routes.py` (`check_entitlement()`, `get_plan_from_entitlement()`). But it doesn't yet differentiate free vs premium features.

### 2.1 Backend — check `isPremium`

CWG's existing entitlement check calls MBS Platform via `MBS_PLATFORM_URL` (which points to `api.magicbusstudios.com`). Verify the response now includes `isPremium`:

```python
# Expected response from MBS:
# { "hasAccess": true, "isPremium": false, "reason": "free_tier" }
# { "hasAccess": true, "isPremium": true, "reason": "product_pass" }
# { "hasAccess": false }
```

**PLATFORM_URL note**: CWG uses the legacy env var name `MBS_PLATFORM_URL` (not the canonical `PLATFORM_URL`). This is fine — don't rename it. It points to `https://api.magicbusstudios.com` for server-to-server calls. The frontend uses `VITE_PLATFORM_URL` = `https://magicbusstudios.com` for API calls (nginx proxies `/api/*`) and user-facing links (`/subscribe/*`).

Update `check_entitlement()` to extract and return `isPremium` alongside the existing fields. Cache for 5 minutes (if not already cached).

### 2.2 Backend — premium gating middleware

Create a utility for routes that need premium checks:

```python
async def check_premium(user_id: str, token: str) -> bool:
    """Returns True if user has premium access."""
    entitlement = await check_entitlement(user_id, token)
    return entitlement.get("isPremium", False)
```

For now, use `effectivePremium = True` pattern — hardcode to bypass until MBS free tier is live:

```python
# TODO: remove effectivePremium when MBS free tier is live
EFFECTIVE_PREMIUM = True

async def is_premium(user_id: str, token: str) -> bool:
    if EFFECTIVE_PREMIUM:
        return True
    return await check_premium(user_id, token)
```

### 2.3 Frontend — expose tier in context

The frontend auth context should expose `isPremium` so components can gate features:

```javascript
// In auth context or user store
const [isPremium, setIsPremium] = useState(true); // TODO: default true until MBS free tier is live

// When entitlement response arrives:
setIsPremium(entitlement.isPremium ?? true); // fallback to true during development
```

### 2.4 Define free vs premium features

**Work with the owner to decide what's free vs premium in CWG.** Only define splits for features that STILL EXIST after feature removal. Possible split:

| Feature | Free | Premium |
|---------|------|---------|
| Daily conversations | Limited (e.g., 3/day) | Unlimited |
| Conversation history | Last 30 days | Full history |
| AI depth | Standard model | Deep model |
| Journal/reflections | Limited entries | Unlimited |
| Export data | JSON only | JSON + PDF |

**Note**: CWG has a feature removal in progress (7 features being stripped). Do NOT add premium gating to features that are being removed. Check current feature list after removal before building this split.

**These are suggestions only** — the owner decides. The code should be wired to check `isPremium` so limits can be toggled later.

### 2.5 Upgrade prompts

When a free user hits a limit or tries a premium feature:

```javascript
toast.info("This feature is available with Premium. Upgrade for unlimited access.", {
  action: {
    label: "Upgrade",
    onClick: () => window.open("https://magicbusstudios.com/subscribe/innerlab", "_blank"),
  },
});
```

### 2.6 Handle `?refresh=true`

After a user upgrades on MBS, they're redirected back with `?refresh=true`. CWG frontend must:
1. Detect `?refresh=true` in the URL
2. Bust the entitlement cache
3. Re-fetch entitlement
4. Remove `?refresh=true` from URL with `history.replaceState`

### 2.7 Downgrade handling

When `isPremium` changes from `true` to `false` (premium cancelled):
- Lock premium features, keep free features working
- Do NOT redirect to a paywall
- Show: "Your premium access has ended. You still have free access."

---

## Part 3: Verify Existing Integration

Quick verification that existing SSO/GDPR/il_* integration is solid:

### 3.1 GDPR endpoint

Verify `DELETE /api/user-data` in `backend/server.py`:
- Deletes all `cwg_*` collections for the user
- Deletes `il_*` entries WHERE `source_module = "cwg"` (check-ins, memories, activity feed, reflections)
- Does NOT delete identity singletons (consciousness profiles, personal histories, wellness profiles)
- Returns `{ success: true }`

### 3.2 il_* source_module consistency

Grep for `source_module` across the backend. Every write to an `il_*` collection must set `source_module: "cwg"`. Verify this is consistent.

---

## What NOT to Change

- **Backend framework** — stays Python/FastAPI. Do NOT suggest migrating to Express.
- **Auth middleware** — `get_current_user_id()` in `core/dependencies.py` works. Don't rewrite.
- **Env var names** — CWG uses `JWT_SECRET_KEY` (fallback `JWT_SECRET`) and `MBS_PLATFORM_URL` (legacy, points to `api.magicbusstudios.com`). These work. Don't rename unless doing a full refactor. Frontend uses `VITE_PLATFORM_URL` = `magicbusstudios.com` (nginx proxies `/api/*` to backend).
- **Database** — `inner_lab` with `cwg_*` prefix. Don't touch collection structure.
- **il_* write paths** — already implemented and working. Don't change.
- **Backend tests** — Playwright + Vitest. Don't break them.

---

## Process

1. Read CURRENT_STATUS.md and SESSION_HANDOFF.md for latest context
2. **Visual polish first** — align design tokens, add skeleton/empty/error states
3. **Entitlement gating** — wire `isPremium`, add effectivePremium flag
4. **Verify** existing GDPR and il_* integration
5. Run existing tests: `npm run test` (frontend), backend tests if configured
6. Run `npm run build` to verify production build
7. Check `git branch` — CWG has `test` → `development` → `main`. Push to `test` branch.
8. Commit with: `feat: visual polish + entitlement gating prep`
