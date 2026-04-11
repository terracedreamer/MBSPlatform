# MBS Platform Review — March 30, 2026

> **Historical snapshot.** This audit was conducted before the RS256 JWT upgrade (Session 11, March 31, 2026). JWT algorithm is now RS256, not HS256. HS256 fallback was fully removed in Session 12.

## Key Findings

### SSO (Verified from actual code)
- JWT payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- Algorithm: HS256, Expiry: 7 days
- Child apps redirect to `magicbusstudios.com/auth/login?redirect=<app_url>`
- Token returned via `?token=JWT` in redirect URL, stored in `localStorage` as `mbs_token`
- Google OAuth: Frontend sends Google ID token → backend verifies → issues JWT
- 2FA: Returns `{ requires2FA: true, tempToken }` → verify at `/api/auth/2fa/verify`
- Open redirect protection validates against CORS_ORIGINS hostnames

### Entitlements (Verified from actual code)
- Endpoint: `GET /api/entitlements/{product_slug}` with Bearer JWT
- Three tiers: not_subscribed → free_tier → paid (product_pass/category_access/mbs_all_access)
- free_tier is a DERIVED reason (not stored) — computed when product_pass has no stripe_subscription_id **[SUPERSEDED Session 31: free_tier is now an explicit Entitlement type]**
- 7-day trial on free subscribe, lazy downgrade after expiry **[SUPERSEDED Session 31: default trial removed, trial days configurable per product via admin]**
- freeTierLimits returned but NO product enforces them yet **[SUPERSEDED Session 31: freeTierLimits removed entirely, modules enforce own limits]**
- 22 products in catalog (11 IL + 5 Arcade + 6 Studio Works)

### GDPR Cascade (Verified from actual code)
- Three levels: app / category / full account
- Uses `Promise.allSettled` with 15-second timeout per app
- Full account deletion: cascade to apps → cancel Stripe → delete all MBS records → anonymize ConsentAuditLog → clean inner_lab DB → delete User
- Special paths: SmartCart uses `/api/users/me/data`, TaskTracker uses `/api/auth/user-data`

### Billing (Verified from actual code)
- Stripe Checkout for payments, Customer Portal for management
- Only CWG pricing functional — IL All Access and MBS All Access price IDs not yet created
- BTCPay routes exist but API key has insufficient permissions
- Webhook handles: checkout.session.completed, subscription.deleted, subscription.updated

### Discrepancies Found
| Area | Documented | Actual |
|------|-----------|--------|
| req.user shape | `{ userId, email, name, avatar }` | Also includes `isAdmin`, `iat`, `exp` |
| free_tier | Listed as entitlement type | Derived reason, not stored type |
| GDPR cascade | "Missing" in data-sovereignty.md | Code exists in repo, may not be deployed |
| Trivia apiUrl | Should be `triviaroast` | Has known typo `trivaroast` |
| MBS logging | Should use Winston | All console.log — no Winston |

### What New Apps Need from Platform
1. Product slug added to `server/config/products.js`
2. Same `JWT_SECRET` as platform
3. JWT verification middleware
4. Entitlement check via `GET /api/entitlements/{slug}`
5. Login redirect to `magicbusstudios.com/auth/login?redirect=<url>`
6. `DELETE /api/user-data` endpoint for GDPR cascade
7. Domain added to platform's CORS_ORIGINS
