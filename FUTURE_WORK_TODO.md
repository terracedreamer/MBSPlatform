# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 31, 2026

**RULE: If an item depends on a specific phase, it goes INTO that phase's platform-instructions document — NOT here. This file is ONLY for items that are either phase-independent or span multiple phases.**

---

## Phase Completion Tracker

| Phase | Status | Instructions Location | Items Tracked In |
|-------|--------|----------------------|-----------------|
| Phase 1: MBS Platform | ✅ DONE — all addendum items #1-15 deployed | `platform-instructions-for-mbs/CLAUDE.md` | That doc (addendum section) |
| Phase 2: IL Middleware + Auth | ✅ DONE — middleware + 4 auth pages live at innerlab.ai | `platform-instructions-for-innerlab/CLAUDE.md` | That doc |
| Phase 3: CWG Migration | ✅ DONE — Phase 3A (data) + Phase 3B (refactor) complete, running on `test` | `platform-instructions-for-cwg/PLATFORM_MIGRATION.md` | That doc |
| Phase 4: FlowState Migration | ✅ DONE — live on production (dev + main), entitlements wired | `platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md` | That doc |
| Phase 5: Standalone Products | ✅ DONE — all 11 deployed and verified live (2026-03-28) | `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` | That doc + 11 PHASE_5_REPORT files |

---

## Cross-Phase Dependencies (CRITICAL — review before starting each phase)

| Item | Depends On | Blocks | Notes |
|------|-----------|--------|-------|
| CWG migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 3 | Script writes to both mbs_platform AND inner_lab |
| FlowState migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 4 | Same pattern as CWG |
| Stripe product/price creation | Pricing decision (no phase dependency) | Real payments in any phase | Backend ready, needs Stripe Dashboard setup |
| GDPR cascade delete testing | Phase 1 (delete endpoint) + Phase 2 (il_* data) | None (works now, test after Phase 2) | Phase 1 built the cascade, Phase 2 creates the data it deletes |

---

## Pricing Decision — PARTIALLY RESOLVED (moved to Phase 1 Addendum #13)

**Status: CWG pricing decided. Bundles decided. Arcade/SW/FlowState deferred.**

- [x] CWG pricing: $9.99/mo, $79.99/yr (Stripe) + 21K sats/mo, 126K sats/yr (Lightning)
- [x] Inner Lab All Access: $19.99/mo, $159.99/yr
- [x] MBS All Access: $29.99/mo, $249.99/yr
- [x] Structured billing page design (category tabs → product → plans)
- [x] Lightning as equal payment option alongside Stripe
- [x] FlowState pricing — $5/mo, $45/yr (Session 9)
- [x] Arcade pricing — $10/mo bundle, $5/mo individual (Session 9)
- [x] Studio Works pricing — $10/mo bundle, $5/mo individual (Session 9)
- [ ] Create actual Stripe products in Dashboard (OWNER ACTION — see Stripe price table below in Reminders section)

**CWG already has Stripe price IDs** in its Coolify env vars. The Phase 1 agent can either reuse those or create new platform-level ones.

---

## Phase-Independent Future Work

These items don't belong to any specific phase and can be done anytime:

### Marketing / Content
- [ ] Convert CWG marketing plan .docx in Desktop/Marketing/ to .md (old content, may delete)

### Moved to Phase 1 Addendum
12 items (login, rate limiting, Lightning, Nostr, LNURL, auth linking, email prefs, transactional emails, admin panel, promo codes, referrals, token refresh) — all completed and deployed. See `platform-instructions-for-mbs/CLAUDE.md` addendum section.

### Reminders (Owner Action Required)
- [ ] **Create Stripe products in Dashboard** — 6 products, 12 prices:

| Product | Monthly | Annual | Env Var (Monthly) | Env Var (Annual) |
|---------|---------|--------|-------------------|------------------|
| FlowState | $5 | $45 | `STRIPE_YOGA_MONTHLY_PRICE_ID` | `STRIPE_YOGA_ANNUAL_PRICE_ID` |
| CWG | $15 | $135 | `STRIPE_CWG_MONTHLY_PRICE_ID` | `STRIPE_CWG_ANNUAL_PRICE_ID` |
| Studio Works All Access | $10 | $90 | `STRIPE_SW_MONTHLY_PRICE_ID` | `STRIPE_SW_ANNUAL_PRICE_ID` |
| Arcade All Access | $10 | $90 | `STRIPE_ARCADE_MONTHLY_PRICE_ID` | `STRIPE_ARCADE_ANNUAL_PRICE_ID` |
| Inner Lab All Access | $20 | $180 | `STRIPE_IL_MONTHLY_PRICE_ID` | `STRIPE_IL_ANNUAL_PRICE_ID` |
| MBS All Access | $30 | $270 | `STRIPE_MBS_MONTHLY_PRICE_ID` | `STRIPE_MBS_ANNUAL_PRICE_ID` |

Steps: Create in Stripe Dashboard (test mode) → copy 12 price IDs → add to Coolify MBS B env vars → redeploy.

- [ ] **Regenerate BTCPay API key** — Current key has insufficient permissions (403). Lightning payments don't work until fixed.

### Completed — Session 10
12 enhancements (ADMIN_EMAILS, response helpers, activity logging, connected accounts, GDPR type-DELETE, onboarding, friends, AdminPage split, user segmentation, revenue analytics, referral emails, promo checkout) + 3 billing features (entitlement sync, premium gating, invoice PDFs). See CHANGELOG.md Session 10 entries.

### Deferred to Future Session
- [x] #16 JWT upgrade to RS256 — **COMPLETED Session 11**. All 15 repos upgraded, all Coolify env vars configured, verified working.
- [ ] #21 Test coverage expansion (frontend + billing + entitlements)

### Completed — Session 11 (RS256 JWT Upgrade)
- [x] **#16 RS256 JWT upgrade** — All 15 repos upgraded + all Coolify env vars configured. MBS Platform signs RS256, all child apps verify RS256→HS256 dual-mode. Fully deployed and verified.

### Session 12 Potential Items
- [ ] #21 Test coverage expansion (deferred from Sessions 10-11)
- [ ] RS256 Phase 2 cleanup — remove HS256 fallback after 7 days, remove `JWT_SECRET` from child apps
- [ ] LazyChef — remove `create_jwt_token` self-issued auth

### Post-Phase 5 Cleanup — All Complete
Dead code cleanup, GDPR endpoints (all 8 apps), MBS deletion UI + cascade, CWG entitlements, TaskTracker transactions. See CHANGELOG.md Session 8.

### Decided & Built — Session 8
Subscribe gating, product picker, CWG entitlements, friends consolidation, admin dashboard, free trial (7 days). All built and deployed. See CHANGELOG.md Session 8.

### Completed — Session 8 Enhancements
AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin. See CHANGELOG.md Session 8.

### Future Work (Decided but Deferred)
- [ ] Stripe subscription portal testing — "Manage Subscription" button calls `/api/billing/portal`. Test full upgrade/downgrade/cancel flow once Stripe products are created.
- [ ] BTCPay expiry reminders — Lightning is a 30-day one-time pass with manual renewal. Add email reminders when the 30 days are about to expire.
- [ ] Win-back offers — needs email + promo system working together
- [ ] User dashboard (My Products, billing history) — needs pricing + real subscriptions
- [ ] Email campaigns + announcements + newsletter — post-launch
- [ ] Multi-currency, family plan, teams — long-term

---

## Standards Compliance — All Passing (Audited Session 8)
Backend + frontend standards audit completed. All items pass. See CHANGELOG.md Session 8 for details.
