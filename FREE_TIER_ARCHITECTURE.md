# Free Tier Architecture — MBS Platform

**Created**: April 8, 2026 (Session 30)
**Status**: Architecture spec. Not yet built.

## Core Principle

**No user gets access to ANY product without explicitly registering on the MBS billing page.** Logging in creates an MBS account, but does NOT grant access to any product. The user must visit `magicbusstudios.com/billing`, pick a product, and click "Register Free" or "Subscribe Premium."

## Three-State Model

Every user-product relationship is in one of three states:

| State | Entitlement Record | What Happens |
|-------|-------------------|--------------|
| **Not registered** | No record exists | App redirects to billing page |
| **Free subscriber** | `type: "product_pass"`, no `stripe_subscription_id` | App allows access with limits from `freeTierLimits` |
| **Premium subscriber** | `type: "product_pass"` + `stripe_subscription_id`, or `category_access`, or `mbs_all_access` | App allows full access |

**Trial is NOT automatic.** Free registration creates a free-tier entitlement with no trial by default. Trial days are a per-product marketing campaign configured via `defaultTrialDays` in products.js (default: 0). Admin can set trial days per product to run campaigns (e.g., "7-day premium trial for CWG this month"). The lazy downgrade logic (trial → free tier) already works in `entitlements.js` for when trials are active.

## Entitlement Flow

```
User logs into CWG (conversationswithgod.ai)
  → CWG calls GET {PLATFORM_URL}/api/entitlements/cwg
  → Response: { hasAccess: false }
  → CWG redirects to: magicbusstudios.com/billing?product=cwg

User lands on billing page
  → Sees CWG card with "Free" tier (5 messages/day) and "Premium" tier ($15/mo)
  → Clicks "Register Free"
  → POST /api/entitlements/subscribe-free { product: "cwg" }
  → Entitlement created (free tier, no trial by default)
  → Redirect back to conversationswithgod.ai?refresh=true

User returns to CWG
  → CWG calls GET {PLATFORM_URL}/api/entitlements/cwg
  → Response: { hasAccess: true, isPremium: false, reason: "free_tier", limits: { messagesPerDay: 5 } }
  → (If admin has set defaultTrialDays > 0 for CWG: { hasAccess: true, isPremium: true, reason: "trial", trialEndsAt: "..." } — downgrades to free tier after trial expires)
```

## What MBS Platform Does

### Backend (already exists, needs updates)

**Existing:**
- `POST /api/entitlements/subscribe-free` — per-product free registration with 7-day trial
- `GET /api/entitlements/:product` — returns `hasAccess`, `isPremium`, `reason`, `limits`
- `GET /api/entitlements/category/:category` — returns all products in category with access info
- Lazy trial-to-free downgrade in entitlement checks
- Admin dashboard excludes `free_tier` from paid counts

**Needs adding:**
- Handle `?product=<slug>` query param on billing page for deep-linking from apps
- Billing page frontend updates (see below)
- WildLens: set `freeTier: true`, `freeTierLimits: {}` in products.js (module decides actual limits)

### Frontend (BillingPage.jsx — needs redesign)

**Current:** Only shows paid plans (monthly/annual) per category.

**Target:** Each product card shows two tiers:

```
┌─────────────────────────────────────────┐
│  CWG — Conversations With God           │
│                                         │
│  FREE                    │  PREMIUM     │
│  ✓ 5 messages/day        │  $15/mo      │
│  ✓ Basic guides          │  Unlimited   │
│  ✓ 7-day premium trial   │  All guides  │
│                           │  History     │
│  [Register Free]         │  [Subscribe] │
│                                         │
│  Status: Active (Free) ← if registered  │
│  [Upgrade to Premium]   ← if free tier  │
└─────────────────────────────────────────┘
```

**States per product card:**
- Not registered: "Register Free" + "Subscribe" buttons
- Free tier active: "Active (Free)" badge + "Upgrade to Premium"
- Trial active: "Trial (X days left)" badge + "Subscribe to Keep Premium"
- Premium active: "Premium" badge + "Manage Subscription"

**Deep-link handling:**
- `?product=cwg` → scroll to CWG card, highlight it
- `?register=innerlab` → auto-register for Inner Lab category, redirect to innerlab.ai (IL agent's flow)

## What Each Child App Does

### Entitlement Check (on every authenticated page load)

```javascript
// Express middleware or frontend check
const res = await fetch(`${PLATFORM_URL}/api/entitlements/${MY_PRODUCT_SLUG}`, {
  headers: { Authorization: `Bearer ${token}` }
});
const data = await res.json();

if (!data.hasAccess) {
  // Redirect to MBS billing page
  window.location.href = `https://magicbusstudios.com/billing?product=${MY_PRODUCT_SLUG}`;
  return;
}

// Store access info for the session
req.entitlement = {
  isPremium: data.isPremium,
  reason: data.reason,        // "free_tier" | "trial" | "product_pass" | "category_access" | "mbs_all_access"
  limits: data.limits || {},   // { messagesPerDay: 5 } etc.
  trialEndsAt: data.trialEndsAt || null,
};
```

### Limit Enforcement

Each app decides HOW to enforce limits. MBS Platform only provides the limit values. Example for CWG:

```python
# CWG checks entitlement on each message send
if not entitlement.is_premium:
    today_count = await count_messages_today(user_id)
    if today_count >= entitlement.limits.get("messagesPerDay", 5):
        return {"success": False, "message": "Daily message limit reached. Upgrade to Premium for unlimited."}
```

### What Each App Does NOT Do

- Does NOT issue its own entitlements
- Does NOT decide pricing
- Does NOT manage subscriptions
- Does NOT show upgrade/billing UI (redirects to MBS Platform)

## Inner Lab Special Case

Inner Lab is both a **category** (containing 12 modules) and a **platform** (dashboard, consciousness, check-ins, journal, etc.).

**Inner Lab dashboard features** (consciousness, weekly review, daily briefing, check-ins, journal):
- Gated behind Inner Lab registration (user must register for IL)
- Which features are free vs premium is decided by the Inner Lab agent, NOT MBS Platform
- MBS Platform provides: `GET /api/entitlements/category/innerlab` → list of all IL products with access status
- Inner Lab queries this on dashboard load and shows/hides features accordingly

**Inner Lab module picker** (post free-registration):
- After user registers for "Inner Lab" on billing page, redirect to `innerlab.ai/modules` (or similar)
- User sees all 12 modules, each with free tier description
- User clicks individual modules to register (calls MBS Platform `subscribe-free` per product)
- Only registered modules appear in their dashboard sidebar

## Products.js — Free Tier Limits

All products have `freeTier: true` and `freeTierLimits`. Each module's agent decides the actual limit values. Current values in products.js:

| Product | Category | Free Limit | Key |
|---------|----------|-----------|-----|
| CWG | innerlab | 5 messages/day | `messagesPerDay: 5` |
| FlowState | innerlab | 3 sessions/day | `sessionsPerDay: 3` |
| Bonds | innerlab | 3 connections/day | `connectionsPerDay: 3` |
| LifeMap | innerlab | 3 entries/day | `entriesPerDay: 3` |
| StarMap | innerlab | 3 reads/day | `readsPerDay: 3` |
| AstroCompass | innerlab | 3 reads/day | `readsPerDay: 3` |
| Arcana | innerlab | 3 reads/day | `readsPerDay: 3` |
| Archetypes | innerlab | 3 reads/day | `readsPerDay: 3` |
| DreamLens | innerlab | 3 entries/day | `entriesPerDay: 3` |
| Rituals | innerlab | 3 rituals/day | `ritualsPerDay: 3` |
| InnerQuest | innerlab | 3 quests/day | `questsPerDay: 3` |
| Nexus | innerlab | (empty) | `{}` |
| BrokenChain | arcade | 30 min/day | `minutesPerDay: 30` |
| MindHacker | arcade | 30 min/day | `minutesPerDay: 30` |
| Trivia Roast | arcade | 30 min/day | `minutesPerDay: 30` |
| Whispering House | arcade | 30 min/day | `minutesPerDay: 30` |
| Fake Artist | arcade | 30 min/day | `minutesPerDay: 30` |
| WildLens | studioworks | TBD by module | `{}` |
| Lazy Chef | studioworks | 5 recipes/day | `recipesPerDay: 5` |
| Task Tracker | studioworks | 50 tasks total | `tasksTotal: 50` |
| AI Tutor | studioworks | 3 sessions/day | `sessionsPerDay: 3` |
| SmartCart | studioworks | 5 lists total | `listsTotal: 5` |
| Movie Picker | studioworks | 10 picks/day | `picksPerDay: 10` |

## Build Order

1. **MBS Platform backend** — already mostly done. Add WildLens `freeTier: true`. Billing page redesign.
2. **MBS Platform frontend** — BillingPage redesign with free/premium tiers per product.
3. **Inner Lab** — entitlement gate on dashboard, module picker page.
4. **CWG** — entitlement check + redirect, limit enforcement on messages.
5. **FlowState** — entitlement check + redirect, limit enforcement on sessions.
6. **All other apps** — entitlement check + redirect (can be done in parallel via agent prompts).

## Agent Prompts

See `AGENT_PROMPTS_FREE_TIER.md` for paste-ready prompts for each product's agent.
