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

## Pricing Decision — PARTIALLY RESOLVED (moved to Phase 1 Addendum #13)

**Status: CWG pricing decided. Bundles decided. Arcade/SW/FlowState deferred.**

- [x] CWG pricing: $9.99/mo, $79.99/yr (Stripe) + 21K sats/mo, 126K sats/yr (Lightning)
- [x] Inner Lab All Access: $19.99/mo, $159.99/yr
- [x] MBS All Access: $29.99/mo, $249.99/yr
- [x] Structured billing page design (category tabs → product → plans)
- [x] Lightning as equal payment option alongside Stripe
- [ ] FlowState pricing (no paid tier defined yet — currently free)
- [ ] Arcade pricing (free for now)
- [ ] Studio Works pricing (free for now)
- [ ] Create actual Stripe products in Dashboard (manual step — owner must do before testing real payments)

**CWG already has Stripe price IDs** in its Coolify env vars. The Phase 1 agent can either reuse those or create new platform-level ones.

---

## Phase-Independent Future Work

These items don't belong to any specific phase and can be done anytime:

### Marketing / Content
- [ ] Convert CWG marketing plan .docx in Desktop/Marketing/ to .md (old content, may delete)

### Moved to Phase 1 Addendum (no longer future work)
All of these are now in `platform-instructions-for-mbs/CLAUDE.md` under "Phase 1 Addendum":
- ~~Login button~~ → Phase 1 Addendum #1
- ~~Rate limiting~~ → Phase 1 Addendum #2
- ~~Lightning billing UI~~ → Phase 1 Addendum #3
- ~~Nostr auth~~ → Phase 1 Addendum #4
- ~~LNURL-Auth~~ → Phase 1 Addendum #5
- ~~Auth method linking~~ → Phase 1 Addendum #6
- ~~Email preferences~~ → Phase 1 Addendum #7
- ~~Transactional emails~~ → Phase 1 Addendum #8
- ~~Admin panel~~ → Phase 1 Addendum #9
- ~~Promo codes~~ → Phase 1 Addendum #10
- ~~Referrals~~ → Phase 1 Addendum #11
- ~~Token refresh~~ → Phase 1 Addendum #12

### Genuinely Future (no phase, needs multiple phases or decisions first)
- [ ] Free trial support (X days free, auto-convert) — needs pricing decision
- [ ] Win-back offers — needs email + promo system working together
- [ ] JWT upgrade to RS256 asymmetric signing — technical debt, no rush
- [ ] User dashboard (My Products, billing history) — needs pricing + real subscriptions
- [ ] Admin dashboard (analytics, revenue, funnel tracking) — needs real data
- [ ] Email campaigns + announcements + newsletter — post-launch
- [ ] Multi-currency, family plan, teams, push notifications, enterprise SSO — long-term
