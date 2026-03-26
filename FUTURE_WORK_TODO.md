# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 26, 2026

---

## Priority 0 — Marketing Brief Enhancement ~~(Next Session)~~ DONE

- [x] Absorb Arcade brief content into MBS master doc
- [x] Check innerlab.ai website for missing product info → updated IL brief
- [x] Add Inner Lab dashboard vision, module connections, activity feed to IL brief
- [x] Check magicbusstudios.com for missing info → updated MBS master doc
- [x] Add Studio Works detail to MBS master doc
- [ ] Convert CWG marketing plan .docx in Desktop/Marketing/ to .md (old content, may delete)
- [x] Sync updated briefs to Desktop/Marketing/Overview/

---

## Priority 1 — MBS Platform Phase 1 (MVP)

### Auth
- [ ] Google SSO login with JWT issuance (JWT spec defined: userId, email, name, avatar, isAdmin)
- [ ] Branded login page (Inner Lab vs MBS branding via `?brand=` param)
- [ ] Open redirect protection (validate ?redirect= against ALLOWED_REDIRECT_DOMAINS derived from CORS_ORIGINS)
- [ ] Token-in-URL handling (extract, store, replaceState)
- [ ] Active session management (track JWTs, device sign-out)
- [ ] JWT verification middleware for product backends
- [ ] Defer: Nostr auth, LNURL auth (Phase 2+)

### User Profiles
- [ ] User model (18 fields including stripe_customer_id — fully defined in platform-instructions)
- [ ] Profile CRUD endpoints

### Entitlements
- [ ] Entitlement model (stripe_customer_id removed — lives on User only)
- [ ] GET /api/entitlements/:product → `{ success: true, hasAccess, reason }` (5 reason values defined)
- [ ] GET /api/entitlements/category/:cat
- [ ] Product catalog config with freeTier flags and freeTierLimits
- [ ] Entitlement priority: mbs_all_access > category_access > product_pass > free_tier
- [ ] Admin grant/revoke endpoints (verify is_admin from DB, not JWT)

### Billing — Stripe
- [ ] Stripe checkout (product passes, category access, MBS all access)
- [ ] Stripe webhook → create/update Entitlement
- [ ] Stripe customer portal
- [ ] Transaction history

### Billing — BTCPay (Lightning)
- [ ] BTCPay checkout → create invoice
- [ ] BTCPay webhook → settled = create Entitlement with expiry (no auto-renew)
- [ ] Lightning checkout flow

### Social
- [ ] Friends model + endpoints
- [ ] Invites model + endpoints

### Platform Infrastructure
- [ ] GDPR deletion cascade (DELETE /api/auth/account — deletes across mbs_platform + inner_lab + all module collections)
- [ ] CORS (18 domains enumerated in platform-instructions)
- [ ] Health check, rate limiting, input validation
- [ ] Product catalog config (22 products, NOT hardcoded)
- [ ] Push subscriptions, feature flags, consent audit log, data requests
- [ ] Deployment: defensive route loading (try/catch on new routes), env vars first
- [ ] JWT security: shared secret risk acknowledged, RS256 upgrade path for Phase 2+
- [ ] Entitlement cache spec documented for downstream products (5min TTL, ?refresh=true)

### Migration Scripts (built here, run once)
- [ ] CWG migration script (56 collections → 3 buckets with field renames)
- [ ] FlowState migration script (7 collections → 3 buckets with field renames)

### Deployment
- [ ] Follow deployment checklist in platform-instructions
- [ ] Set all env vars in Coolify BEFORE deploying
- [ ] Test: login flow, entitlement check, existing marketing pages still work

---

## Priority 2 — Inner Lab Middleware + Dashboard (Layer 2) — Build Immediately After Phase 1

**Reference: `platform-instructions-for-innerlab/CLAUDE.md`**

### Backend
- [ ] Mongoose + inner_lab DB connection on existing Express server
- [ ] JWT validation middleware
- [ ] il_* Mongoose models matching schema contracts (check-ins, consciousness, histories, wellness, memories, activity feed)
- [ ] MongoDB JSON Schema validation on critical collections
- [ ] All API routes at /api/ prefix (20 endpoints defined)
- [ ] Deploy to Coolify

### Frontend
- [ ] Auth-gated routing (dashboard pages require JWT + category_access entitlement)
- [ ] Dashboard, Consciousness, Memories, Activity pages
- [ ] Upsell page for users without Inner Lab All Access

### Deferred
- [ ] Daily briefing engine + view (need data from 2-3 modules)
- [ ] Cross-module insights engine + view

---

## Priority 3 — CWG & FlowState Migration

- [ ] Run CWG migration script from MBS/ (copy data, field renames, upsert on email)
- [ ] Refactor CWG backend (inner_lab DB, JWT middleware in Python, cwg_* prefix, remove auth/billing)
- [ ] Refactor CWG frontend (remove login/billing pages, redirect to platform)
- [ ] Run FlowState migration script (0 users — mostly structural)
- [ ] Refactor FlowState backend + frontend (same pattern as CWG)
- [ ] Deploy and test both

---

## Priority 4 — Connect Standalone Products (11 apps)

- [ ] Copy standalone-products instructions to each project
- [ ] Add JWT middleware to each Arcade game (5) and Studio Works app (6)
- [ ] Remove standalone auth from each
- [ ] Set JWT_SECRET + PLATFORM_URL in each Coolify service
- [ ] Test login → entitlement check for each product

---

## Priority 5 — MBS Platform Phase 2: Communication

- [ ] Welcome email on signup
- [ ] Purchase confirmation emails
- [ ] Subscription change emails (cancelled, expiring, payment failed)
- [ ] BTCPay expiry warning emails
- [ ] Basic admin panel: list users, view entitlements, grant/revoke
- [ ] Email preferences (opt-in/out per category)
- [ ] One-click unsubscribe (token-based)
- [ ] Auth method linking (POST /api/auth/link-nostr, link-lnurl)
- [ ] Nostr authentication
- [ ] LNURL-Auth

---

## Priority 6 — MBS Platform Phase 3: Growth

- [ ] Promo code engine
- [ ] Free trial support (X days free, auto-convert)
- [ ] Referral program
- [ ] Win-back offers

---

## Priority 7 — MBS Platform Advanced

- [ ] Phase 5: User dashboard (My Products, billing history, manage subscriptions)
- [ ] Phase 6: Admin dashboard (analytics, revenue, funnel tracking)
- [ ] Phase 7: Email campaigns + announcements + newsletter
- [ ] Phase 8: Multi-currency, family plan, teams, push notifications, enterprise SSO
- [ ] JWT upgrade to RS256 asymmetric signing
- [ ] Token refresh endpoint (silent re-authentication)
