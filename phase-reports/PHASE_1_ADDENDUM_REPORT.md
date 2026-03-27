# Phase 1 Addendum Report — Email/Password Auth + 2FA/TOTP

**Date:** 2026-03-26
**Items completed:** #14 (Email/Password Authentication) and #15 (2FA/TOTP with Backup Codes)

---

## 1. What Was Built

### Backend (server/)

| File | Action | Description |
|------|--------|-------------|
| `server/models/User.js` | Modified | Added password_hash, auth_methods[], email_verified, email_verification_token/expires, password_reset_token/expires, totp_enabled, totp_secret, totp_backup_codes. Removed auth_provider enum. |
| `server/routes/auth.js` | Rewritten | Added 6 email/password routes + 6 2FA routes. Updated Google SSO to use auth_methods. Updated Nostr/LNURL to use auth_methods. |
| `server/middleware/auth.js` | Modified | Added `issueTempToken()` for 2FA temp tokens (5-min expiry, purpose: "2fa"). |
| `server/services/emailService.js` | Modified | Added `sendVerificationEmail()` and `sendPasswordResetEmail()` with branded HTML templates. |
| `server/package.json` | Modified | Added bcryptjs, otplib, qrcode dependencies. |
| `server/package-lock.json` | Regenerated | Synced for Docker `npm ci`. |

### Frontend (src/)

| File | Action | Description |
|------|--------|-------------|
| `src/pages/AuthSignupPage.jsx` | New | Branded signup page with Google SSO + email/password form, age confirmation, terms acceptance. |
| `src/pages/AuthLoginPage.jsx` | Rewritten | Added email/password form, forgot password link, 2FA code input, backup code toggle, ?verified=true banner. |
| `src/pages/AuthResetPasswordPage.jsx` | New | Password reset page that reads ?token= param and submits new password. |
| `src/pages/AccountPage.jsx` | Modified | Added 2FA section (setup QR, verify, disable, regenerate backup codes). Updated profile to show auth_methods instead of auth_provider. Added email verification status + resend button. |
| `src/App.jsx` | Modified | Added /auth/signup and /auth/reset-password routes with defensive loading. |

---

## 2. New API Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/auth/signup | No | Email/password registration |
| POST | /api/auth/login | No | Email/password login (returns requires2FA if TOTP enabled) |
| POST | /api/auth/forgot-password | No | Send password reset email |
| POST | /api/auth/reset-password | No | Set new password with reset token |
| GET | /api/auth/verify-email/:token | No | Verify email address (redirects to login) |
| POST | /api/auth/resend-verification | No | Resend verification email |
| POST | /api/auth/2fa/setup | JWT | Generate TOTP secret + QR code + backup codes |
| POST | /api/auth/2fa/verify-setup | JWT | Confirm TOTP setup with a code |
| POST | /api/auth/2fa/verify | No* | Verify TOTP/backup code during login (*uses tempToken) |
| POST | /api/auth/2fa/disable | JWT | Disable 2FA (requires password) |
| GET | /api/auth/2fa/status | JWT | Check if 2FA is enabled |
| POST | /api/auth/2fa/regenerate-backup-codes | JWT | Generate new backup codes (requires password) |

---

## 3. New Frontend Routes

| Path | Component | Auth-gated? |
|------|-----------|-------------|
| /auth/signup | AuthSignupPage | No |
| /auth/reset-password | AuthResetPasswordPage | No |

(Login page at /auth/login was updated, not a new route.)

---

## 4. Env Vars

**No new env vars required.** The email/password and 2FA features use existing vars:
- `SENDGRID_API_KEY` — for verification and password reset emails
- `FROM_EMAIL` — sender address for all emails
- `JWT_SECRET` — for signing JWTs and temp tokens

---

## 5. User Model Changes

The `auth_provider` field (enum: google/nostr/lnurl) has been replaced by `auth_methods` (array of strings). Existing users created with Google SSO have `auth_provider: "google"` but no `auth_methods` field. The code handles this gracefully — `ensureAuthMethods()` initializes the array if missing.

**Migration note for existing users:** The `/api/auth/me` endpoint now returns `auth_methods` instead of `auth_provider`. Downstream product frontends that display the auth method should use `user.auth_methods` (array). For backward compat, the AccountPage falls back to `user.auth_provider` if `auth_methods` is empty.

---

## 6. Auth Method Linking (Cross-Method)

When a user signs up with email/password and later clicks "Sign in with Google" (same email):
- Existing user found by email → `google_id` added, `"google"` added to `auth_methods`
- No duplicate user created

Same logic in reverse: Google user who signs up with email/password gets `"email"` added to `auth_methods` and `password_hash` set.

---

## 7. 2FA Login Flow

1. User enters email + password → POST /api/auth/login
2. Backend verifies password, sees `totp_enabled: true`
3. Returns `{ success: true, requires2FA: true, tempToken: "..." }`
4. Frontend shows 6-digit TOTP input + "Use backup code" toggle
5. User enters code → POST /api/auth/2fa/verify with `{ tempToken, code }`
6. Backend verifies tempToken (5-min expiry, purpose: "2fa") + TOTP code → returns full JWT
7. Google SSO, Nostr, and LNURL bypass 2FA (inherently strong auth)

**tempToken payload:** `{ userId, purpose: "2fa", iat, exp }` — 5-minute expiry.

---

## 8. Gotchas for Downstream Phases

1. **auth_provider → auth_methods migration**: The User model no longer has `auth_provider` as a required enum field. It's been removed from the schema. Existing documents in MongoDB still have the field — it won't be deleted automatically. Downstream products that read `user.auth_provider` should switch to `user.auth_methods[0]` or check the array.

2. **Email verification not blocking**: Users can use the app immediately after signup. The JWT is issued before email verification. Products should check `user.email_verified` if they need verified emails (e.g., for sensitive operations).

3. **2FA only on email/password login**: Google SSO, Nostr, and LNURL bypass 2FA entirely. If a user has both email and Google auth methods and has 2FA enabled, they can bypass 2FA by using Google Sign-In. This is by design — Google already provides strong auth.

4. **Backup codes are bcrypt-hashed**: The `totp_backup_codes` array contains bcrypt hashes, not plaintext. They're shown to the user once during setup and once during regeneration. They cannot be retrieved again.

5. **Forgot-password always returns success**: To prevent email enumeration, the forgot-password endpoint always returns `{ success: true }` regardless of whether the email exists.

6. **CWG user migration**: When migrating CWG users to the platform, the migration script should set `auth_methods: ["google"]` (not `auth_provider: "google"`) and `email_verified: true` for Google users.

---

## 9. Testing Commands

### Email/Password Signup
```bash
curl -k -X POST https://api.magicbusstudios.com/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"testpass123","age_confirmed":true,"terms_accepted":true}'
```

### Email/Password Login
```bash
curl -k -X POST https://api.magicbusstudios.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123"}'
```

### Forgot Password
```bash
curl -k -X POST https://api.magicbusstudios.com/api/auth/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com"}'
```

### 2FA Status (with JWT)
```bash
curl -k https://api.magicbusstudios.com/api/auth/2fa/status \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### 2FA Setup (with JWT)
```bash
curl -k -X POST https://api.magicbusstudios.com/api/auth/2fa/setup \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

## 10. Known Gaps

- **Rate limiting not added** to the new auth routes (signup, login, forgot-password, resend-verification, 2fa/verify). This was listed as addendum item #2 but not part of this task.
- **Forgot-password page** (at /auth/forgot-password) is linked from the login page but not built as a standalone page. The link exists; the page component does not. Users can access forgot-password functionality via the login page's link which goes to a URL that doesn't have a component — this should be built as a small form page similar to reset-password.

---

## 11. Assumptions Made

1. **Email verification is non-blocking** — JWT issued immediately at signup, verification email sent in background. The spec said "return JWT immediately (user can use app while unverified)."
2. **Google SSO auto-marks email as verified** — since Google already verified the email, `email_verified: true` is set.
3. **Backup codes are 8-char hex strings** (from crypto.randomBytes(4).toString("hex")) — matching the spec's "8-char alphanumeric."
4. **2FA disable requires password** — per spec. Google-only users without a password cannot disable 2FA (they'd need to set a password first via account settings — a future feature).
5. **Signup creates EmailPreference record** — default all opted-in, matching the spec for addendum item #7.
