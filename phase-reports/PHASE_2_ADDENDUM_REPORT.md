# Phase 2 Addendum Report — Inner Lab Auth Pages
**Date:** 2026-03-26
**Branch:** main
**Commit:** 5a09951
**Status:** ✅ FULLY COMPLETE — live and verified

---

## Live Verification (2026-03-26)

| URL | Status | Notes |
|-----|--------|-------|
| `https://innerlab.ai/auth/login` | ✅ Live | Google SSO button showing ("Continue as Abhinav") |
| `https://innerlab.ai/auth/signup` | ✅ Live | Registration form with checkboxes |
| `https://innerlab.ai/auth/forgot-password` | ✅ Live | Email form rendering |
| `https://innerlab.ai/auth/reset-password` | ✅ Live | Redirects to forgot-password (correct — no token present) |
| `https://api.innerlab.ai/health` | ✅ Live | `{"status":"ok","service":"innerlab-middleware"}` |

---

## What Was Built

### 4 New Auth Pages

All pages are Inner Lab branded (teal/dark theme), full-page layout with no nav/footer (rendered outside the Layout component), and call MBS Platform APIs at `https://magicbusstudios.com` (via `VITE_PLATFORM_URL` env var with fallback).

#### 1. `/auth/login` → `src/pages/AuthLoginPage.jsx`
- Google SSO button (Google Identity Services) at top, rendered via `VITE_GOOGLE_CLIENT_ID`
- Email + password form with show/hide toggle
- "Forgot password?" link → `/auth/forgot-password`
- **2FA step:** If MBS Platform responds with `requires2FA: true`, transitions to a second panel:
  - Shows 6-digit TOTP input (numeric keyboard on mobile)
  - Toggle to switch to backup code mode
  - POSTs `{ code, tempToken }` or `{ backupCode, tempToken }` to `/api/auth/2fa/verify`
  - "Back to sign in" link resets to login step
- **URL param banners:**
  - `?verified=true` → success toast "Email verified! You can now sign in."
  - `?reset=true` → success toast "Password reset successfully."
- **`?redirect=` handling:** After login, redirects to the original path (validated same-origin only, falls back to `/dashboard`)
- Stores `mbs_token` and `mbs_user` in localStorage; uses `window.location.href` for redirect to force AuthContext reinit

#### 2. `/auth/signup` → `src/pages/AuthSignupPage.jsx`
- Google SSO button (top)
- Name, email, password, confirm password fields
- Two required checkboxes:
  - Age confirmation (13+)
  - Terms of Service + Privacy Policy links (magicbusstudios.com/terms, /privacy)
- POST to `VITE_PLATFORM_URL/api/auth/signup`
- **Non-blocking email verification:** If token returned → store and redirect to `/dashboard` immediately; if no token → redirect to `/auth/login`

#### 3. `/auth/forgot-password` → `src/pages/AuthForgotPasswordPage.jsx`
- Simple email input form
- POST to `VITE_PLATFORM_URL/api/auth/forgot-password`
- **Always shows generic success** regardless of response — prevents email enumeration
- Success state: "Check your email" card with the submitted address shown

#### 4. `/auth/reset-password` → `src/pages/AuthResetPasswordPage.jsx`
- Reads `?token=` from URL params
- If token is missing → redirects to `/auth/forgot-password` with error toast
- New password + confirm password with show/hide toggles
- POST `{ token, password }` to `VITE_PLATFORM_URL/api/auth/reset-password`
- On success → navigates to `/auth/login?reset=true`

---

## Files Modified

### `src/components/ProtectedRoute.jsx`
**Before:** `window.location.href = ${PLATFORM_URL}/login?redirect=...` → sent users to magicbusstudios.com/login (no such route → 404)
**After:** `window.location.href = /auth/login?redirect=<pathname>` → sends users to Inner Lab's own login page with same-site return path

### `src/App.jsx`
- Added 4 lazy-loaded auth page imports
- Added 4 auth routes **outside** the `<Route element={<Layout />}>` wrapper — auth pages have no nav/footer
- Route placement: auth routes declared before Layout to ensure they match first

---

## Env Vars (Coolify — Innerlab A frontend service)

| Var | Required | Notes |
|-----|----------|-------|
| `VITE_GOOGLE_CLIENT_ID` | ✅ Set | Same Google OAuth client as MBS Platform |
| `VITE_API_URL` | ✅ Already set | `https://api.magicbusstudios.com` — for form submissions |
| `VITE_FORM_SOURCE` | ✅ Already set | `IL` — identifies Inner Lab form submissions |
| `VITE_PLATFORM_URL` | Not needed | Has fallback to `https://magicbusstudios.com` — not required unless Platform URL changes |

> **Note:** `VITE_*` vars are build-time only in Vite/Docker — must be set as build args in Coolify, not runtime env vars.

---

## API Endpoints Called (on MBS Platform)

| Endpoint | Method | Body | Response |
|----------|--------|------|----------|
| `/api/auth/login` | POST | `{ email, password }` | `{ token, user }` or `{ requires2FA: true, tempToken }` |
| `/api/auth/2fa/verify` | POST | `{ code, tempToken }` or `{ backupCode, tempToken }` | `{ token, user }` |
| `/api/auth/google` | POST | `{ credential }` | `{ token, user }` |
| `/api/auth/signup` | POST | `{ name, email, password }` | `{ token?, user? }` |
| `/api/auth/forgot-password` | POST | `{ email }` | Response not used (always show generic success) |
| `/api/auth/reset-password` | POST | `{ token, password }` | Success or error |

---

## Phase 2 Findings (Resolved)

### Coolify "Innerlab B" (backend service)
- **Issue found:** Dockerfile Location was set to `/Dockerfile` (frontend) instead of `/Dockerfile.server`
- **Issue found:** Domain was set to `https://api.innerlab.com` (.com) instead of `https://api.innerlab.ai` (.ai)
- **Resolution:** User fixed both in Coolify and redeployed → backend now live at `api.innerlab.ai`

### DNS
- `api.innerlab.ai` → `72.61.69.223` (Contabo VPS) — correct, no action needed

### JWT Expiry
- Inner Lab backend validates but does NOT issue JWTs — MBS Platform is the only issuer
- Expiry is embedded in the token — no separate expiry config needed on Inner Lab side

### MongoDB Validation Script
- `server/scripts/setupValidation.js` auto-runs on server startup (commit `bf6104a`)
- No manual steps required

### Google Cloud Console
- `https://innerlab.ai` added as authorized JavaScript origin on the shared MBS Platform OAuth client
- No redirect URIs needed — auth pages use Google Identity Services (client-side callback, not server-side OAuth flow)

---

## Complete Status

| Item | Status |
|------|--------|
| Auth pages built (login, signup, forgot-password, reset-password) | ✅ Done |
| ProtectedRoute fixed (was 404-ing to magicbusstudios.com/login) | ✅ Done |
| App.jsx routes added with React.lazy() | ✅ Done |
| Coolify Innerlab B — Dockerfile.server + domain fix + redeploy | ✅ Done |
| Coolify Innerlab A — VITE_GOOGLE_CLIENT_ID build arg added | ✅ Done |
| Google Cloud Console — innerlab.ai authorized as JS origin | ✅ Done |
| Dockerfile.server committed to repo | ✅ Done |
| MongoDB validation auto-runs on server startup | ✅ Done |
| DNS for api.innerlab.ai | ✅ Correct |
| Backend health check live | ✅ `{"status":"ok","service":"innerlab-middleware"}` |
| Frontend auth pages live with Google SSO | ✅ Confirmed on live site |

**Nothing pending. Phase 2 is complete end-to-end.**
