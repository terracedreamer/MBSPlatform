# Phase 1 Learnings — Impact on Downstream Phases

**Date**: March 26, 2026
**Source**: MBS Platform (Layer 1) build + live testing

---

## 1. JWT Payload (CONFIRMED — live code)

```javascript
{
  userId: "ObjectId as string",   // user._id.toString()
  email: "user@email.com",       // may be null for Nostr/LNURL users
  name: "User Name",
  avatar: "https://..." || null,
  isAdmin: false,                 // boolean
  iat: 1711468800,                // issued at (auto)
  exp: 1712073600                 // expires (7d default)
}
```

**Key: `userId` (camelCase, string).** All downstream phases must use `req.user.userId` after JWT verification, NOT `req.user.user_id` or `req.user.id`.

---

## 2. Auth Header Format (CONFIRMED)

```
Authorization: Bearer <JWT>
```

Token verification: `jwt.verify(token, process.env.JWT_SECRET)` — same HS256 shared secret.

---

## 3. User.email Is Now OPTIONAL (CHANGED from original spec)

The User model was changed from `email: required` to `email: sparse unique` to support pseudonymous Nostr/LNURL users. **Downstream phases must handle users with null email:**
- Don't assume user has email when sending notifications
- Don't use email as a unique identifier across systems — use `userId` (ObjectId)
- Email preference system skips users with no email

---

## 4. Token Extraction Pattern (CONFIRMED — live frontend)

After SSO redirect back, the frontend:
1. Reads `?token=` from URL
2. Stores in `localStorage.setItem("mbs_token", token)`
3. Calls `window.history.replaceState(null, '', cleanUrl)` to remove token from URL

All product frontends doing SSO redirect must implement this same pattern.

---

## 5. BTCPay Status: 403 — Permission Issue (OPEN ISSUE)

**BTCPay API key needs additional permissions:**
- Missing: `btcpay.store.canviewstoresettings`
- Also needs: `btcpay.store.cancreateinvoice`, `btcpay.store.canviewinvoices`

**Impact**: Lightning payments don't work on MBS or CWG until the API key is updated in BTCPay Server admin. This is an infrastructure fix, not a code fix.

**Action**: Owner must regenerate BTCPay API key with full store permissions, then update `BTCPAY_API_KEY` in Coolify for both MBS and CWG backend services.

---

## 6. Stripe Price ID Resolution (CHANGED from original spec)

The billing route does NOT expect `priceId` from the frontend. Instead:
- Frontend sends: `{ product: "cwg", type: "product_pass", period: "monthly" }`
- Backend resolves price ID from env vars:
  - `STRIPE_MONTHLY_PRICE_ID` / `STRIPE_ANNUAL_PRICE_ID` (CWG)
  - `STRIPE_IL_MONTHLY_PRICE_ID` / `STRIPE_IL_ANNUAL_PRICE_ID` (IL bundle)
  - `STRIPE_MBS_MONTHLY_PRICE_ID` / `STRIPE_MBS_ANNUAL_PRICE_ID` (MBS bundle)

**Impact on CWG migration**: When CWG removes its own Stripe integration, the platform billing handles it. CWG's existing Stripe price IDs are already set in MBS backend env vars.

---

## 7. Rate Limiting (NEW — not in original spec)

Three tiers applied at MBS Platform:
- General: 100 req/15min on all /api routes
- Auth: 20 req/15min on /api/auth (stacks with general)
- Billing: 30 req/15min on /api/billing (stacks with general)

**Impact on downstream**: Modules calling the platform API should implement retry with backoff if they get 429. Entitlement checks should be cached (5min TTL as spec'd).

---

## 8. Entitlement Check URL Pattern (CONFIRMED)

```
GET https://magicbusstudios.com/api/entitlements/{product_slug}
Headers: Authorization: Bearer <JWT>

Response:
{ success: true, hasAccess: true/false, reason: "product_pass"|"category_access"|"mbs_all_access"|"free_tier"|"none" }
```

---

## 9. Routes Actually Deployed (complete list)

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| POST | /api/auth/google | No | Google SSO login |
| POST | /api/auth/nostr/challenge | No | Get Nostr challenge |
| POST | /api/auth/nostr | No | Verify Nostr signature |
| GET | /api/auth/lnurl | No | LNURL-auth challenge |
| GET | /api/auth/lnurl/callback | No | Wallet callback |
| GET | /api/auth/lnurl/poll | No | Frontend polls for auth result |
| POST | /api/auth/refresh | Yes | Refresh JWT |
| GET | /api/auth/me | Yes | Current user + entitlements |
| POST | /api/auth/logout | Yes | Invalidate sessions |
| POST | /api/auth/link-nostr | Yes | Link Nostr to existing account |
| POST | /api/auth/link-lnurl | Yes | Link LNURL to existing account |
| DELETE | /api/auth/account | Yes | GDPR cascade delete |
| GET | /api/entitlements/:product | Yes | Check access |
| GET | /api/entitlements/category/:cat | Yes | Products in category |
| POST | /api/billing/checkout | Yes | Stripe checkout |
| POST | /api/billing/portal | Yes | Stripe customer portal |
| GET | /api/billing/history | Yes | Transaction list |
| POST | /api/billing/webhook | No* | Stripe webhook |
| POST | /api/billing/btcpay/checkout | Yes | BTCPay Lightning checkout |
| POST | /api/billing/btcpay/webhook | No* | BTCPay webhook |
| GET | /api/friends | Yes | List friends |
| POST | /api/friends/invite | Yes | Create invite |
| POST | /api/friends/accept | Yes | Accept invite |
| GET | /api/email-preferences | Yes | Get preferences |
| PUT | /api/email-preferences | Yes | Update preferences |
| GET | /api/email-preferences/unsubscribe | No | One-click unsubscribe |
| GET | /api/admin/stats | Admin | Dashboard stats |
| GET | /api/admin/users | Admin | User list |
| GET | /api/admin/users/:id | Admin | User detail |
| POST | /api/admin/entitlements/grant | Admin | Grant access |
| POST | /api/admin/entitlements/revoke | Admin | Revoke access |
| GET | /api/admin/entitlements | Admin | List entitlements |
| POST | /api/promotions/validate | Yes | Check promo code |
| POST | /api/promotions/redeem | Yes | Apply promo code |
| POST | /api/promotions | Admin | Create promo |
| GET | /api/promotions | Admin | List promos |
| DELETE | /api/promotions/:id | Admin | Deactivate promo |
| GET | /api/referrals | Yes | Referral info |
| POST | /api/referrals/invite | Yes | Send referral |
| GET | /health | No | Health check |
| GET | /api/health | No | Health check |

---

## 10. GDPR Cascade Delete (CONFIRMED — live code)

DELETE /api/auth/account deletes across:
1. Cancel Stripe subscriptions
2. Delete from mbs_platform: entitlements, transactions, sessions, push subs, referrals, friends, invites, email prefs, data requests, nostr/lnurl challenges, btcpay invoices, activity logs
3. Anonymize consent audit logs (hash user_id)
4. Delete from inner_lab: all collections matching user_id (cross-database)
5. Delete User record

**Impact on Phase 2**: Inner Lab middleware must store `user_id` as the same ObjectId from the platform. The cascade delete uses `mongoose.connection.client.db("inner_lab")` to reach across databases.

---

## 11. localStorage Key (CONFIRMED)

Token stored as: `localStorage.getItem("mbs_token")`

All product frontends must use this exact key name.
