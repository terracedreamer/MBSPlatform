# FUTURE WORK TODO — MBS Platform

**Last Updated**: March 29, 2026

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

### Reminders (Owner Action Required)
- [ ] **Create Stripe products in Dashboard** — IL All Access ($19.99/mo, $159.99/yr) and MBS All Access ($29.99/mo, $249.99/yr). CWG prices already exist.
- [ ] **Regenerate BTCPay API key** — Current key has insufficient permissions (403). Lightning payments don't work until fixed.

### Post-Phase 5 Cleanup
- [ ] **Dead code cleanup across Phase 5 apps** — Old auth files, unused npm/pip deps (bcryptjs, passport, etc.) still on disk in most apps. Harmless but messy. Cleanup prompt ready in SESSION_HANDOFF.md Session 6.
- [x] **GDPR `DELETE /api/user-data` endpoint** — Built for all 6 missing apps (Session 7). Code NOT yet committed/pushed to individual repos.
- [x] **MBS Platform deletion UI + cascade service** — Built in Session 7. Cascade service, new routes, AccountPage UI. Code NOT yet committed/pushed.
- [ ] **CWG entitlements** — CWG doesn't call the entitlements API yet. No free/premium enforcement.
- [ ] **TaskTracker MongoDB transactions** — Legacy user migration uses transactions (requires replica set). May fail on standalone MongoDB. Needs non-transactional fallback.

### Decided — Ready to Build (Session 7 Decisions, 2026-03-29)
- [ ] **Three-tier subscription model** — All products require explicit subscription before access (even free tier). Three states: Not Subscribed → Free Subscriber → Premium Subscriber. Gives owner engagement/interest data.
- [ ] **Subscribe gating for all apps** — Every product gates access behind subscription. Users must click "Subscribe" to get free access. Premium gating only on products with defined free/premium splits (currently only CWG).
- [ ] **CWG entitlement enforcement** — CWG has free/premium defined but doesn't call entitlements API yet. Wire it up to enforce free vs premium.
- [ ] **Product/module picker (all products)** — Authenticated page showing ALL products (Inner Lab modules, Arcade games, Studio Works tools). Users subscribe to each individually. Not just Inner Lab — covers entire catalog.
- [ ] **Consolidate friends to MBS Platform level (Option A)** — Remove friends code from CWG and FlowState. Point both to MBS Platform friends API (`/api/friends/*`), already built but unused. Can evolve to Option C (product-context tags) later.
- [ ] **Admin dashboard (hierarchical)** — Master dashboard at magicbusstudios.com/admin. Top level: all users, revenue, signups, subscriptions. Drill-down: MBS → Inner Lab → CWG, MBS → Arcade → per-game, MBS → Studio Works → per-tool. Absorbs CWG's existing admin dashboard data.
- [ ] **Free trial (7 days premium)** — On subscription, users get 7 days premium free for that product. Only applies to products with premium features (currently CWG only). Future: require credit card upfront, auto-charge after 7 days.

### Future Work (Decided but Deferred)
- [ ] **JWT upgrade to RS256 asymmetric signing** — All products share same HS256 secret; leaked key = forge tokens for all apps. Important before real users/launch. Move to RS256: MBS Platform holds private key (signing), all products hold public key (verification only).
- [ ] **Admin accounts via ADMIN_EMAILS env var** — Currently `is_admin` is a DB field. Future: check email against `ADMIN_EMAILS` env var in Coolify. Easier to manage, no DB changes. Admin emails: `terracedreamer@gmail.com`, `1984.abhinav@gmail.com`.
- [ ] **Premium feature gating (per-product)** — Entitlement infrastructure wired in all 13 apps but nothing enforces free vs paid. Needs per-product decisions on what's free vs premium for each app beyond CWG.
- [ ] Win-back offers — needs email + promo system working together
- [ ] User dashboard (My Products, billing history) — needs pricing + real subscriptions
- [ ] Email campaigns + announcements + newsletter — post-launch
- [ ] Multi-currency, family plan, teams, push notifications — long-term

---

## Standards Compliance (Global CLAUDE.md Audit)

> Reference: Check global CLAUDE.md and ~/.claude/rules/ for full standards.

- [x] **Verify Architecture Docs**: Fixed Movie Picker (single container, not 2), updated GDPR limitation note for Session 7 cascade service, bumped version to 1.1 (2026-03-29)
- [x] **Verify GDPR Status Table**: Updated data-sovereignty.md and CLAUDE.md — 6 apps moved from "Missing" to "Built (pending deploy)", Whispering House still missing
- [x] **Verify Deployment Quirks**: Added Movie Picker (single container, TMDB API) and AI Tutor (Python/FastAPI, JWT fallback chain) entries. Fixed Lazy Chef DB name in env-standards.md (`lazychef` → `lazy_chef`)

> Note: This is an architecture-only repo — no application code to audit.
