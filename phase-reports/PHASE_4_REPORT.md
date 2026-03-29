# FlowState — Phase 4 Migration Completion Report

**Date**: 2026-03-28
**Session**: 29
**Branch**: dev + main (both at `076addb`)
**Migration**: Standalone auth/billing → MBS Platform + Inner Lab database
**Status**: ✅ COMPLETE — Live-verified on production

---

## 1. What Was Built/Changed

### Backend (Modified)
| File | Changes |
|------|---------|
| `server/auth.js` | **Complete rewrite** — removed Google OAuth (`google-auth-library`, `OAuth2Client`, `verifyGoogleToken`), scrypt password hashing (`hashPassword`, `verifyPassword`), all standalone auth routes (`registerAuthRoutes` with 7 endpoints), token signing (`signToken`, `signResetToken`). Now contains only JWT validation middleware (`requireAuth`, `optionalAuth`) using `jsonwebtoken` HS256 with shared `JWT_SECRET`. Extracts 5 fields from JWT payload: `userId`, `email` (nullable for Nostr/LNURL), `name`, `avatar`, `isAdmin`. |
| `server/index.js` | Changed DB connection from `yogaghost` → `inner_lab` via `MONGO_URL` env var (with `MONGODB_URI` and `MONGO_URI` fallbacks for compatibility). All collection references renamed to `yoga_*` prefix. Removed `registerAuthRoutes(app, db)` and `registerPaymentRoutes(app, db)` calls. Removed Stripe raw body middleware (`express.raw` for `/api/webhook`). Removed auth rate limiter (`authRateLimitMap`, 5 req/min). Removed `ObjectId` import (no longer needed — `user_id` is a string field). Authenticated sync routes now use `{ user_id: req.userId }` string lookup instead of `{ _id: new ObjectId(req.userId) }`. Auto-provisions new user profiles on first sync push via upsert. Email triggers guard `user.email` for null (Nostr/LNURL users). |
| `server/social.js` | Collection renames: `communityFlows` → `yoga_community_flows`, `activity` → `yoga_activity`. Added `product: "flowstate"` field to invites, community flows, and activity entries. Added `source_product: "flowstate"` to friend records. Leaderboard and friend queries now use `yoga_user_profiles` (with `user_id` field) and `yoga_sessions`. User lookup changed from `_id` ObjectId to `user_id` string. Index creation updated for new collection names. |
| `server/email.js` | Removed `sendPasswordResetEmail` function (password reset handled by Inner Lab). Changed collection from `users` → `yoga_user_profiles` in both route handlers and `checkAndSendNotifications` cron. Added null guards on `user.email` throughout (Nostr/LNURL users have no email). Removed password reset from test email route (`case 'reset'` deleted). |
| `server/ai.js` | Import path unchanged (still `./auth.js`), added comment clarifying it's MBS Platform JWT validation. No functional changes — AI routes don't touch DB collections directly. |
| `server/package.json` | Removed `stripe` (^20.3.1) and `google-auth-library` (^10.5.0) dependencies. Kept: `jsonwebtoken` (^9.0.3), `mongodb` (^6.16.0), `express` (^5.1.0), `cors` (^2.8.5), `@sendgrid/mail` (^8.1.6), `openai` (^4.67.0). |
| `server/package-lock.json` | Regenerated after removing stripe/google-auth-library — 720 lines removed (59 packages). |

### Backend (Deleted)
| File | Lines | Reason |
|------|-------|--------|
| `server/payments.js` | 249 lines | Entire Stripe integration removed — billing handled by MBS Platform at `magicbusstudios.com/billing`. Contained: checkout session creation, webhook handler (checkout.session.completed, customer.subscription.updated/deleted), subscription status check, customer portal, free tier constants, `isPremium()` helper. |

### Frontend (Modified)
| File | Changes |
|------|---------|
| `src/App.tsx` | Removed 3 lazy imports (LoginPage, PricingPage, ResetPasswordPage). Added `extractTokenFromURL()` function — on mount, checks URL for `?token=` param from Inner Lab redirect, decodes JWT payload via base64, stores token in `mbs_token` localStorage and user in `mbs_user`, calls `useAuthStore.setAuth()`, then `history.replaceState` to clean URL. Added `ExternalRedirect` component (renders nothing, triggers `window.location.href` in useEffect). Unauthenticated routes: `/login`, `/signup`, `/forgot-password`, `/reset-password` → `ExternalRedirect` to Inner Lab; `/pricing` → `ExternalRedirect` to MBS billing. Authenticated routes: `/pricing` → `ExternalRedirect` to MBS billing; `/reset-password` → `Navigate` to `/home`. Removed old `yogaghost-*` → `mbflow-*` localStorage migration code (no longer needed). |
| `src/store/useAuthStore.ts` | Token key changed from `mbflow-token` → `mbs_token`. Zustand persist key changed from `mbflow-auth` → `mbs-flowstate-auth`. Removed `subscription` state and `fetchSubscription()` action (was calling local `/api/subscription/:userId`). Added `entitlements: EntitlementInfo | null` state, `entitlementsCachedAt: number | null`, and `fetchEntitlements()` action — calls `GET https://api.magicbusstudios.com/api/entitlements/flowstate` with Bearer token, caches result for 5 minutes (`ENTITLEMENT_CACHE_MS = 5 * 60 * 1000`). `isPremium()` now returns `entitlements.hasAccess` instead of checking local subscription object. `logout()` removes `mbs_token` and `mbs_user` from localStorage. On rehydration, auto-calls `fetchEntitlements()` if token exists. |
| `src/lib/api.ts` | Removed 13 functions: `getDeviceId`, `pullSync`, `pushSync`, `pushSession` (device-based sync), `loginWithGoogle`, `signupWithEmail`, `loginWithEmail`, `forgotPassword`, `resetPassword` (standalone auth), `getAuthUser`, `getSubscription`, `createCheckout`, `createPortalSession` (Stripe billing). Removed `SubscriptionInfo` type. Added `EntitlementInfo` type (`{ hasAccess: boolean; reason: string }`). Added `fetchEntitlements()` function calling MBS Platform API. Added `INNER_LAB_LOGIN_URL` and `MBS_BILLING_URL` constants. `getAuthHeader()` now reads from `mbs_token` localStorage key. Kept: `pullSyncAuth`, `pushSyncAuth`, `pushSessionAuth`, `checkHealth`, all AI functions (`getAIDebrief`, `getAIPlan`, `getAITip`), all social functions (`getFriends`, `getLeaderboard`, `createInvite`, `acceptInvite`, `getCommunityFlows`, `getActivityFeed`, `getActivityFeedAuth`), `updateEmailPreferences`. |
| `src/components/PaywallGate.tsx` | Removed `useNavigate` import. Upgrade button changed from `navigate('/pricing')` → `window.location.href = MBS_BILLING_URL`. Removed reference comment `/* must match server/payments.js */`. `isPremium()` still comes from `useAuthStore` — no consumer changes needed. |
| `src/pages/LandingPage.tsx` | All `navigate('/login')` calls (2 instances) → `window.location.href = 'https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com'`. `navigate('/pricing')` (1 instance) → `window.location.href = 'https://magicbusstudios.com/billing'`. `useNavigate` kept — still used for `/privacy`, `/terms`, `/help` links. |
| `src/pages/BreathworkPracticePage.tsx` | Camera mode paywall: `navigate('/pricing')` → `window.location.href = 'https://magicbusstudios.com/billing'`. |
| `src/pages/SettingsPage.tsx` | Removed `subscription` state selector from `useAuthStore`. Old subscription display (showing plan name + upgrade button) replaced with single "Manage Subscription" button → MBS billing URL. Sign In button → `window.location.href` to Inner Lab login URL. |
| `src/pages/SocialPage.tsx` | Sign In button: `navigate('/login')` → `window.location.href` to Inner Lab login URL. |
| `index.html` | Removed Google Identity Services (GSI) script tag: `<script src="https://accounts.google.com/gsi/client" async defer></script>`. |

### Frontend (Deleted)
| File | Lines | Reason |
|------|-------|--------|
| `src/pages/LoginPage.tsx` | 258 lines | Auth handled by Inner Lab — contained Google SSO button, email/password form, forgot password link |
| `src/pages/LoginPage.module.css` | 266 lines | Styles for removed page |
| `src/pages/PricingPage.tsx` | 120 lines | Billing handled by MBS Platform — contained 3 plan cards, Stripe checkout integration |
| `src/pages/PricingPage.module.css` | 153 lines | Styles for removed page |
| `src/pages/ResetPasswordPage.tsx` | 105 lines | Password reset handled by Inner Lab |

### Config (Modified)
| File | Changes |
|------|---------|
| `Dockerfile` | Removed `ARG VITE_GOOGLE_CLIENT_ID` and `ENV VITE_GOOGLE_CLIENT_ID=$VITE_GOOGLE_CLIENT_ID` build arg lines. Build no longer requires any Vite build args. |
| `nginx.conf` | CSP `script-src`: removed `https://accounts.google.com`. CSP `style-src`: removed `https://accounts.google.com`. CSP `connect-src`: removed `https://accounts.google.com https://www.googleapis.com`, added `https://api.magicbusstudios.com`. CSP `frame-src`: removed entire directive (was `frame-src https://accounts.google.com`). |
| `.env.example` | `MONGODB_URI` → `MONGO_URL`. `DB_NAME` default → `inner_lab`. Removed 7 vars: `GOOGLE_CLIENT_ID`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_PRICE_MONTHLY`, `STRIPE_PRICE_YEARLY`, `STRIPE_PRICE_LIFETIME`, `VITE_GOOGLE_CLIENT_ID`. Added: `MBS_PLATFORM_API=https://api.magicbusstudios.com`, `INNER_LAB_LOGIN_URL=https://innerlab.ai/auth/login`. Updated `SENDGRID_FROM` default to `noreply@magicbusstudios.com`. |

### Net Code Change
- **28 files changed** across 3 commits
- **814 insertions, 2,065 deletions** (net -1,251 lines)
- **5 files deleted** (LoginPage, PricingPage, ResetPasswordPage + CSS modules, payments.js)

---

## 2. What Changed from the Plan

| Deviation | Reason |
|-----------|--------|
| Removed Google GSI script from `index.html` | Not in original migration spec but necessary — Google SSO is no longer handled locally. Without removal, browser would load unused 200KB script and CSP would need to allow `accounts.google.com`. |
| `friends` and `invites` collections kept in `inner_lab` DB (not moved to `mbs_platform`) | FlowState backend only connects to one MongoDB database (`inner_lab`). Moving these to `mbs_platform` would require a second DB connection. Added `product: "flowstate"` and `source_product: "flowstate"` fields for future cross-product separation. |
| Did not implement `il_user_wellness_profiles` extraction | Deferred — 0 users, no data to extract. The Inner Lab middleware layer (Layer 2) is not yet ready for shared wellness data. `healthConditions` and `injuries` remain in `yoga_user_profiles` only. |
| `useSync.ts` unchanged | Already used only authenticated sync functions (`pullSyncAuth`/`pushSyncAuth`). The token key change is handled at the `useAuthStore` layer — the sync hook reads the token from the store, not directly from localStorage. |
| Added `MONGO_URI` fallback (not in plan) | Discovered during deployment that Coolify had `MONGO_URI` set (not `MONGO_URL` or `MONGODB_URI`). Added 3-way fallback: `MONGO_URL || MONGODB_URI || MONGO_URI`. |
| Regenerated `server/package-lock.json` (not in plan) | Old lockfile still referenced `stripe` and `google-auth-library` packages. Docker build's `npm install --production` could fail or install orphaned packages. Regenerated cleanly (-720 lines). |

---

## 3. Migration Results

- **Users migrated**: 0 (no real users existed in `yogaghost` database)
- **Collections copied**: 0 (no data migration script needed)
- **Errors during migration**: 0
- **Skipped records**: 0
- **Old database preserved**: Yes — `yogaghost` database untouched as backup

The migration is code-only — the new collection names (`yoga_*`) are auto-created on first write when a user signs in through the MBS Platform. Verified: first login via Inner Lab → sync push → `yoga_user_profiles` document created with `user_id: "69c53401fe8f1763b9046ae5"`.

---

## 4. Collection Mapping

| Old Collection | New Collection | Database | Index Changes |
|---------------|---------------|----------|---------------|
| `users` | `yoga_user_profiles` | `inner_lab` | Primary: `{ user_id: 1 }` unique. Legacy: `{ deviceId: 1 }` sparse. Kept: `{ updatedAt: 1 }`. Removed: `{ googleId: 1 }`, `{ email: 1 }` (identity fields now in `mbs_platform.users`). |
| `sessions` | `yoga_sessions` | `inner_lab` | Primary: `{ user_id: 1, timestamp: -1 }`. TTL: `{ createdAt: 1 }` (365 days). Legacy: `{ deviceId: 1, timestamp: -1 }`. |
| `achievements` | `yoga_achievements` | `inner_lab` | Primary: `{ user_id: 1, achievementId: 1 }` unique sparse. Legacy: `{ deviceId: 1, achievementId: 1 }` sparse. |
| `communityFlows` | `yoga_community_flows` | `inner_lab` | Kept: `{ publishedAt: -1 }`. Added field: `product: "flowstate"`. |
| `activity` | `yoga_activity` | `inner_lab` | Kept: `{ userId: 1, createdAt: -1 }`. Added field: `product: "flowstate"`. |
| `friends` | `friends` (unchanged) | `inner_lab` | Kept: `{ users: 1 }`. Added field: `source_product: "flowstate"`. |
| `invites` | `invites` (unchanged) | `inner_lab` | Kept: `{ code: 1 }` unique, `{ expiresAt: 1 }` TTL. Added field: `product: "flowstate"`. |

---

## 5. Field Renames

| Old Field | New Field | Where | Reason |
|-----------|-----------|-------|--------|
| `userId` | `user_id` | `yoga_sessions`, `yoga_achievements` | Consistent with MBS Platform's GDPR cascade delete pattern (searches `inner_lab` for `user_id` field) |
| `_id` ObjectId lookup | `user_id` string lookup | `yoga_user_profiles` | Platform JWT `userId` is a string, not a MongoDB ObjectId. Profile documents now keyed by `user_id` string field, not `_id`. |
| `picture` | `avatar` | JWT payload → `req.userAvatar`, social/leaderboard responses | Aligns with MBS Platform user model which uses `avatar` not `picture` |
| (new field) | `product: "flowstate"` | `yoga_activity`, `yoga_community_flows`, `invites` | Enables future cross-product queries when multiple modules share the database |
| (new field) | `source_product: "flowstate"` | `friends` | Tracks which product originated the friendship |

---

## 6. Env Vars Required

### Active Env Vars (11 total)
| Variable | Type | Required | Default | Notes |
|----------|------|----------|---------|-------|
| `MONGO_URL` | Runtime | Yes | `mongodb://localhost:27017` | Accepts `MONGODB_URI` or `MONGO_URI` as fallback |
| `DB_NAME` | Runtime | No | `inner_lab` | Must be `inner_lab` (with underscore) |
| `JWT_SECRET` | Runtime | Yes | — | **Must match MBS Platform's `JWT_SECRET`** |
| `API_PORT` | Runtime | No | `3001` | Express server port |
| `CORS_ORIGIN` | Runtime | Yes | `*` | Comma-separated origins (e.g., `https://yoga.magicbusstudios.com`) |
| `APP_URL` | Runtime | No | `https://yoga.magicbusstudios.com` | Used in email links |
| `SENDGRID_API_KEY` | Runtime | No | — | Email features disabled if not set |
| `SENDGRID_FROM` | Runtime | No | `noreply@magicbusstudios.com` | Verified sender in SendGrid |
| `OPENAI_API_KEY` | Runtime | No | — | AI coaching disabled if not set |
| `AI_MODEL_MAIN` | Runtime | No | `gpt-4o-mini` | Model for debrief/plan |
| `AI_MODEL_FAST` | Runtime | No | `gpt-4o-mini` | Model for quick tips/corrections |

### Removed Env Vars (8 total)
| Variable | Type | Reason |
|----------|------|--------|
| `GOOGLE_CLIENT_ID` | Runtime | Google SSO handled by Inner Lab |
| `VITE_GOOGLE_CLIENT_ID` | Build arg | Google GSI script removed from index.html |
| `STRIPE_SECRET_KEY` | Runtime | Billing handled by MBS Platform |
| `STRIPE_WEBHOOK_SECRET` | Runtime | Webhook handler deleted |
| `STRIPE_PRICE_MONTHLY` | Runtime | Stripe products managed at platform level |
| `STRIPE_PRICE_YEARLY` | Runtime | Stripe products managed at platform level |
| `STRIPE_PRICE_LIFETIME` | Runtime | Stripe products managed at platform level |
| `VITE_GOOGLE_CLIENT_ID` | Coolify build arg | No longer used in Dockerfile |

---

## 7. Code Removed

### Auth Routes Removed (7 endpoints)
| Method | Path | Function | Lines |
|--------|------|----------|-------|
| `POST` | `/api/auth/google` | Google SSO token exchange → JWT | ~45 |
| `POST` | `/api/auth/signup` | Email/password registration with scrypt | ~50 |
| `POST` | `/api/auth/login` | Email/password login with timing-safe compare | ~25 |
| `POST` | `/api/auth/forgot-password` | Password reset email (anti-enumeration) | ~30 |
| `POST` | `/api/auth/reset-password` | Reset password with nonce-based single-use token | ~40 |
| `GET` | `/api/auth/me` | Current user profile | ~15 |
| `POST` | `/api/auth/logout` | Logout (client-side, server logged for analytics) | ~3 |

### Stripe Routes Removed (4 endpoints)
| Method | Path | Function | Lines |
|--------|------|----------|-------|
| `POST` | `/api/checkout` | Create Stripe Checkout session | ~40 |
| `POST` | `/api/webhook` | Stripe webhook (checkout.completed, sub.updated, sub.deleted) | ~60 |
| `GET` | `/api/subscription/:userId` | Subscription status check with IDOR protection | ~15 |
| `POST` | `/api/portal` | Create Stripe Customer Portal session | ~20 |

### Frontend Pages Removed (5 files, 902 lines)
| File | Lines | What It Contained |
|------|-------|-------------------|
| `LoginPage.tsx` | 258 | Google SSO button (GSI One Tap), email/password form, signup form, forgot password link, device ID migration |
| `LoginPage.module.css` | 266 | Auth page styling (split layout, form cards, social buttons) |
| `PricingPage.tsx` | 120 | 3-tier pricing cards (monthly/yearly/lifetime), Stripe checkout redirect, current plan display |
| `PricingPage.module.css` | 153 | Pricing grid layout, plan cards, feature lists |
| `ResetPasswordPage.tsx` | 105 | Token verification, new password form, success redirect |

### API Functions Removed (13 functions)
| Function | Purpose | Replaced By |
|----------|---------|-------------|
| `getDeviceId()` | Generate/retrieve device ID for anonymous sync | Removed — auth required |
| `pullSync(deviceId)` | Pull data by device ID | `pullSyncAuth()` (already existed) |
| `pushSync(deviceId, data)` | Push data by device ID | `pushSyncAuth(data)` (already existed) |
| `pushSession(deviceId, session)` | Push single session by device ID | `pushSessionAuth(session)` (already existed) |
| `loginWithGoogle(idToken, deviceId)` | Exchange Google ID token for local JWT | Inner Lab handles login |
| `signupWithEmail(email, password, name, deviceId)` | Create local account | Inner Lab handles signup |
| `loginWithEmail(email, password)` | Authenticate with local credentials | Inner Lab handles login |
| `forgotPassword(email)` | Request password reset email | Inner Lab handles password reset |
| `resetPassword(token, newPassword)` | Reset password with token | Inner Lab handles password reset |
| `getAuthUser()` | Fetch current user from `/api/auth/me` | JWT payload contains user info |
| `getSubscription(userId)` | Check local subscription status | `fetchEntitlements()` via MBS Platform API |
| `createCheckout(userId, plan)` | Create Stripe checkout session | Redirect to `magicbusstudios.com/billing` |
| `createPortalSession(userId)` | Open Stripe customer portal | Redirect to `magicbusstudios.com/billing` |

### Dependencies Removed (2 packages)
| Package | Version | Size Impact |
|---------|---------|-------------|
| `stripe` | ^20.3.1 | ~2.5MB installed |
| `google-auth-library` | ^10.5.0 | ~1.8MB installed |

---

## 8. JWT Integration

### Implementation Details
- **Library**: `jsonwebtoken` (^9.0.3) — was already a dependency, no new install
- **Algorithm**: HS256 (HMAC-SHA256), enforced via `{ algorithms: ['HS256'] }` option
- **Header format**: `Authorization: Bearer <JWT>`
- **Secret**: `JWT_SECRET` env var — **must be identical to the MBS Platform's `JWT_SECRET`**
- **Validation**: `jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] })` — rejects tokens signed with other algorithms

### Payload Fields Extracted
| JWT Field | Express Property | Type | Notes |
|-----------|-----------------|------|-------|
| `userId` | `req.userId` | `string` | ObjectId.toString() from MBS Platform — e.g., `"69c53401fe8f1763b9046ae5"` |
| `email` | `req.userEmail` | `string \| null` | **Can be null** for Nostr/LNURL users |
| `name` | `req.userName` | `string` | Display name |
| `avatar` | `req.userAvatar` | `string \| null` | Google profile photo URL or null |
| `isAdmin` | `req.isAdmin` | `boolean` | Platform admin flag |

### Frontend Token Flow
1. User clicks "Get Started" on landing page
2. Browser redirects to `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com`
3. User authenticates via Inner Lab (Google SSO, email/password, Nostr, or LNURL)
4. Inner Lab redirects back: `https://yoga.magicbusstudios.com/?token=<JWT>`
5. `extractTokenFromURL()` in `App.tsx` runs on mount:
   - Extracts `?token=` from URL search params
   - Decodes JWT payload via base64url (no verification — server already verified)
   - Calls `useAuthStore.setAuth(token, user)` — stores in Zustand + `localStorage.setItem('mbs_token', token)`
   - Stores user JSON in `localStorage.setItem('mbs_user', JSON.stringify(user))`
   - Cleans URL with `history.replaceState({}, '', pathname + hash)`
6. React re-renders — `isAuthenticated` becomes true, app shows authenticated view

### Middleware Exports
- `requireAuth(req, res, next)` — Returns 401 if no valid Bearer token. Used on all `/api/user/*`, `/api/social/*`, `/api/ai/*`, `/api/email/*` routes.
- `optionalAuth(req, _res, next)` — Extracts user from token if present, continues without auth if not. Available but currently unused.

---

## 9. deviceId Sunset

### Status: Kept for 90-day sunset window (remove after ~2026-06-27)

### Routes Preserved (3 endpoints)
| Method | Path | Deprecation Warning |
|--------|------|-------------------|
| `GET` | `/api/sync/:deviceId` | `DEPRECATED: device-based sync pull — will be removed after 90-day sunset` |
| `POST` | `/api/sync/:deviceId` | `DEPRECATED: device-based sync push — will be removed after 90-day sunset` |
| `POST` | `/api/sessions/:deviceId` | `DEPRECATED: device-based session push — will be removed after 90-day sunset` |

### What Changed
- All three routes now log `logger.warn('DEPRECATED: ...')` on every call
- Collection references updated to new names (`yoga_sessions`, `yoga_achievements`, `yoga_user_profiles`)
- Routes still require `requireAuth` middleware (as they did pre-migration)

### What Stayed the Same
- `isValidDeviceId()` helper function preserved
- Legacy `deviceId` indexes kept as sparse indexes
- Route logic unchanged (just collection names updated)

### Frontend Changes
- `getDeviceId()` function deleted from `api.ts`
- `mbflow-device-id` localStorage key no longer created
- No frontend code calls device-based sync endpoints anymore

### After 90 Days (TODO)
1. Remove all three device-based routes from `server/index.js`
2. Remove `isValidDeviceId()` helper
3. Drop sparse `deviceId` indexes from `yoga_user_profiles`, `yoga_sessions`, `yoga_achievements`
4. Remove `deviceId` field from `yoga_user_profiles` upsert logic

---

## 10. Assumptions Made

1. **0 real users** — No email-based user resolution layer needed. New profiles are auto-provisioned on first authenticated sync push via upsert on `{ user_id: req.userId }`. Verified: first login created `yoga_user_profiles` doc with platform user ID.

2. **Keep native MongoDB driver** — The migration spec said "Agent decides" on Mongoose vs native driver. Kept native `mongodb` driver for consistency with existing codebase, avoiding a second migration within the migration. All 6 server files use the native driver consistently.

3. **Embedded arrays kept** — Per migration spec recommendation (Option A), breathwork sessions, meditation sessions, flow sessions, breathwork achievements, active programs, and completed programs remain as embedded arrays inside `yoga_user_profiles` documents. No extraction to separate collections. This is appropriate given 0 users and no scale concern.

4. **`friends` and `invites` stay in `inner_lab`** — FlowState backend connects to only one database. Moving these collections to `mbs_platform` would require a second MongoDB connection. Added `product` / `source_product` fields for future cross-product separation if needed.

5. **`il_user_wellness_profiles` deferred** — No shared wellness data collection created. `healthConditions` and `injuries` remain in `yoga_user_profiles` only. Will be extracted when Inner Lab middleware (Layer 2) is ready to consume shared wellness data.

6. **No migration script** — Code-only refactor. Zero users means zero data to migrate. Collections are auto-created by MongoDB on first write. Old `yogaghost` database left untouched as backup.

7. **Entitlement check returns `free_tier`** — Until Stripe products are configured for FlowState on the MBS Platform, all users receive `{ hasAccess: true, reason: "free_tier" }`. This is correct behavior — free tier access is unrestricted.

8. **Login redirect URL hardcoded to production** — `INNER_LAB_LOGIN_URL` in `api.ts` points to `https://yoga.magicbusstudios.com` (production). For dev site testing, users go through the same login flow and get redirected to production. This is acceptable since both sites share the same codebase and database.

9. **Service worker cache invalidation** — First deploy after migration requires users to hard-refresh or wait for service worker update. The old service worker caches reference deleted assets (LoginPage CSS). This is a one-time issue that resolves automatically when `vite-plugin-pwa` updates the service worker manifest.

---

## 11. Known Gaps

| # | Gap | Impact | Severity | When to Address |
|---|-----|--------|----------|----------------|
| 1 | `il_user_wellness_profiles` not implemented | `healthConditions` + `injuries` stay in `yoga_user_profiles` only, not shared across Inner Lab modules | Low | When Inner Lab middleware (Layer 2) is ready |
| 2 | `friends`/`invites` not in `mbs_platform` DB | Social features are FlowState-scoped, not cross-product | Low | When MBS Platform adds social features |
| 3 | `name`/`email` not persisted to `yoga_user_profiles` from JWT | User profile display relies on JWT payload data; profile doc only has practice data | Low | Could store on first sync push; unnecessary with 0 users |
| 4 | Service worker caches old assets after first deploy | Users see `LoginPage-*.css` preload error until hard-refresh | Low (self-resolving) | Clears automatically when SW updates; manual clear with DevTools |
| 5 | Electron wrapper broken (pre-existing) | `electron-builder` not in devDependencies | Low | Separate task (pre-dates migration) |
| 6 | Login redirect hardcoded to production URL | Dev site login goes through production redirect | Low | Could make configurable via env var in future |
| 7 | `yogaghost-*` localStorage migration removed | Users with very old localStorage keys (pre-rebrand) won't auto-migrate | None | 0 real users affected |
| 8 | HelpPage and TermsPage still mention "subscription" in text | User-facing text references subscriptions generically | None | Cosmetic — accurate since subscriptions exist via MBS Platform |

---

## 12. Testing Steps — Live Verification Results

### Prerequisites Completed
| Step | Status | Notes |
|------|--------|-------|
| `JWT_SECRET` matches MBS Platform | ✅ Done | Set in Coolify |
| `MONGO_URL` / `MONGO_URI` pointing to shared MongoDB | ✅ Done | Using existing connection string |
| `DB_NAME=inner_lab` | ✅ Done | Changed from `innerlab` (no underscore) to `inner_lab` |
| Removed `GOOGLE_CLIENT_ID` | ✅ Done | Deleted from Coolify env vars |
| Removed `VITE_GOOGLE_CLIENT_ID` build arg | ✅ Done | Removed from Coolify |

### Auth Flow — Live Test (2026-03-28)
| # | Step | Expected | Actual | Status |
|---|------|----------|--------|--------|
| 1 | Visit `yogadev.magicbusstudios.com` | Landing page | Landing page with Unsplash background, "Get Started" button | ✅ |
| 2 | Click "Get Started" | Redirect to Inner Lab | `https://innerlab.ai/auth/login?redirect=https://yoga.magicbusstudios.com` | ✅ |
| 3 | Sign in via Google SSO | "Continue as Abhinav" button works | Clicked, authenticated via Google | ✅ |
| 4 | Redirect back with token | `?token=<JWT>` in URL | `yoga.magicbusstudios.com/?token=eyJhbGciOiJIUzI1NiIs...` | ✅ |
| 5 | Token extracted | `mbs_token` in localStorage | `eyJhbGciOiJIUzI1NiIs...` (JWT stored) | ✅ |
| 6 | User stored | `mbs_user` in localStorage | `{"id":"69c53401fe8f1763b9046ae5","email":"1984.abhinav@gmail.com","name":"Abhinav Gupta","picture":"https://lh3.googleusercontent.com/..."}` | ✅ |
| 7 | URL cleaned | No `?token=` in URL | Redirected to `/onboarding` (clean URL) | ✅ |
| 8 | Onboarding shown | New user sees onboarding | Onboarding page rendered (first login in new DB) | ✅ |
| 9 | After onboarding | Authenticated home page | Home page with sidebar, daily insight, pose cards | ✅ |

### Legacy Route Redirects — Live Test
| Route | Expected Destination | Actual | Status |
|-------|---------------------|--------|--------|
| `/login` | Inner Lab login | `innerlab.ai/auth/login?redirect=...` | ✅ |
| `/signup` | Inner Lab login | `innerlab.ai/auth/login?redirect=...` | ✅ |
| `/reset-password` | Inner Lab login | `innerlab.ai/auth/login?redirect=...` | ✅ |
| `/forgot-password` | Inner Lab login | `innerlab.ai/auth/login?redirect=...` | ✅ |
| `/pricing` | MBS billing | `magicbusstudios.com/billing` | ✅ |

### API Endpoints — Live Test
| Endpoint | Method | Status | Response |
|----------|--------|--------|----------|
| `/api/health` | GET | 200 | `{"status":"ok","db":"connected"}` |
| `/api/user/sync` | GET | 200 | Sync pull with user profile data |
| `/api/user/sync` | POST | 200 | Sync push accepted |
| `api.magicbusstudios.com/api/entitlements/flowstate` | GET | 200 | Entitlements check returned successfully |

### Settings Page — Live Test
| Element | Expected | Actual | Status |
|---------|----------|--------|--------|
| Account section | Shows email | "Signed in as 1984.abhinav@gmail.com" | ✅ |
| Manage Subscription button | Present, links to billing | "Manage Subscription" button visible | ✅ |
| Sign Out button | Present | "Sign Out" button visible | ✅ |
| Cloud Sync | Shows sync status | "Synced 11:33:20 PM" + email | ✅ |
| Email Notifications | Toggles visible | Streak Reminders, Weekly Digest, Product Updates | ✅ |
| No pricing/login references | No old auth UI | No "Upgrade" with plan name, no "Sign In" with local form | ✅ |

### CSP Headers — Live Test
| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| `accounts.google.com` removed | Not in CSP | Not found in CSP header | ✅ |
| `api.magicbusstudios.com` added | In connect-src | Found in connect-src | ✅ |

### Console Errors — Live Test
| Error | Cause | Severity |
|-------|-------|----------|
| `Unable to preload CSS for /assets/LoginPage-gdVi23in.css` | Stale service worker serving old JS bundle that references deleted LoginPage CSS | **Low** — self-resolving after SW update or hard-refresh. Not a code bug. |

### Build Verification — Local Test
| Check | Command | Result |
|-------|---------|--------|
| TypeScript compilation | `npx tsc -b` | **Zero errors** |
| Stale imports from deleted files | `grep -r "LoginPage\|PricingPage\|ResetPasswordPage" src/` | **No matches** |
| Old localStorage keys | `grep -r "mbflow-token\|mbflow-device-id" src/` | **No matches** |
| Old API functions | `grep -r "loginWithGoogle\|signupWithEmail\|..." src/` | **No matches** |
| Old collection names in server | `grep collection('users'\|'sessions'\|'achievements')` | **No matches** (only `yoga_*` prefixed) |
| Stale server imports | `grep -r "import.*payments" server/` | **No matches** |
| Untracked source files | `git ls-files --others --exclude-standard src/ server/` | **None** |
| Server startup | `node -e "import('./server/index.js')"` | **Starts cleanly**, connects to `inner_lab` DB |
| Server npm install | `npm install --production` in server/ | **Clean** — 59 packages removed (stripe/google-auth-library) |

### Code Audit (Automated Agent — 12 checks)
| # | Audit Check | Result |
|---|-------------|--------|
| 1 | Old collection names in server files | ✅ PASS — all `yoga_*` prefixed |
| 2 | Imports from deleted files | ✅ PASS — none found |
| 3 | Old localStorage keys (`mbflow-token`, `mbflow-device-id`) | ✅ PASS — replaced with `mbs_token` |
| 4 | Old API function references | ✅ PASS — none found |
| 5 | `useSync.ts` imports valid | ✅ PASS — `pullSyncAuth`/`pushSyncAuth`/`checkHealth` exist |
| 6 | `SubscriptionInfo` type removed | ✅ PASS — replaced with `EntitlementInfo` |
| 7 | `ObjectId` removed from `index.js` | ✅ PASS — all `user_id` lookups use strings |
| 8 | Sidebar/TopHeader clean | ✅ PASS — no removed functionality |
| 9 | Google GSI script removed from `index.html` | ✅ PASS |
| 10 | OnboardingPage clean | ✅ PASS — no auth imports |
| 11 | Email test route `reset` case removed | ✅ PASS — only 6 valid types |
| 12 | nginx CSP no Google OAuth | ✅ PASS — `api.magicbusstudios.com` added |

---

## 13. Commits

| # | Hash | Message | Files | Lines |
|---|------|---------|-------|-------|
| 1 | `cbb9ab0` | `feat: Migrate FlowState to MBS Platform auth + Inner Lab database` | 28 files | +814 / -2065 |
| 2 | `e90d1ed` | `fix: Add MONGO_URI fallback for env var compatibility` | 1 file | +3 / -3 |
| 3 | `076addb` | `chore: Regenerate server package-lock after removing stripe/google-auth-library` | 1 file | +1 / -720 |

**Both branches** (`dev` and `main`) are at commit `076addb`.

---

## 14. Deployment Notes

### Coolify Configuration Changes Made
| Setting | Before | After |
|---------|--------|-------|
| `GOOGLE_CLIENT_ID` env var | Set | **Deleted** |
| `VITE_GOOGLE_CLIENT_ID` build arg | Set | **Deleted** |
| `DB_NAME` | `innerlab` | `inner_lab` |
| `MONGO_URI` | Set | Unchanged (code now accepts it as fallback) |

### Post-Deployment Issues Encountered
1. **502 Bad Gateway** after first deploy — caused by `MONGO_URI` env var name not being recognized (code only checked `MONGO_URL` and `MONGODB_URI`). Fixed by adding `MONGO_URI` fallback in commit `e90d1ed`.
2. **Service worker cache** serving old JS bundle — caused `LoginPage-*.css` preload error. Resolved by clearing service worker via DevTools. Self-resolves for end users when SW updates.
3. **`DB_NAME=innerlab`** (missing underscore) — didn't cause crash but wrong database name. Fixed by user updating Coolify env var to `inner_lab`.

### Production URLs
| Environment | URL | Branch | Status |
|-------------|-----|--------|--------|
| Dev | `https://yogadev.magicbusstudios.com` | dev | ✅ Live |
| Prod | `https://yoga.magicbusstudios.com` | main | ✅ Live |
| Inner Lab Login | `https://innerlab.ai/auth/login` | — | ✅ Accepting redirect |
| MBS Billing | `https://magicbusstudios.com/billing` | — | ✅ Live |
| Entitlements API | `https://api.magicbusstudios.com/api/entitlements/flowstate` | — | ✅ Returning 200 |
