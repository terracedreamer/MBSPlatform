# Standalone Product — MBS Platform SSO Migration

## Prerequisites — Do NOT Start Until This Is Built
1. **MBS Platform (Layer 1)** at magicbusstudios.com must be live with SSO + entitlements API
2. The platform must be tested and working before ANY standalone product migration begins

## What's Happening
This product is being migrated to use the centralized MBS Platform for authentication and entitlement checks. This is a **simple SSO integration** — no database migration, no shared data layer.

## Important: Fill In These Values First

Before starting, the agent (or you) must fill in these product-specific values:

| Variable | Value | Example |
|----------|-------|---------|
| PRODUCT_SLUG | ??? | `brokenchain`, `wildlens`, `lazychef` |
| PRODUCT_DOMAIN | ??? | `brokenchain.magicbusstudios.com` |
| PRODUCT_CATEGORY | ??? | `arcade` or `studioworks` |
| BRAND_PARAM | `mbs` | Arcade and Studio Works always use MBS branding |
| CURRENT_AUTH | ??? | What auth does this product currently have? (Google SSO, email/password, none, etc.) |
| CURRENT_BILLING | ??? | What billing does this product currently have? (Stripe, none, etc.) |

## Target State
- Auth handled by MBS Platform at magicbusstudios.com (Google SSO + Email/Password + Nostr + LNURL + 2FA/TOTP)
- Billing handled by MBS Platform (Stripe + BTCPay) — if product has premium features
- Product keeps its own database (no migration to inner_lab)
- Product keeps all its own backend logic
- Only auth and billing change

## What This Product Does NOT Get
- No shared data layer (that's Inner Lab only)
- No consciousness profiles, memories, or check-ins
- No cross-product intelligence
- No connection to `inner_lab` database
- Friends/invites are platform-level but cross-product visibility is optional

---

## Migration Steps

### Step 1: Add JWT Validation Middleware

Add this middleware to your backend. Every protected route uses it:

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

### Step 2: Update Login Flow

**Remove:**
- Any standalone login/signup pages
- Any standalone auth routes (`/auth/login`, `/auth/signup`, `/auth/logout`, etc.)
- Password hashing, session management, auth token generation
- Google OAuth callback handling (the platform handles this now)

**Add:**
- Login button redirects to:
  ```
  https://magicbusstudios.com/auth/login?redirect={PRODUCT_DOMAIN}&brand=mbs
  ```
- After login, MBS Platform redirects back with `?token={JWT}`
- Frontend stores JWT (localStorage or cookie), sends in `Authorization: Bearer {JWT}` header
- Logout clears the stored JWT and optionally calls `POST https://magicbusstudios.com/api/auth/logout`

### Step 3: Remove Standalone Billing (If Any)

**Remove:**
- Stripe integration (checkout, webhooks, portal, env vars)
- Pricing pages
- Subscription management UI

**Add:**
- Upgrade/billing buttons redirect to:
  ```
  https://magicbusstudios.com/billing?product={PRODUCT_SLUG}&brand=mbs
  ```
- Check access from your backend:
  ```javascript
  const response = await fetch(
    `https://magicbusstudios.com/api/entitlements/${PRODUCT_SLUG}`,
    { headers: { Authorization: `Bearer ${userToken}` } }
  );
  const { hasAccess, reason } = await response.json();
  // hasAccess: true/false
  // reason: "free_tier", "product_pass", "category_access", "mbs_all_access", "no_subscription"
  ```
- If `hasAccess: false`: redirect to MBS Platform billing page

### Step 4: Update Frontend

- Remove login/signup page components
- Remove pricing/billing page components (if any)
- Update auth store/context to use MBS Platform JWT
- Update API client to send JWT in Authorization header
- Add "Login" button that redirects to MBS Platform
- Handle `?token={JWT}` on redirect back from platform

### Step 5: Update Backend

- Add `requireAuth` middleware to all protected routes
- Remove standalone auth routes
- Remove Stripe/billing routes (if any)
- Add `JWT_SECRET` to environment variables (must match MBS Platform)
- Keep all product-specific routes and logic unchanged

### Step 6: Update Environment Variables

**Remove:**
- Any auth-related env vars (GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET — if handled locally)
- Any Stripe env vars (STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, etc.)

**Add:**
- `JWT_SECRET` — must match the MBS Platform's secret exactly
- `PLATFORM_URL` — `https://magicbusstudios.com`
- `PRODUCT_SLUG` — your product's slug (e.g., `brokenchain`)

**Keep (do NOT remove):**
- `MONGO_URL` — your product's own database connection (NOT inner_lab, NOT mbs_platform)
- `DB_NAME` — your product's own database name
- All other existing product-specific env vars (OPENAI_API_KEY, etc.)

---

## Free Tier Handling

Some products have free tiers (e.g., Arcade: 30 min/day free). The platform's entitlement check returns:
- `{ hasAccess: true, reason: "free_tier" }` for free users
- `{ hasAccess: true, reason: "product_pass" }` for paid users

Your product's backend is responsible for enforcing free tier limits (time, usage caps, etc.) based on the `reason` field.

## Products That Need This Migration

### The Arcade (5 games) — Category: `arcade`, Brand: `mbs`

| Product | Slug | Domain |
|---------|------|--------|
| Broken Chain | `brokenchain` | brokenchain.magicbusstudios.com |
| MindHacker | `mindhacker` | mindhacker.magicbusstudios.com |
| Trivia Roast | `triviaroast` | triviaroast.magicbusstudios.com |
| Whispering House | `whisperinghouse` | whisperinghouse.magicbusstudios.com |
| Fake Artist | `fakeartist` | fakeartist.magicbusstudios.com |

### Studio Works (6 apps) — Category: `studioworks`, Brand: `mbs`

| Product | Slug | Domain |
|---------|------|--------|
| WildLens | `wildlens` | wildlens.magicbusstudios.com |
| Lazy Chef | `lazychef` | lazy-chef.magicbusstudios.com |
| Family Task Tracker | `tasktracker` | tasktracker.magicbusstudios.com |
| AI Tutor | `tutor` | tutor.magicbusstudios.com |
| SmartCart | `smartcart` | smartcart.magicbusstudios.com |
| Movie Picker | `moviepicker` | moviepicker.magicbusstudios.com |

## What NOT to Do
- Do NOT touch the product's own database — keep it as-is
- Do NOT connect to `inner_lab` database — that's Inner Lab modules only
- Do NOT build your own Stripe integration — the platform handles billing
- Do NOT keep standalone auth after migration — remove it completely
- Do NOT hardcode the product slug — use env var `PRODUCT_SLUG`

## GDPR Compliance Note
The MBS Platform's GDPR cascade delete (`DELETE /api/auth/account`) automatically deletes user data from `mbs_platform` and `inner_lab` databases. It does NOT reach into standalone product databases. If your product stores user-specific data (keyed by `userId` from JWT), you MUST implement a `DELETE /api/user-data` endpoint that deletes all user data when called. The MBS Platform will call this endpoint during account deletion (planned — not yet implemented). Until then, standalone products with user data have a GDPR gap for account deletion.

---

## Completion Report (REQUIRED)

When you finish the SSO migration, generate a file called `PHASE_5_REPORT.md` in the project root. The orchestrator will fetch it from here. The report must contain:

1. **What was built/changed** — Every file created, modified, or deleted
2. **What changed from the plan** — Any deviations from this document. Why?
3. **Env vars required** — Complete list for the migrated app
4. **Code removed** — List of removed auth/billing routes, pages, and files (if any existed)
5. **JWT integration** — How JWT middleware was implemented (library, header format, fields extracted)
6. **Entitlement check** — How entitlement is checked (API call? cached? which endpoint?)
7. **Assumptions made** — Anything you had to decide that wasn't explicitly in the spec
8. **Known gaps** — Anything deferred or issues discovered
9. **Testing steps** — How to verify the SSO migration works

This report helps the orchestrator track which standalone products are fully migrated.

---

## Phase 1 Learnings (Added by Orchestrator — 2026-03-26)

These are real-world implementation details from the MBS Platform build that affect this migration:

### JWT Details (as actually implemented)
- Header: `Authorization: Bearer <token>`
- Payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- `userId` is a **string** (ObjectId.toString())
- Use `jsonwebtoken` (Node.js) or `PyJWT` (Python), algorithm `HS256`, verify with shared `JWT_SECRET`

### Frontend Token Storage
- After login redirect, JWT arrives as `?token=<JWT>` in the URL
- Your frontend must: extract token, store as `localStorage.setItem("mbs_token", token)`, store user as `localStorage.setItem("mbs_user", JSON.stringify(user))`, then `history.replaceState` to remove from URL

### Entitlement Check
- `GET https://magicbusstudios.com/api/entitlements/{your_slug}` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }`
- Cache response for 5 minutes in-memory

### Open Redirect Protection
- The `?redirect=` URL in the login flow is validated against CORS_ORIGINS. Your product's domain MUST be in the MBS Platform's CORS_ORIGINS list, or users will be redirected to magicbusstudios.com instead of your product.
