# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 25, 2026

---

## Priority 1 — MBS Platform Phase 1 (MVP)

### Auth
- [ ] Google SSO login with JWT issuance
- [ ] Nostr authentication (challenge → signature verification)
- [ ] LNURL-Auth (k1 challenge → signature verification)
- [ ] Branded login page (Inner Lab vs MBS branding via `?brand=` param)
- [ ] Active session management (track JWTs, device sign-out)
- [ ] JWT verification middleware for product backends

### User Profiles
- [ ] User model (email, name, avatar, google_id, nostr_npub, lnurl_linking_key, auth_provider, consent_preferences, is_admin)
- [ ] Profile CRUD endpoints

### Entitlements
- [ ] Entitlement model (user_id, category, type, product, status, dates)
- [ ] GET /entitlements/:product → `{ hasAccess, reason }`
- [ ] GET /entitlements/category/:cat
- [ ] Admin grant/revoke endpoints

### Billing — Stripe
- [ ] Stripe checkout (product passes, category access, MBS all access)
- [ ] Stripe webhook handler (payment success, subscription changes, refunds)
- [ ] Stripe customer portal (manage subscriptions)
- [ ] Transaction history

### Billing — BTCPay (Lightning)
- [ ] BTCPay integration (invoice creation, payment verification)
- [ ] Lightning checkout flow
- [ ] BTCPay webhook handler

### Social
- [ ] Friends model (sorted user pair, source_product)
- [ ] Invites model (code, from_user, product, TTL)
- [ ] Friend/invite endpoints

### Platform Infrastructure
- [ ] Push subscription storage (device tokens)
- [ ] Feature flag system
- [ ] GDPR consent audit log
- [ ] Data request handling (access/deletion/portability)
- [ ] Health check endpoint
- [ ] CORS (all 22+ product domains), rate limiting, input validation
- [ ] Product catalog config (slugs, categories, URLs — NOT hardcoded)

### Migration
- [ ] CWG migration script (56 collections → mbs_platform users + inner_lab cwg_*)
- [ ] FlowState migration script (7 collections → mbs_platform users + inner_lab yoga_*)
- [ ] Verify migration with existing ~10 CWG users

### Deployment
- [ ] Deploy to Coolify at platform.magicbusstudios.com:3002
- [ ] Configure all env vars in Coolify
- [ ] Test full login flow from CWG → platform → back to CWG

---

## Priority 2 — MBS Platform Phase 2: Communication

- [ ] Welcome email on signup
- [ ] Purchase confirmation emails
- [ ] Subscription change emails (cancelled, expiring, payment failed)
- [ ] Basic admin panel: list users, view entitlements, manually grant/revoke
- [ ] Email preferences (opt-in/out per category)
- [ ] One-click unsubscribe (token-based)

---

## Priority 3 — MBS Platform Phase 3: Growth

- [ ] Promo code engine (create, validate, redeem, track usage)
- [ ] Free trial support (X days free, auto-convert to paid)
- [ ] Referral program (unique links, invite emails, automatic rewards)
- [ ] Win-back offers (auto-send discount to churned/inactive users)

---

## Priority 4 — CWG & FlowState Refactoring

- [ ] Update CWG backend to verify MBS Platform JWTs (remove standalone auth)
- [ ] Update CWG frontend to redirect to MBS Platform for login
- [ ] Remove CWG's standalone Stripe integration
- [ ] Remove CWG's standalone BTCPay integration
- [ ] Update FlowState backend to verify MBS Platform JWTs
- [ ] Update FlowState frontend to redirect to MBS Platform for login
- [ ] Remove FlowState's standalone Stripe integration
- [ ] Test full flow for both: login → purchase → access product

---

## Priority 5 — Connect All Products (Phase 4)

- [ ] Add checkAccess middleware to each Arcade game backend (5)
- [ ] Add checkAccess middleware to each Studio Works backend (6)
- [ ] Update each frontend to use MBS Platform JWT
- [ ] Test full flow for each product: login → purchase → access

---

## Priority 2B — Inner Lab Middleware (Layer 2) — Build NOW

**Build immediately after MBS Platform so migrations write shared data correctly. Reference: `InnerLab-middleware/CLAUDE.md`**

- [ ] Set up separate project + GitHub repo
- [ ] Express + Mongoose scaffolding (connects to inner_lab DB)
- [ ] JWT validation middleware (verifies MBS Platform tokens)
- [ ] il_consciousness_profiles collection + API (GET/PUT)
- [ ] il_personal_histories collection + API
- [ ] il_check_ins collection + API (mood, energy, stress, intention)
- [ ] il_user_memories collection + API (per-module + opt-in sharing)
- [ ] il_user_wellness_profiles collection (health conditions, injuries, goals — shared by FlowState + BreathArc)
- [ ] il_activity_feed collection + API
- [ ] Encryption & data export infrastructure
- [ ] Deploy to Coolify
- [ ] Daily briefing engine (LATER — when enough data exists)
- [ ] Cross-module insights (LATER — when enough data exists)

---

## Priority 7 — Inner Lab Frontend / Unified Dashboard (Build LATER)

- [ ] innerlab.ai logged-in dashboard (for category_access: "innerlab" subscribers)
- [ ] Daily briefing view
- [ ] Cross-module insights view
- [ ] Unified consciousness profile view
- [ ] Module launcher
- [ ] Upsell from standalone to Inner Lab All Access

---

## Priority 8 — MBS Platform Advanced Phases

- [ ] Phase 5: User dashboard (My Products, billing history, manage subscriptions)
- [ ] Phase 6: Admin dashboard (analytics, revenue, product stats, funnel tracking, audit log)
- [ ] Phase 7: Email campaigns + announcements + surveys + newsletter
- [ ] Phase 8: Multi-currency, family plan, teams, achievements, push notifications, enterprise SSO, API keys
