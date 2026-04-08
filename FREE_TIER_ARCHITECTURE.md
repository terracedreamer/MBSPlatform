# Free Tier Architecture — MBS Platform

**Created**: April 8, 2026 (Session 30)
**Updated**: April 8, 2026 (Session 31)
**Status**: Architecture spec. Not yet built.

## Core Principle

**No user gets access to ANY product without explicitly registering on the MBS Subscribe page.** Logging in creates an MBS account, but does NOT grant access to any product. The user must visit `magicbusstudios.com/subscribe`, pick a product (or category + modules), and click "Register Free" or "Subscribe Premium."

## Three-State Model

Every user-product relationship is in one of three states:

| State | Entitlement Record | What Happens |
|-------|-------------------|--------------|
| **Not registered** | No record exists | App redirects to subscribe page |
| **Free subscriber** | `type: "free_tier"` | App allows access — module enforces its own limits |
| **Premium subscriber** | `type: "product_pass"` + `stripe_subscription_id`, or `category_access`, or `mbs_all_access` | App allows full access |

**Trial is NOT automatic.** Free registration creates a free-tier entitlement with no trial by default. Trial days are a per-product marketing campaign configured via `defaultTrialDays` in products.js (default: 0). Admin can set trial days per product to run campaigns (e.g., "7-day premium trial for CWG this month").

## What MBS Platform Returns

MBS answers **one question**: "Does this user have access, and is it free or premium?"

### Per-product check (standalone apps like CWG, FlowState, WildLens, etc.)

`GET /api/entitlements/:product`

```json
{ "hasAccess": true, "isPremium": false, "reason": "free_tier" }
```
or
```json
{ "hasAccess": true, "isPremium": true, "reason": "product_pass" }
```
or
```json
{ "hasAccess": false }
```

**No limits, no freeTierLimits, no daily caps.** MBS does not know or care how each module enforces its free tier. Each module defines and enforces its own limits internally.

### Per-category check (Inner Lab special case)

`GET /api/entitlements/category/innerlab`

```json
{
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

IL reads the `products` array → only shows registered modules in the sidebar.

## Entitlement Flow — Standalone Products

```
User logs into CWG (conversationswithgod.ai)
  → CWG calls GET {PLATFORM_URL}/api/entitlements/cwg
  → Response: { hasAccess: false }
  → CWG redirects to: magicbusstudios.com/subscribe?product=cwg

User lands on subscribe page
  → Sees CWG card with "Free" and "Premium" options
  → Clicks "Register Free"
  → POST /api/entitlements/subscribe-free { product: "cwg" }
  → Entitlement created: { type: "free_tier", product: "cwg" }
  → Redirect back to conversationswithgod.ai?refresh=true

User returns to CWG
  → CWG calls GET {PLATFORM_URL}/api/entitlements/cwg
  → Response: { hasAccess: true, isPremium: false, reason: "free_tier" }
  → CWG allows access, enforces its own limits internally
```

## Entitlement Flow — Inner Lab (Special Case)

```
User logs into innerlab.ai
  → IL calls GET {PLATFORM_URL}/api/entitlements/category/innerlab
  → Response: { hasAccess: false }
  → IL shows freemium onboarding → "Register Free" links to magicbusstudios.com/subscribe?category=innerlab

User lands on MBS Subscribe page (Inner Lab section)
  → Sees all 12 IL modules with checkboxes
  → Checks CWG + FlowState
  → Clicks "Register Free"
  → MBS creates:
    1. Category entitlement: { type: "free_tier", category: "innerlab" }
    2. Product entitlement: { type: "free_tier", product: "cwg" }
    3. Product entitlement: { type: "free_tier", product: "flowstate" }
  → Redirect to innerlab.ai/dashboard?refresh=true

User returns to IL dashboard
  → IL calls GET {PLATFORM_URL}/api/entitlements/category/innerlab
  → Response: { hasAccess: true, isPremium: false, reason: "free_tier", products: [{ slug: "cwg", registered: true }, { slug: "flowstate", registered: true }, ...] }
  → IL shows CWG + FlowState in sidebar
  → "Add Module" link takes user back to magicbusstudios.com/subscribe (IL section)
```

## Premium Subscription Flow

```
Free user clicks "Upgrade to Premium" (anywhere)
  → Redirects to magicbusstudios.com/subscribe
  → User selects premium plan ($20/mo Inner Lab All Access, $30/mo Everything, etc.)
  → Stripe checkout flow
  → Entitlement upgraded: { type: "category_access", category: "innerlab", stripe_subscription_id: "sub_..." }
  → All 12 IL modules become available (category_access = all products in category)
```

## Premium Cancellation (Downgrade)

```
Premium user cancels Stripe subscription
  → Stripe webhook: customer.subscription.deleted
  → MBS marks category_access entitlement as "cancelled"
  → MBS auto-creates: { type: "free_tier", category: "innerlab" }
  → Per-module free_tier entitlements preserved (user keeps their module selections)
  → Next IL load: { hasAccess: true, isPremium: false, reason: "free_tier" }
  → IL locks premium features, keeps free features accessible
  → User sees: "Your premium access has ended. You still have free access."
  → User is NOT kicked out or redirected to a paywall
```

## Entitlement Model Changes

Add `free_tier` to the type enum in `server/models/Entitlement.js`:

```javascript
type: {
  type: String,
  enum: ["product_pass", "category_access", "mbs_all_access", "free_tier"],
  required: true,
}
```

Free-tier entitlements:
- No `stripe_subscription_id`
- No `expires_at` (never expires)
- Can be per-product (`product: "cwg"`) or per-category (`category: "innerlab"`)

## What MBS Platform Does

### Backend
- `POST /api/entitlements/subscribe-free` — creates free_tier entitlement (per product or category + selected modules)
- `GET /api/entitlements/:product` — returns `{ hasAccess, isPremium, reason }`
- `GET /api/entitlements/category/:cat` — returns `{ hasAccess, isPremium, reason, products: [...] }`
- Stripe webhook handles premium → free_tier downgrade on cancellation

### Frontend — Two Subscribe Pages

**Main Subscribe Page** (`/subscribe` — was `/billing`):
- Rename `BillingPage.jsx` → `SubscribePage.jsx`
- Shows all 3 categories equally, MBS All Access hero
- Each category: free vs premium options
- Arcade + Studio Works: category-level registration (all games/tools included)
- Inner Lab section: summary + "See all modules →" link to `/subscribe/innerlab`
- Deep-link: `?product=cwg` scrolls to CWG card

**Dedicated Inner Lab Subscribe Page** (`/subscribe/innerlab` — NEW):
- `SubscribeInnerLabPage.jsx` — focused entirely on Inner Lab
- Hero: "Inner Lab — Your Inner Growth System"
- All 12 modules displayed as selectable cards with checkboxes
- User picks modules → "Register Free — N modules selected" button
- Premium option: "$20/mo — All 12 modules, unlimited everything" → Stripe checkout
- Bottom section: "Want everything? MBS All Access — $30/mo" → link to `/subscribe`
- This is where `innerlab.ai` redirects unregistered users
- `?register=innerlab` on main subscribe page redirects here
- Same backend APIs — just different frontend layout

## What Each Module Does

- Calls MBS to check `hasAccess` and `isPremium` — that's it
- If `hasAccess: false` → redirect to `magicbusstudios.com/subscribe`
- If `isPremium: false` → enforce its own free tier limits (daily caps, feature locks, etc.)
- If `isPremium: true` → full access, no limits
- **Each module defines and enforces its own limits.** MBS does not pass limits, daily caps, or feature lists.

## What Each Module Does NOT Do

- Does NOT issue its own entitlements
- Does NOT decide pricing
- Does NOT manage subscriptions
- Does NOT show subscribe/billing UI (redirects to MBS Subscribe page)
- Does NOT define limits in MBS products.js (limits live inside each module)

## Products.js Cleanup

**Remove `freeTierLimits` from all products in `products.js`.** These were placeholder values. Each module owns its own limits internally. Products.js only needs:

```javascript
{
  slug: "cwg",
  name: "Conversations With God",
  category: "innerlab",
  freeTier: true,            // keep — indicates free tier is available
  premiumFeatures: [...],    // keep — used for subscribe page display
  // freeTierLimits: REMOVED — module enforces its own limits
}
```

## Build Order

1. **MBS Platform backend** — add `free_tier` type to Entitlement model, update `subscribe-free` to handle category + module selections, update `GET /api/entitlements/category/:cat` to return `products` array with registered status, handle premium→free downgrade in Stripe webhook, remove `freeTierLimits` from products.js
2. **MBS Platform frontend** — rename Billing→Subscribe, redesign page with free/premium per category, IL module checkboxes, deep-link handling
3. **Inner Lab** — accept `free_tier` reason in AuthContext, read `products` array for registered modules, gate dashboard features by `isPremium`, show "Add Module" link
4. **CWG** — entitlement check + redirect, enforce its own free limits internally
5. **FlowState** — entitlement check + redirect, enforce its own free limits internally
6. **All other apps** — entitlement check + redirect (can be parallelized)

## Agent Prompts

See `AGENT_PROMPTS_FREE_TIER.md` for paste-ready prompts (needs updating to match this doc).
