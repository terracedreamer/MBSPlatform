# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 24, 2026

---

## Priority 1 — Immediate (Next Session)

- [ ] Inspect CWG MongoDB database (collections, fields, sample data)
- [ ] Design data migration plan: what goes to `mbs_platform` vs `inner_lab`
- [ ] Identify which CWG data might become shared `il_*` tables
- [ ] Begin MBS Platform Phase 1 scaffolding (Express, Mongoose, project structure)

## Priority 2 — MBS Platform Phase 1 (MVP)

- [ ] Google SSO login with JWT issuance
- [ ] User profiles (name, email, avatar, preferences)
- [ ] Entitlements CRUD (grant, revoke, check access)
- [ ] Stripe checkout (product passes, category access, MBS all access)
- [ ] Stripe webhook handler (payment success, subscription changes)
- [ ] Stripe customer portal
- [ ] CWG user migration script (~10 users)
- [ ] Health check endpoint
- [ ] CORS, rate limiting, input validation
- [ ] Deploy to Coolify at platform.magicbusstudios.com

## Priority 3 — CWG Refactoring

- [ ] Migrate CWG data to `inner_lab` database with `cwg_` prefix
- [ ] Remove CWG's standalone auth — use MBS Platform SSO
- [ ] Remove CWG's Stripe integration — use MBS Platform billing
- [ ] Update CWG backend to verify JWTs from MBS Platform
- [ ] Update CWG frontend to redirect to MBS Platform for login

## Priority 4 — Inner Lab Shared Data Design

- [ ] Design `il_*` shared table schemas (profiles, consciousness, state, check-ins)
- [ ] Define read/write rules (which modules can write to shared tables)
- [ ] Define the Inner Lab Core API surface
- [ ] Decide exact domains for future modules

## Priority 5 — Inner Lab Core Container

- [ ] Build when 2-3 modules exist
- [ ] Daily briefing engine
- [ ] Cross-module insights
- [ ] Check-in API (mood, energy, stress, intention)
- [ ] Consciousness profile API
- [ ] State engine (aggregated user state)

## Priority 6 — Inner Lab Unified Dashboard

- [ ] innerlab.ai logged-in dashboard (for category_access subscribers)
- [ ] Daily briefing view
- [ ] Cross-module insights view
- [ ] Unified consciousness profile view
- [ ] Module launcher

## Priority 7 — Connect All Products

- [ ] Add checkAccess middleware to each Arcade game backend
- [ ] Add checkAccess middleware to each Studio Works backend
- [ ] Update each frontend to use MBS Platform JWT
- [ ] Test full flow for each product: login → purchase → access

## Backlog — Future Phases

- [ ] MBS Platform Phase 2: Transactional emails + basic admin
- [ ] MBS Platform Phase 3: Promo codes + free trials + referrals
- [ ] MBS Platform Phase 5: User dashboard (My Products, billing history)
- [ ] MBS Platform Phase 6: Admin dashboard (analytics, revenue)
- [ ] MBS Platform Phase 7: Email campaigns + announcements
- [ ] MBS Platform Phase 8: Multi-currency, family plan, teams, API keys
- [ ] Build next Inner Lab modules (BreathArc, StarMap, etc.)
