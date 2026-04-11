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
- Logout clears the stored JWT and optionally calls `POST https://api.magicbusstudios.com/api/auth/logout`

### Step 3: Remove Standalone Billing (If Any)

**Remove:**
- Stripe integration (checkout, webhooks, portal, env vars)
- Pricing pages
- Subscription management UI

**Add:**
- Upgrade/billing buttons redirect to:
  ```
  https://magicbusstudios.com/subscribe?product={PRODUCT_SLUG}&brand=mbs
  ```
- Check access from your backend:
  ```javascript
  const response = await fetch(
    `https://api.magicbusstudios.com/api/entitlements/${PRODUCT_SLUG}`,
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

### Step 5: Handle Legacy User Collision (CRITICAL)

**If this product has existing users in its database**, you MUST handle the case where a platform user ID doesn't match the local user's `_id`. This is the #1 cause of "login appears to fail" bugs.

**The problem:**
1. User `you@gmail.com` exists locally with `_id: ObjectId("abc123...")` (from before SSO)
2. Platform assigns `userId: "def456..."` (a different ID)
3. `User.findById("def456...")` returns `null` — no user with that platform ID
4. Auto-provisioning tries `User.create({ _id: "def456...", email: "you@gmail.com" })` — **CRASHES on unique email index**
5. `/auth/me` returns 500, frontend sees no user → appears not signed in

**The fix — add this to your `requireAuth` or `/auth/me` endpoint:**
```javascript
async function getOrProvisionUser(platformUser) {
  // platformUser = { userId, email, name, avatar } from JWT

  // 1. Try finding by platform ID (fast path — works after first login)
  let user = await User.findById(platformUser.userId);
  if (user) return user;

  // 2. Check for legacy user with same email (migration path)
  if (platformUser.email) {
    const legacyUser = await User.findOne({ email: platformUser.email });
    if (legacyUser) {
      const oldId = legacyUser._id;

      // Preserve user data from legacy record
      const userData = legacyUser.toObject();
      delete userData._id;
      delete userData.__v;

      // Delete old record, recreate with platform _id
      await User.deleteOne({ _id: oldId });
      user = await User.create({
        _id: platformUser.userId,
        ...userData,
        name: platformUser.name || userData.name,
        avatar: platformUser.avatar || userData.avatar,
      });

      // UPDATE ALL RELATED COLLECTIONS that reference the old _id
      // Replace these with your actual collection names:
      const collections = ['sessions', 'posts', 'bookmarks', 'notifications'];
      for (const col of collections) {
        await mongoose.connection.collection(col).updateMany(
          { user_id: oldId.toString() },
          { $set: { user_id: platformUser.userId } }
        );
        // Also check 'userId' field if your collections use camelCase
        await mongoose.connection.collection(col).updateMany(
          { userId: oldId.toString() },
          { $set: { userId: platformUser.userId } }
        );
      }

      return user;
    }
  }

  // 3. Brand new user — create fresh profile
  user = await User.create({
    _id: platformUser.userId,
    email: platformUser.email,
    name: platformUser.name,
    avatar: platformUser.avatar,
  });
  return user;
}
```

**Important:**
- This is a **one-time migration per user** — after the first login, `findById` hits directly
- You MUST update ALL related collections that reference the old `_id`, or data (sessions, posts, progress, achievements, etc.) will be orphaned
- Grep your codebase for `user_id`, `userId`, `user`, `author`, `created_by` fields to find all collections that need updating
- If your product uses MongoDB ObjectId for `_id` (most do), the old ID is an ObjectId and the new one is a string — handle both types in your `updateMany` queries

### Step 6: Update Backend

- Add `requireAuth` middleware to all protected routes
- Add `getOrProvisionUser` (Step 5) to your `/auth/me` or user-loading endpoint
- Remove standalone auth routes
- Remove Stripe/billing routes (if any)
- Add `JWT_SECRET` to environment variables (must match MBS Platform)
- Keep all product-specific routes and logic unchanged

### Step 7: Update Environment Variables

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

## GDPR — Data Deletion (Three Levels)

Data deletion is **layered**, not a single nuclear cascade:

1. **App-level** ("Delete my data from this app") — triggered from within YOUR app's Settings page. Deletes ONLY this app's data from its own database. The user's MBS Platform account, entitlements, and data in other apps stay intact.
2. **Category-level** — triggered from magicbusstudios.com/settings. MBS Platform calls `DELETE /api/user-data` on every app in the category (e.g., all Arcade games or all Studio Works tools).
3. **Full account deletion** — triggered from magicbusstudios.com/settings. MBS Platform calls every product's delete endpoint, then deletes the user record.

**What you MUST implement:** `DELETE /api/user-data` — an authenticated endpoint that deletes all data for `req.user.userId` from your product's database. This endpoint serves BOTH purposes:
- Called by your own app when the user clicks "Delete my data" in Settings
- Called by the MBS Platform during category-level or full account deletion (MBS Layer 1 wired in Session 31 — email confirmation flow)

**Important:** This endpoint must NOT delete the user's MBS Platform account. It only deletes local product data. The user can still log into other apps after deleting their data from yours.

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
- `GET https://api.magicbusstudios.com/api/entitlements/{your_slug}` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }`
- Cache response for 5 minutes in-memory
- **This MUST be wired** — without it, there's no free/premium enforcement

### Open Redirect Protection
- The `?redirect=` URL in the login flow is validated against CORS_ORIGINS. Your product's domain MUST be in the MBS Platform's CORS_ORIGINS list, or users will be redirected to magicbusstudios.com instead of your product.

---

## Phase 3B Learnings (Added by Orchestrator — 2026-03-27)

These are real-world lessons from the CWG migration:

### API URL: Use `api.magicbusstudios.com` NOT `magicbusstudios.com`
The MBS Platform API lives at `https://api.magicbusstudios.com` (the backend container). `https://magicbusstudios.com` is the frontend (nginx). Calling the frontend URL for API requests causes CORS errors. All entitlement checks and API calls must use `api.magicbusstudios.com`.

### After Deleting Auth Files — Grep for ALL Imports
When you delete auth routes and services, other files may still import functions from them causing runtime crashes. **Before deploying, grep for all imports from deleted files.**

### Legacy Auth Routes — Redirect to Platform Login, Not Homepage
When removing `/login`, `/signup` pages, redirect those routes to `https://magicbusstudios.com/auth/login?redirect=https://{YOUR_DOMAIN}` — NOT to `/` (homepage). Users with bookmarks need to reach the platform login.

---

## Phase 4 Learnings (Added by Orchestrator — 2026-03-28)

These are real-world lessons from the FlowState migration:

### MONGO_URL / MONGODB_URI / MONGO_URI — Check All Three
Coolify services may use `MONGO_URL`, `MONGODB_URI`, or `MONGO_URI` depending on when they were set up. Your code should support whichever one exists, or add fallbacks: `process.env.MONGO_URL || process.env.MONGODB_URI || process.env.MONGO_URI`.

### Remove Google GSI Script from index.html
If your `index.html` loads `https://accounts.google.com/gsi/client` (Google Identity Services), remove it — Google SSO is now handled by the MBS Platform. Also update CSP headers in nginx.conf to remove `accounts.google.com` from `script-src`, `style-src`, `connect-src`, and `frame-src`.

### Coolify Env Vars — Separate Lines
When pasting env vars in Coolify, ensure each var is on its own line. A known issue: concatenated env vars cause JWT validation to fail silently.

---

## Phase 5 Learnings (Added by Orchestrator — 2026-03-28)

These are real-world lessons from WildLens and other standalone product SSO migrations:

### Legacy User Collision — The #1 SSO Migration Bug
When an app has existing users, the platform assigns a different `userId` than the local `_id`. The auto-provisioning code tries to `create()` a new user with the platform `_id` but the same email → **unique index crash** → `/auth/me` returns 500 → frontend shows user as not logged in. See Step 5 for the complete fix pattern. This hit WildLens and will hit EVERY standalone app with existing users.

### Python Apps: JWT_SECRET_KEY vs JWT_SECRET
Python frameworks (Flask, FastAPI) commonly use `JWT_SECRET_KEY` in their config files. Coolify has the env var as `JWT_SECRET` (matching the MBS Platform). If your app's `config.py` reads `JWT_SECRET_KEY`, the JWT verification silently fails because the secret is empty/None. Fix: add a fallback in your config: `os.environ.get("JWT_SECRET_KEY") or os.environ.get("JWT_SECRET") or ""`. This caused a login loop in a Python app — user authenticates on platform, gets redirected back, JWT verification fails, gets sent back to login.

### After Legacy Migration — Update ALL Related Collections
When migrating a legacy user's `_id` to the platform ID, you must also update every collection that references the old ID. Grep for fields like `user_id`, `userId`, `user`, `author`, `created_by`, `owner` across all collections. Missing even one causes data loss (orphaned records). WildLens had to update 8+ collections (discoveries, posts, bookmarks, chat sessions, collections, expeditions, notifications, challenges).

### CORS_ORIGINS — Both Platform AND Product Domains
The MBS Platform's `CORS_ORIGINS` must include your product domain (e.g., `https://wildlens.magicbusstudios.com`), otherwise the `?token=` redirect never arrives. Check this BEFORE debugging login issues — it's the second most common cause of "SSO doesn't work".

---

## Phase 6 Learnings (Added by Orchestrator — 2026-04-08)

These are real-world lessons from the AI Tutor RS256 migration:

### Python Apps: `pyjwt[crypto]` Required for RS256
The MBS Platform now signs JWTs with **RS256** (asymmetric). Plain `pyjwt` only supports HS256 — it does NOT include the `cryptography` package needed for RSA operations. If a Python app's `requirements.txt` has `PyJWT==2.x.x` without the `[crypto]` extra, RS256 verification fails with `InvalidAlgorithmError: Algorithm not supported`, falls through to HS256 (which also fails since the token is RS256-signed), and returns 401. The user sees a login loop — authenticates on the platform, gets redirected back, but appears not logged in.

**Fix**: In `requirements.txt`, use `PyJWT[crypto]==2.9.0` (not `PyJWT==2.9.0`). This installs the `cryptography` package as a dependency.

**Symptoms**: Backend logs show `InvalidAlgorithmError: Algorithm not supported` on RS256, then `InvalidAlgorithmError: The specified alg value is not allowed` on HS256 fallback. The `JWT_PUBLIC_KEY` env var IS set and the PEM is valid — the error is a missing dependency, not a key problem.

### RS256 Env Var for Python Apps
Python apps need `JWT_PUBLIC_KEY` in Coolify (multiline PEM format). The auth service should handle `\\n` → `\n` conversion for Coolify's env var storage:
```python
_raw = os.environ.get("JWT_PUBLIC_KEY", "")
JWT_PUBLIC_KEY = _raw.replace("\\n", "\n") if _raw else None
```
