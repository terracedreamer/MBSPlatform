# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 26, 2026

**RULE: If an item depends on a specific phase, it goes INTO that phase's platform-instructions document — NOT here. This file is ONLY for items that are either phase-independent or span multiple phases.**

---

## Phase Completion Tracker

| Phase | Status | Instructions Location | Items Tracked In |
|-------|--------|----------------------|-----------------|
| Phase 1: MBS Platform | ✅ Core done, addendum in progress | `platform-instructions-for-mbs/CLAUDE.md` | That doc (addendum section) |
| Phase 2: IL Middleware | Not started | `platform-instructions-for-innerlab/CLAUDE.md` | That doc |
| Phase 3: CWG Migration | Not started | `platform-instructions-for-cwg/PLATFORM_MIGRATION.md` | That doc |
| Phase 4: FlowState Migration | Not started | `platform-instructions-for-yogaghost/PLATFORM_MIGRATION.md` | That doc |
| Phase 5: Standalone Products | Not started | `platform-instructions-for-standalone-products/PLATFORM_MIGRATION.md` | That doc |

---

## Cross-Phase Dependencies (CRITICAL — review before starting each phase)

| Item | Depends On | Blocks | Notes |
|------|-----------|--------|-------|
| CWG migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 3 | Script writes to both mbs_platform AND inner_lab |
| FlowState migration script | Phase 1 (users table) + Phase 2 (il_* collections) | Phase 4 | Same pattern as CWG |
| Stripe product/price creation | Pricing decision (no phase dependency) | Real payments in any phase | Backend ready, needs Stripe Dashboard setup |
| GDPR cascade delete testing | Phase 1 (delete endpoint) + Phase 2 (il_* data) | None (works now, test after Phase 2) | Phase 1 built the cascade, Phase 2 creates the data it deletes |

---

## Pricing Decision (BLOCKS: Real payments across all phases)

**Status: Not decided. Deferred.**

Before any product can charge real money, we need:
- [ ] Decide pricing structure: per-product subscriptions vs category bundles vs all-access — what combinations?
- [ ] Decide which products are free forever (Arcade? Studio Works?) vs freemium (CWG free tier) vs paid-only
- [ ] Decide how existing CWG Stripe subscriptions appear on the platform
- [ ] Create Stripe products in Dashboard (prefixed `[MBS]` per convention)
- [ ] Create Stripe Price IDs and wire into billing checkout flow
- [ ] Design full product catalog billing page (22 products, 3 categories)
- [ ] Add Lightning as equal payment option alongside Stripe in billing UI

**This does NOT block Phase 2, 3, 4, or 5.** Entitlement checks work. Free tier works. Products can launch and be used. Payments are the last mile.

---

## Phase-Independent Future Work

These items don't belong to any specific phase and can be done anytime:

### Marketing / Content
- [ ] Convert CWG marketing plan .docx in Desktop/Marketing/ to .md (old content, may delete)
- [ ] Add "Login" button to MBS marketing site nav (currently in Phase 1 addendum — UX refinement later)

### Platform Enhancements (after all phases complete)
- [ ] Welcome email on signup (SendGrid)
- [ ] Purchase confirmation emails
- [ ] Subscription change emails (cancelled, expiring, payment failed)
- [ ] BTCPay expiry warning emails (30-day passes)
- [ ] Basic admin panel: list users, view entitlements, grant/revoke
- [ ] Email preferences (opt-in/out per category)
- [ ] One-click unsubscribe (token-based)
- [ ] Nostr authentication (models exist, routes deferred)
- [ ] LNURL-Auth (models exist, routes deferred)
- [ ] Auth method linking (POST /api/auth/link-nostr, link-lnurl)
- [ ] Promo code engine (model exists)
- [ ] Free trial support (X days free, auto-convert)
- [ ] Referral program (model exists)
- [ ] Win-back offers

### Technical Debt (no phase dependency)
- [ ] JWT upgrade to RS256 asymmetric signing (eliminates shared secret risk)
- [ ] Token refresh endpoint (silent re-authentication)
- [ ] User dashboard (My Products, billing history, manage subscriptions)
- [ ] Admin dashboard (analytics, revenue, funnel tracking)
- [ ] Email campaigns + announcements + newsletter
- [ ] Multi-currency, family plan, teams, push notifications, enterprise SSO
