# FlowState — Visual Polish & Entitlement Gating Prompt

> **Purpose**: Bring FlowState up to visual parity with the Inner Lab dashboard and add free/premium entitlement gating. FlowState is already migrated to MBS Platform SSO with GDPR and il_* integration — this prompt covers what's STILL MISSING.
>
> **What's already done (DO NOT redo)**: SSO auth (Phase 4, live on production), GDPR `DELETE /api/user-data` (works, deletes 7 yoga_* + 3 il_* collections), il_* writes (activity_feed, check_ins, user_wellness_profiles), auth middleware (requireAuth + optionalAuth), TypeScript frontend, Zustand stores, i18n (5 languages).

---

## Scope — What FlowState Needs

FlowState is an **Express/Node.js** backend + **React/Vite/TypeScript** frontend. It's already SSO-migrated and live on production. This prompt targets two gaps:

1. **Visual polish** — align design tokens with IL dashboard (DECISION REQUIRED on CSS Modules)
2. **Entitlement gating** — add `isPremium` check for free vs premium feature differentiation

**Do NOT change**: auth flow, GDPR endpoint, il_* data writes, backend API routes, core business logic (yoga sessions, poses, programs), PWA/service worker, i18n translations.

---

### Reference: Read these first

- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\index.css` — IL dashboard design tokens (target palette)
- `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab\src\components\ui\` — UI component patterns
- `C:\Users\1984a\OneDrive\Desktop\Codes\YogaGhost\CURRENT_STATUS.md` — current state
- `C:\Users\1984a\OneDrive\Desktop\Codes\YogaGhost\SESSION_HANDOFF.md` — latest session context
- `~/.claude/reference/entitlement-integration.md` — entitlement gating patterns

---

## Part 1: Visual Polish

### IMPORTANT — CSS Modules Decision

FlowState uses **CSS Modules** (`.module.css` per component) with CSS custom properties in `src/index.css`. This is a **documented valid deviation** from the Tailwind standard in the global CLAUDE.md.

**Decision: Option A — Keep CSS Modules, align colors only.** FlowState's CSS Modules deviation is documented and valid. Update CSS custom properties in `src/index.css` to match IL dashboard palette. Keep all `.module.css` files — just update color values. Do NOT migrate to Tailwind.

### 1.1 Design token alignment

Whether CSS Modules or Tailwind, update the color palette in `src/index.css` to match IL:

```css
:root {
  /* Backgrounds */
  --bg-darkest: #050505;
  --bg-dark: #0a0a0a;
  --bg-panel: #111111;
  --bg-elevated: #1a1a1a;

  /* Borders */
  --border-default: rgba(255, 255, 255, 0.06);
  --border-hover: rgba(255, 255, 255, 0.1);

  /* Text */
  --text-primary: #e5e5e5;
  --text-secondary: #aaaaaa;
  --text-muted: #666666;

  /* Glass */
  --glass-bg: rgba(255, 255, 255, 0.03);
  --glass-bg-hover: rgba(255, 255, 255, 0.06);
  --glass-blur: 24px;

  /* Accent — FlowState keeps teal as its identity color */
  --accent: #14b8a6;
  --accent-glow: rgba(20, 184, 166, 0.08);
}
```

Map these to FlowState's existing CSS variable names (the variable names may differ — update values, not necessarily names, unless consolidating).

### 1.2 Custom toast → Sonner migration

FlowState uses a **custom Toast implementation** (`src/components/Toast.tsx` + `Toast.module.css` with `ToastProvider` + `useToast()` hook). The MBS standard is **Sonner**.

**Migrate to Sonner:**
1. Install `sonner`
2. Replace `<ToastProvider>` with `<Toaster theme="dark" />` in App.tsx
3. Replace all `useToast()` calls with `import { toast } from "sonner"`
4. Delete `Toast.tsx`, `Toast.module.css`, and the custom toast context
5. Sonner usage: `toast.success("Saved")`, `toast.error("Failed")`

### 1.3 UI patterns

Ensure these exist (create if missing):

- **Skeleton loading** — shimmer placeholders on data-fetching pages (replace any "Loading..." text)
- **Empty states** — title + description + CTA (not just blank screens)
- **Error states** — retry button on failures

### 1.4 Typography check

Verify fonts match (or add if missing):
- Headings: Space Grotesk (600/700)
- Body: **Inter** (FlowState's established body font — keep this, do NOT change to DM Sans)
- Display/accent: Instrument Serif italic
- Mono: JetBrains Mono
- Load via Google Fonts with `display=swap`

### 1.5 Frontend safety rules

Verify these are followed (fix any violations):
- `(array || []).map()` for all array renders — never bare `.map()` on potentially null data
- No `alert()`, `prompt()`, or `confirm()` — use Sonner toasts or modal components
- Mobile responsive at 640px, 768px, 1024px breakpoints

### 1.6 API client check

FlowState has `src/lib/api.ts` with centralized fetch and `getAuthHeader()`. Verify it has:
- Auth header injection on every call
- 401 handling (clear token, redirect to `innerlab.ai/auth/login`)
- Consistent error handling

---

## Part 2: Entitlement Gating (Free vs Premium)

FlowState already has an entitlement check in `src/lib/api.ts` (`fetchEntitlements()`) that calls `GET /api/entitlements/flowstate`. But it only checks `hasAccess` — it doesn't differentiate free vs premium.

### 2.1 Backend & frontend — add `isPremium` to entitlement check

FlowState's frontend calls MBS directly from `src/lib/api.ts` (`fetchEntitlements()` → `GET /api/entitlements/flowstate`).

**PLATFORM_URL dual-use pattern**:
- Backend `PLATFORM_URL` = `https://api.magicbusstudios.com` — server-to-server calls
- Frontend `VITE_PLATFORM_URL` = `https://magicbusstudios.com` — API calls (nginx proxies `/api/*` to backend) AND user-facing links (`/subscribe/*`)
- Both are correct for their context. Do NOT "fix" one to match the other.

Update `fetchEntitlements()` in `api.ts` to read `isPremium` from the response:

Expected MBS response:
```json
{ "hasAccess": true, "isPremium": false, "reason": "free_tier" }
```

Also add a backend middleware for server-side premium checks on protected routes (uses `PLATFORM_URL` = `api.magicbusstudios.com`).

### 2.2 Frontend — expose tier in auth store

FlowState uses **Zustand** (`useAuthStore`). Add `isPremium` to the store:

```typescript
interface AuthState {
  // ... existing fields
  isPremium: boolean;
  setIsPremium: (val: boolean) => void;
}

// Default to true until MBS free tier is live
// TODO: remove effectivePremium when MBS free tier is live
const EFFECTIVE_PREMIUM = true;
```

### 2.3 Backend — premium gating middleware

Create middleware for routes that need premium checks:

```javascript
async function requirePremium(req, res, next) {
  // TODO: remove effectivePremium when MBS free tier is live
  const EFFECTIVE_PREMIUM = true;
  if (EFFECTIVE_PREMIUM) return next();

  const entitlement = req.entitlement; // set by requireEntitlement middleware
  if (!entitlement?.isPremium) {
    return res.status(403).json({
      success: false,
      message: "Premium feature — upgrade for access",
      upgradeUrl: "https://magicbusstudios.com/subscribe/innerlab",
    });
  }
  next();
}
```

### 2.4 Define free vs premium features

**Decision: Wire `isPremium` with `effectivePremium = true` and decide the actual split later.** No specific limits are defined yet for any module. Just get the plumbing in place so features can be gated when limits are decided.

Wire `isPremium` checks in the auth store and create a `requirePremium` backend middleware, but keep `effectivePremium = true` so everything works as full-access until MBS free tier is live.

### 2.5 Upgrade prompts

When a free user tries a premium feature:

```typescript
import { toast } from "sonner"; // after migration

toast.info("This feature is available with Premium.", {
  action: {
    label: "Upgrade",
    onClick: () => window.open("https://magicbusstudios.com/subscribe/innerlab", "_blank"),
  },
});
```

### 2.6 Handle `?refresh=true`

After a user upgrades on MBS, they're redirected back with `?refresh=true`. FlowState frontend must:
1. Detect `?refresh=true` in the URL
2. Bust the entitlement cache
3. Re-fetch entitlement
4. Remove `?refresh=true` from URL with `history.replaceState`

### 2.7 Downgrade handling

When `isPremium` changes from `true` to `false`:
- Lock premium features, keep free features working
- Do NOT redirect to a paywall
- Show: "Your premium access has ended. You still have free access."

---

## Part 3: Verify Existing Integration

Quick checks that existing SSO/GDPR/il_* integration is solid:

### 3.1 GDPR endpoint (IMPORTANT — potential bug)

Verify `DELETE /api/user-data` in `server/index.js` (line ~674):
- Deletes all 7 `yoga_*` collections for the user — correct
- Deletes `il_activity_feed` and `il_check_ins` WHERE `source_module = "flowstate"` — correct
- **Does NOT delete `il_user_wellness_profiles`** — this is an identity singleton (one per user, shared across body-aware modules). Per data sovereignty rules, identity singletons are only deleted at category-level or higher, NEVER at app-level.
- Returns `{ success: true }`

**Check if the current code deletes `il_user_wellness_profiles` during app-level deletion. If it does, that's a bug — remove it from the app-level delete.** The wellness profile should survive a FlowState-only data deletion because other modules (or future ones) may depend on it. Only Inner Lab middleware (category-level deletion) should delete it.

### 3.2 il_* source_module consistency

Grep for `source_module` across the backend. Every write to an `il_*` collection must set `source_module: "flowstate"`. Verify this is consistent.

### 3.3 Trust proxy

Verify `app.set("trust proxy", 1)` is present BEFORE any middleware in `server/index.js`. FlowState was flagged as "needs check" in the deployment quirks reference. If missing, add it — required for Coolify/Traefik deployment.

---

## What NOT to Change

- **Backend framework** — stays Express/Node.js
- **Auth middleware** — `requireAuth` + `optionalAuth` in `server/auth.js` works. Don't rewrite.
- **Zustand stores** — keep the existing state management pattern
- **PWA/Service worker** — don't touch offline caching
- **i18n** — don't modify translations (5 languages). New UI strings need translation keys but can be English-only initially.
- **Database** — `inner_lab` with `yoga_*` prefix. Don't touch collection structure.
- **il_* write paths** — already implemented and working
- **Backend tests** — 29 tests (Jest + supertest). Don't break them.

---

## Process

1. Read CURRENT_STATUS.md and SESSION_HANDOFF.md for latest context
2. **Ask owner: CSS Modules (Option A) or Tailwind (Option B)?** Default to A if no answer.
3. **Visual polish** — align colors, migrate toasts to Sonner, add skeleton/empty/error states
4. **Entitlement gating** — wire `isPremium`, add effectivePremium flag
5. **Verify** GDPR, il_* integration, trust proxy
6. Run tests: `npm run test` — all 29 tests pass
7. Run `npm run build` to verify production build
8. FlowState is currently on `dev` branch. Push to `dev`. Owner will merge to `main` when ready.
9. Commit with: `feat: visual polish + entitlement gating prep`
