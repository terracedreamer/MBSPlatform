# Inner Lab — Entitlement Integration Instructions

**Created**: April 8, 2026 (Session 31)
**From**: MBS Platform Agent → Inner Lab Agent
**Status**: Wire up now. MBS will build its side in parallel.

---

## Overview

MBS Platform is adding free tier support. Every user must explicitly register (even for free) before accessing any product. Inner Lab needs to integrate with this system. This document tells you everything you need to wire up on the IL side.

## The One API Call

When a user logs into `innerlab.ai/dashboard`, make this call:

```
GET https://api.magicbusstudios.com/api/entitlements/category/innerlab
Authorization: Bearer <mbs_jwt>
```

Cache the response in localStorage under key `il_entitlement` with a **5-minute TTL**. Bust the cache when the URL contains `?refresh=true` (user returning from MBS after registration/upgrade).

## What MBS Returns

```json
{
  "success": true,
  "hasAccess": true,
  "isPremium": false,
  "reason": "free_tier",
  "products": [
    { "slug": "cwg", "registered": true },
    { "slug": "flowstate", "registered": true },
    { "slug": "bonds", "registered": false },
    { "slug": "lifemap", "registered": false },
    { "slug": "starmap", "registered": false },
    { "slug": "astrocompass", "registered": false },
    { "slug": "arcana", "registered": false },
    { "slug": "archetypes", "registered": false },
    { "slug": "dreamlens", "registered": false },
    { "slug": "rituals", "registered": false },
    { "slug": "innerquest", "registered": false },
    { "slug": "nexus", "registered": false }
  ]
}
```

**MBS sends NO limits, NO daily caps, NO feature lists.** Only:
- `hasAccess` — does this user have any IL entitlement?
- `isPremium` — is it a paid subscription?
- `reason` — what type (`free_tier`, `category_access`, `mbs_all_access`, `none`)
- `products` — which individual modules the user registered for

## Three User States

| State | MBS Returns | What IL Shows |
|-------|------------|---------------|
| **No entitlement** | `{ hasAccess: false }` | Freemium onboarding page. "Register Free" button links to `magicbusstudios.com/subscribe/innerlab` |
| **Free tier** | `{ hasAccess: true, isPremium: false, reason: "free_tier", products: [...] }` | Dashboard with only registered modules in sidebar. IL enforces its own free tier limits. |
| **Premium** | `{ hasAccess: true, isPremium: true, reason: "category_access" }` or `reason: "mbs_all_access"` | Full dashboard, all 12 modules visible, everything unlimited. |

## AuthContext Update

Current access check:
```javascript
const hasInnerLabAccess =
  isAdmin ||
  entitlement?.hasAccess === true ||
  entitlement?.reason === "mbs_all_access" ||
  entitlement?.reason === "category_access";
```

**Add `free_tier`:**
```javascript
const hasInnerLabAccess =
  isAdmin ||
  entitlement?.hasAccess === true ||
  entitlement?.reason === "mbs_all_access" ||
  entitlement?.reason === "category_access" ||
  entitlement?.reason === "free_tier";
```

**Note:** The `free_tier` reason doesn't exist in MBS yet — it will be built. Wire it up now so it works automatically when MBS deploys the update.

## Dashboard Sidebar — Module Visibility

Read the `products` array from the entitlement response:
- **`registered: true`** → show module in sidebar
- **`registered: false`** → hide from sidebar
- **Premium users (`isPremium: true`)** → show ALL 12 modules regardless of `products` array (category_access = everything)
- **"Add Module" link** in sidebar → links to `magicbusstudios.com/subscribe/innerlab` where user can register for more modules

## Free vs Premium Feature Gating

**IL decides what's free vs premium. MBS does not.** Use `isPremium` from the entitlement response:

- `isPremium: false` → free tier. You define the limits (daily caps, feature locks, etc.)
- `isPremium: true` → premium. Everything unlimited.

When you build the free/premium split, ask the user what limits should exist for each feature. MBS does not dictate this.

**Upgrade prompts:** When a free user tries to access a premium feature, show a prompt with a link to `magicbusstudios.com/subscribe/innerlab`.

## The "Register Free" Flow

1. User arrives at `innerlab.ai` with no entitlement
2. IL shows freemium onboarding page
3. User clicks "Register Free"
4. Browser navigates to: **`magicbusstudios.com/subscribe/innerlab`**
5. MBS shows a dedicated Inner Lab subscribe page with:
   - All 12 modules as checkboxes (user picks which ones they want)
   - "Register Free" button
   - Premium option ($20/mo IL All Access)
   - "Want everything?" link to MBS All Access ($30/mo)
6. User picks modules, clicks Register Free
7. MBS creates entitlements → redirects to `innerlab.ai/dashboard?refresh=true`
8. IL sees `?refresh=true`, busts cache, re-fetches entitlements
9. Dashboard loads with selected modules in sidebar

## Premium → Free Downgrade

When a premium user cancels their Stripe subscription:
- MBS auto-downgrades them to `free_tier` (does NOT remove access)
- Next IL entitlement check returns `{ isPremium: false, reason: "free_tier" }`
- IL should handle this gracefully:
  - Premium features become locked
  - Free features stay accessible
  - Show: "Your premium access has ended. You still have free access."
  - Show upgrade prompt linking to `magicbusstudios.com/subscribe/innerlab`
- **Do NOT redirect to a paywall. Do NOT show "Access Denied."**
- The user's registered modules are preserved — they just get limits now instead of unlimited.

## URLs (Important — Billing is being renamed to Subscribe)

| Old URL | New URL |
|---------|---------|
| `magicbusstudios.com/billing` | `magicbusstudios.com/subscribe` |
| `magicbusstudios.com/billing?category=innerlab` | `magicbusstudios.com/subscribe/innerlab` (dedicated IL page) |
| `magicbusstudios.com/billing?product=cwg` | `magicbusstudios.com/subscribe?product=cwg` |

Use the **new URLs** in all links. If the rename hasn't deployed yet when you build, the old URLs will still work (MBS will add redirects).

## Endpoint Reference

| Endpoint | Method | What It Does | Who Calls It |
|----------|--------|-------------|-------------|
| `/api/entitlements/category/innerlab` | GET | Returns access + registered modules | IL frontend |
| `/api/entitlements/subscribe-free` | POST | Creates free_tier entitlement | MBS subscribe page (NOT IL) |
| `/api/entitlements/:product` | GET | Single product access check | Standalone products (CWG, FlowState directly) |
| `/api/billing/checkout` | POST | Stripe checkout | MBS subscribe page (NOT IL) |

**IL does NOT call `subscribe-free` or any billing endpoint.** All registration and payment happens on `magicbusstudios.com/subscribe/innerlab`. IL only reads entitlements.

## What IL Does NOT Do

- Does NOT create, modify, or delete entitlements
- Does NOT call any billing/Stripe/BTCPay endpoints
- Does NOT define limits in MBS products.js (limits live inside IL)
- Does NOT show subscribe/billing UI (redirects to MBS)
- Does NOT store entitlement state in its own DB (localStorage cache only)
- Does NOT verify JWT signatures on the frontend (backend requireAuth does that)

## Admin Override

Users with `isAdmin: true` in the JWT payload always have full access, bypassing all entitlement checks. Current admins: `terracedreamer@gmail.com`, `1984.abhinav@gmail.com`.

## Files to Change (IL Side)

| File | Change |
|------|--------|
| `src/contexts/AuthContext.jsx` | Add `free_tier` to access check, read `products` array, expose `registeredModules` |
| `src/components/ProtectedRoute.jsx` | Accept `free_tier` reason (shows dashboard, not paywall) |
| `src/components/Layout.jsx` | Sidebar shows only registered modules, "Add Module" link |
| Freemium onboarding page | "Register Free" links to `magicbusstudios.com/subscribe/innerlab` |
| All premium-gated features | Check `isPremium` — show upgrade prompt if false |

## What's Not Built Yet on MBS Side

These are in progress on the MBS Platform side. Wire up IL to expect them:

| Feature | Status | What IL Should Do Now |
|---------|--------|----------------------|
| `free_tier` entitlement type | Not yet in Entitlement model | Add `free_tier` to your access check — it will activate when MBS deploys |
| `products` array in category response | Not yet returned | Read it when available, fall back to showing all modules if missing |
| `/subscribe/innerlab` dedicated page | Not yet built | Link to it — MBS will redirect to `/subscribe?category=innerlab` until the dedicated page is ready |
| Premium → free downgrade | Not yet in Stripe webhook | Handle `isPremium: false` gracefully — it will start appearing when MBS adds this |

## Env Vars (No Changes Needed)

IL already has:
- `VITE_PLATFORM_URL` = `https://magicbusstudios.com` (build arg on Coolify)
- The entitlement API call goes to `${VITE_PLATFORM_URL}/api/entitlements/category/innerlab`

No new env vars required for this integration.
