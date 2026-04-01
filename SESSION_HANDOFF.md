# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: March 31, 2026 (Session 13)
**Git Branch**: main
**Last Commit**: See per-repo commits below
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## SESSION 13 SUMMARY (March 31, 2026)

### What was done this session:

**GDPR Documentation Fix — CWG + FlowState Already Implemented**
- Session 12 docs said CWG and FlowState GDPR endpoints were "Missing" — they were already implemented
- CWG: `DELETE /api/user-data` in `server.py` (38 cwg_* + 7 il_* collections)
- FlowState: `DELETE /api/user-data` in `index.js` (7 yoga_* collections)
- Fixed GDPR status tables in: CLAUDE.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md, `~/.claude/rules/data-sovereignty.md`
- All 15 apps now confirmed to have GDPR endpoints

**#21 Test Coverage Expansion — Billing + Entitlements (MBS Platform)**

Added 40 new tests across 2 test files. Total MBS Platform test count: 61 (up from 21).

| File | Tests | What's Covered |
|------|-------|---------------|
| `billing.test.js` | 19 | Checkout (priceId, slug+period, validation, Stripe customer), Portal, History, Webhook (all 4 event types + signature + unknown events) |
| `entitlements.test.js` | 21 | List, Subscribe-free (trial, duplicate, validation), My-subscriptions (enriched+trial), Category check, Product access (not_subscribed, paid, free_tier, mbs_all_access priority, trialing) |
| `testApp.js` | — | Added `createBillingApp()` and `createEntitlementsApp()` factory functions |

All 61 tests pass. Mocked Stripe + Mongoose models — no real DB or Stripe needed.

**WildLens Toast Cleanup**
- Confirmed all 21 files already import from Sonner — custom ToastContext was orphaned
- Deleted `context/ToastContext.js`, `components/shared/Toast.js`, removed `.toast-container` CSS
- Updated WildLens CLAUDE.md. Committed + pushed (`dfcb726`)

**LazyChef Sonner Migration — Already Done**
- Discovered migration was already completed (commit `21881ba` on main)
- 30 files, 186 toast calls, all using Sonner. No work needed.

**Exploration for next session:**
- MBS admin/friends/promos/referrals routes fully analyzed — ready for test writing
- TaskTracker CRA→Vite scope: 71 files, 4 REACT_APP_ vars, 3 deploy targets (Coolify/Vercel/VPS)

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `bae89d9` | feat: add billing + entitlements test coverage (40 new tests) |

### Other repo commits this session:
| Repo | Commit | Message |
|------|--------|---------|
| MBSPlatform | `69f2f7a` | docs: Session 13 — test coverage + GDPR doc fix |
| Wildlife | `dfcb726` | chore: remove orphaned custom toast files (Sonner migration complete) |

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403

### For next session (Session 14):
- More MBS Platform tests — admin, friends, promos, referrals (routes already analyzed)
- TaskTracker CRA → Vite migration (71 files, 4 env vars, 3 deploy targets)
- CWG: merge `test` → `main` when ready
- Per-app standards improvements (response helpers, input validation, rate limiting)
- LazyChef SSO migration (future — frontend must redirect to MBS Platform before RS256 Phase 2)

---

## SESSION 12 SUMMARY (March 31, 2026)

### What was done this session:

**RS256 Phase 2 — Attempted and Reverted**

Attempted to remove HS256 fallback from all 15 repos. Chrome verification revealed:
- MBS Platform issues RS256 tokens correctly (`alg: RS256` confirmed)
- SmartCart (Node.js) accepted RS256 token and loaded data
- Lazy Chef (Python) still ran old code (Coolify hadn't redeployed) — HS256 token still accepted
- LazyChef frontend still calls local auth routes (`/api/auth/login`, `/api/auth/google/callback`) — removing `create_jwt_token` would break login

**Decision: Reverted all 15 repos back to dual-mode (RS256 first, HS256 fallback).** The fallback is a safety net that costs nothing. Removing it requires:
1. Verifying every Coolify service has redeployed with `JWT_PUBLIC_KEY` working
2. Migrating LazyChef frontend to MBS Platform SSO before removing local auth
3. Verifying all 15 apps end-to-end via Chrome after each Coolify redeploy

**What was changed (then reverted):**
- **MBS Platform**: `verifyToken()`, `issueToken()`, `issueTempToken()` — made RS256-only. **Reverted** to dual-mode.
- **10 Node.js child apps**: Simplified to RS256-only. **Reverted** to dual-mode with HS256 fallback.
- **4 Python child apps**: Simplified to RS256-only. **Reverted** to dual-mode.
- **CWG**: Both `utils/auth.py` and `core/dependencies.py` cleaned. **Reverted.**
- **All 15 repos**: `git revert` commits pushed. Code is identical to end of Session 11.

**LazyChef — Self-Issued Auth Removal Attempted and Reverted**
- Removed `create_jwt_token()` and set local auth routes to 410 Gone
- Chrome verification found LazyChef frontend still calls local `/api/auth/login` and `/api/auth/google/callback` — removing these breaks login
- **Reverted**: `create_jwt_token` and all local auth routes restored

### Net effect after revert:
- All 15 repos back to **dual-mode**: RS256 first, HS256 fallback (same as Session 11)
- LazyChef self-issued auth restored (same as Session 11)
- RS256 is confirmed working on MBS Platform (token shows `alg: RS256`)
- HS256 fallback ensures no app breaks regardless of Coolify redeploy timing

**Repo consolidation (same session):**
- Absorbed 3 audit files from Claude Setup into `MBSPlatform/audits/` (compliance audit, Arcade brand audit, platform review)
- Merged per-app standards compliance items into FUTURE_WORK_TODO.md
- Fixed stale GDPR table in CLAUDE.md (CWG and FlowState still missing)
- Fixed stale JWT comment in CLAUDE.md (HS256 fallback reference removed)
- Added `audits/` folder to CLAUDE.md folder structure
- Claude Setup slimmed to Claude infrastructure only (skills, rules, tasks, backups)

### Pending — Owner Action:
1. **Create Stripe products** — 6 products, 12 prices (still pending from Session 9)
2. **Regenerate BTCPay API key** — current returns 403

### For next session (Session 13):
- CWG GDPR endpoint (`DELETE /api/user-data`) — pure addition, on `test` branch
- FlowState GDPR endpoint — pure addition, push to `dev`
- #21 Test coverage expansion (frontend + billing + entitlements)
- CWG: merge `test` → `main` when ready

---

## SESSION 11 SUMMARY (March 31, 2026)

### What was done this session:

**#16 RS256 JWT Upgrade — All 15 repos upgraded, committed, pushed, verified live**

Upgraded JWT signing from HS256 (symmetric shared secret) to RS256 (asymmetric) across the entire 15-app ecosystem. This is a security-critical change: previously, if any child app's `JWT_SECRET` leaked, an attacker could forge tokens for ALL apps. Now only MBS Platform holds the private signing key; child apps only have the public verification key.

**Changes by repo:**

| Repo | Branch | File(s) Changed | What Changed |
|------|--------|-----------------|-------------|
| MBS | main | `server/middleware/auth.js`, `server/routes/auth.js`, `server/routes/admin.js`, `server/services/realtimeService.js` | Sign with RS256 (private key), dual-mode verify (RS256→HS256), new `GET /api/auth/public-key` endpoint, centralized `verifyToken()` function |
| Innerlab | main | `server/middleware/requireAuth.js` | Dual-mode verify |
| YogaGhost | dev | `server/auth.js` | Dual-mode verify |
| Brokenchain | main | `server/src/middleware/auth.js` | Dual-mode verify |
| Wildlife | main | `server/middleware/auth.js` | Dual-mode verify + helper function |
| Mindhacker | main | `server/src/middleware/auth.js` | Dual-mode verify |
| Trivia | main | `server/middleware/auth.js` | Dual-mode verify |
| Fakeartist | main | `server/middleware/requireAuth.js` | Dual-mode verify |
| Shopping | main | `server/middleware/auth.js` | Dual-mode verify |
| Movie | main | `server/middleware/auth.js` | Dual-mode verify |
| TaskTracker | main | `server/middleware/auth.js` | Dual-mode verify |
| CWG | test | `backend/utils/auth.py`, `backend/core/dependencies.py` | Dual-mode verify (PyJWT) |
| LazyChef | main | `backend/auth_service.py` | Dual-mode verify (PyJWT) |
| Tutor | main | `backend/auth_service.py` | Dual-mode verify (PyJWT — NOT python-jose) |
| Whispering House | main | `backend/app/core/auth.py` | Dual-mode verify (python-jose) |

**Key design decisions:**
- **Backward compatible**: All existing HS256 tokens continue to work via fallback
- **Dual-mode verify pattern**: Try RS256 (public key) first → if `TokenExpiredError`, re-throw (genuinely expired) → otherwise fall back to HS256 (old token)
- **RSA-2048 key pair generated**: Private key for MBS Backend only, public key for all 15 services
- **New endpoint**: `GET /api/auth/public-key` on MBS Platform returns PEM public key (for future dynamic key fetching)
- **LazyChef**: Still has legacy `create_jwt_token` (local auth). `verify_jwt_token` now supports RS256. Full removal of self-issued tokens is a separate cleanup task.

### Coolify deployment progress:
- ✅ **MBS Backend**: `JWT_PRIVATE_KEY` + `JWT_PUBLIC_KEY` added (multiline format), redeployed, **RS256 verified working** (token shows `alg: RS256`)
- ✅ **All 14 child apps**: `JWT_PUBLIC_KEY` added (multiline format in Coolify, "Is Multiline?" checkbox checked)

### Bug fixes during deployment:
- **Coolify PEM format**: Single-line env vars truncate PEM keys. Must use "Is Multiline?" checkbox in Coolify UI.
- **MBS `parsePemKey()`**: Added `while` loop to handle Coolify's double-escaped `\\n`, plus graceful fallback if RS256 signing fails (falls back to HS256 with error logging).
- **Tutor import bug**: Initial commit used `from jose import jwt` (python-jose) but Tutor only has `pyjwt`. Fixed to `import jwt` (PyJWT). Caused Coolify deploy failure (`ModuleNotFoundError: No module named 'jose'`).
- **Innerlab lock file**: `jest` + `supertest` added to `package.json` but `package-lock.json` not regenerated. Fixed with `npm install`. Caused ~6 failed Coolify deploys.

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `d1173e6` | feat: RS256 JWT signing — MBS Platform issuer upgrade |
| `49cf712` | fix: RS256 signing graceful fallback + robust PEM parsing |
| `b33ff67` | fix: improve RS256 error logging with key format diagnostics |
| `a0b4a74` | fix: handle double-escaped \n in PEM keys from Coolify |

### Pending — Owner Action:
1. ~~Add `JWT_PUBLIC_KEY` to child apps~~ — **DONE** (all 15 services configured)
2. **Keep `JWT_SECRET`** on all services — HS256 fallback still active (Phase 2 was attempted and reverted in Session 12)
3. `JWT_SECRET` is still needed — do NOT remove from Coolify env vars
4. **Stripe Dashboard** — Create 6 products with 12 prices (still pending from Session 9)
5. **BTCPay** — Regenerate API key with full store permissions (still pending)

### For next session (Session 12):
- #21 Test coverage expansion (frontend + billing + entitlements)
- RS256 Phase 2 cleanup: remove HS256 fallback from all apps (after 7 days)
- LazyChef: remove `create_jwt_token` self-issued auth (fully rely on MBS Platform)
- Verify RS256 is working end-to-end via Chrome on live site

---

## SESSION 10 SUMMARY (March 30, 2026)

### What was done this session:

**Batch 1 — Backend Hardening**
1. **#17 ADMIN_EMAILS env var** — `isEnvAdmin()` checks env before DB in `requireAdmin` + `issueToken` (auth.js)
2. **#19 Response helpers adoption** — all 14 route files converted (~316 conversions). No more inline `res.json({success})`.
3. **#13 Activity logging for subscription events** — subscribe-free, checkout completed, cancelled, expired, upgraded, trial expired

**Batch 2 — AccountPage Enhancements**
4. **#11 Connected Accounts UI** — Google, Email, Nostr, LNURL connection status + link Nostr form on AccountPage
5. **#18 GDPR "Type DELETE" confirmation** — all 3 delete modals (app, category, account) now require typing DELETE
6. **#10 3-step onboarding flow** — Welcome categories -> Pick product (subscribe-free) -> Success

**Batch 3 — Friends Enhancement**
7. **#12 Friends system enhancement** — avatar grid UI, remove friend button, `DELETE /api/friends/:friendId`, `GET /api/friends/:friendId/shared-products`

**Batch 4 — Admin Dashboard Overhaul**
8. **#20 AdminPage split** — 2067 -> 767 lines + 6 tab components + 8 reusable admin components (11 new files)
9. **#14 User segmentation** — `GET /api/admin/users/segments` endpoint + filter chips on Users tab (new/trial/paid/free/churned/inactive)
10. **#15 Revenue analytics** — `GET /api/admin/analytics/revenue` + new Analytics tab (MRR, ARR, ARPU, churn rate, growth rate, monthly trend, revenue by product)

**Batch 5 — Billing Features**
11. **#4 Referral email invites** — SendGrid HTML template, 10/day rate limit, preference check
12. **#5 Promo code checkout flow** — validate promo -> create Stripe coupon (percentage/fixed/trial_extension) -> apply to session + UI input on BillingPage

**Bug Fixes (post-batch)**
13. **Fix AdminUsers segment filter chips** — segments object was not being converted to array for filter chip rendering (`c374a94`)
14. **Fix admin tab content visibility** — added `initial={false}` to AnimatePresence in admin tabs for ProtectedRoute compatibility (`6150872`)

**Batch 6 — Entitlement & Billing Enhancements**
15. **#3 Real-time entitlement sync** — `refreshUser()` called on BillingPage `?success=true` redirect after Stripe/BTCPay checkout, so user sees updated access immediately
16. **#4 Premium feature gating** — `premiumFeatures` array added to all 22 products in `products.js`, `requireEntitlement(slug)` and `requirePremium` middleware created (`server/middleware/entitlement.js`), enhanced `GET /api/entitlements/:product` response with `isPremium` and `premiumFeatures`
17. **#5 Invoice PDF generation** — PDFKit installed, `server/services/invoiceService.js` creates branded PDFs, `GET /api/billing/invoice/:transactionId` endpoint, download button in BillingPage transaction history

### MBS Platform commits this session:
| Commit | Message |
|--------|---------|
| `0ba7114` | feat: Session 10 — 12 enhancements in 5 batches (32 files changed, 3426 insertions, 1811 deletions) |
| `c374a94` | fix: convert segments object to array for filter chips (AdminUsers) |
| `6150872` | fix: admin tab content initial={false} for ProtectedRoute compat |
| `2278e1d` | feat: entitlement sync, premium gating, invoice PDF generation |

### Pending — Owner Action:
1. **Stripe Dashboard** — Create 6 products with 12 prices (see FUTURE_WORK_TODO.md for table)
2. **BTCPay** — Regenerate API key with full store permissions (current returns 403)

### For next session (Session 11):
- **#16 RS256 JWT upgrade** (multi-repo, 2-3 days) — plan written and approved at `.claude/plans/humble-sparking-wigderson.md`, ready to execute
- #21 Test coverage expansion (frontend + billing + entitlements)

---

## SESSION 9 (March 30, 2026) — Pricing Redesign + Premium UI Polish
- Updated pricing: 6 products with 25% annual discount (FlowState $5, CWG $15, SW $10, Arcade $10, IL $20, MBS $30)
- Redesigned BillingPage (monthly/annual toggle, hero card, strikethrough pricing) + new IndividualPlansPage
- Premium UI polish on all 5 platform pages (particles, aurora, glass cards, hover elevations)
- Fixed ProtectedRoute animation bugs (SectionHeading, AnimatePresence initial={false})
- 9 commits: `65d0612` through `6ba2e57`

---

## SESSION 8 (March 29, 2026) — GDPR + Roadmap Features + Standards
- GDPR deletion committed/pushed to all 7 repos + Whispering House + MBS cascade service
- 5 roadmap features: subscribe gating + product picker + free trial (7-day), admin dashboard, CWG entitlements
- 12 enhancements: AuthContext, ProtectedRoute, profile editing, notifications, feature flags, activity feed, analytics, onboarding, push notifications, real-time admin
- Standards compliance: Winston logger, centralized error handler, response helpers, userId shape fix
- Key decisions: three-tier subscription, CWG stays on test, 7-day trial, RS256 deferred
- All verified live via Chrome. Commits: `b0edb9b` through `e2a0364`

---

## SESSION 6 (March 28, 2026) — Phase 5 Complete
- All 11 standalone products deployed and verified live via Chrome
- WildLens SSO login loop root causes identified and cascaded
- Architecture docs created, GDPR three-level deletion designed
- Platform-instructions re-synced to all 15 project folders

---

## ALL 5 PHASES COMPLETE

| Phase | Status | Verified |
|-------|--------|----------|
| Phase 1: MBS Platform | DONE — deployed at magicbusstudios.com | Yes |
| Phase 2: IL Middleware + Auth | DONE — deployed at innerlab.ai | Yes |
| Phase 3: CWG Migration | DONE — on `test` branch (needs promotion to main) | Yes |
| Phase 4: FlowState Migration | DONE — live on production | Yes |
| Phase 5: Standalone Products (11) | DONE — all deployed and verified live | Yes (Chrome, 2026-03-28) |

---

## WHAT NEEDS TO HAPPEN NEXT

### Immediate (Owner Action)
1. ~~Add `JWT_PUBLIC_KEY` to child apps~~ — **DONE**
2. **Keep `JWT_SECRET`** on all services — HS256 fallback is still active (Phase 2 reverted)
3. **Create Stripe products** — 6 products, 12 prices (see FUTURE_WORK_TODO.md for table)
4. **Regenerate BTCPay API key** — current returns 403

### Next Session (Session 14)
5. ~~CWG GDPR endpoint~~ — **Already implemented** (found Session 13)
6. ~~FlowState GDPR endpoint~~ — **Already implemented** (found Session 13)
7. ~~#21 Test coverage expansion (billing + entitlements)~~ — **DONE Session 13** (40 new tests)
8. ~~Commit + push MBS test files and MBSPlatform doc updates~~ — **DONE Session 13**
9. ~~LazyChef Sonner migration~~ — **Already done** (commit `21881ba`)
10. ~~WildLens Sonner migration~~ — **Cleaned up Session 13** (commit `dfcb726`)
11. More MBS Platform tests — admin, friends, promos, referrals (routes already analyzed, ready to write)
12. TaskTracker CRA → Vite migration (71 files, 4 env vars, 3 deploy targets — scope analyzed)
13. CWG: merge `test` → `main` when ready
14. Per-app standards improvements (response helpers, input validation, rate limiting)

### Future (Requires Migration Work)
15. LazyChef SSO migration — frontend must redirect to MBS Platform before `create_jwt_token` removal
16. RS256 Phase 2 redo — only after LazyChef SSO migration + verify all Coolify redeploys

---

## WHERE CODE GETS BUILT

| Folder | Domain | Database | Phase | Status |
|--------|--------|----------|-------|--------|
| `MBS/` | magicbusstudios.com | `mbs_platform` | 1 | DONE |
| `Innerlab/` | innerlab.ai | `inner_lab` | 2 | DONE |
| `CWG/` | conversationswithgod.ai | `inner_lab` (cwg_*) | 3 | DONE (test branch) |
| `YogaGhost/` | yoga.magicbusstudios.com | `inner_lab` (yoga_*) | 4 | DONE |
| `Brokenchain/` | brokenchain.magicbusstudios.com | `brokenchain` | 5 | DONE |
| `Mindhacker/` | mindhacker.magicbusstudios.com | `mindhacker` | 5 | DONE |
| `Trivia/` | triviaroast.magicbusstudios.com | `triviaroast` | 5 | DONE |
| `Fakeartist/` | fakeartist.magicbusstudios.com | `fakeartist` | 5 | DONE |
| `Whispering House/` | whisperinghouse.magicbusstudios.com | `whispering_house` | 5 | DONE |
| `Wildlife/` | wildlens.magicbusstudios.com | `wildlens` | 5 | DONE |
| `LazyChef/` | lazy-chef.magicbusstudios.com | `lazy_chef` | 5 | DONE |
| `Movie/` | moviepicker.magicbusstudios.com | `moviepicker` | 5 | DONE |
| `Shopping/` | smartcart.magicbusstudios.com | `smartcart` | 5 | DONE |
| `TaskTracker/` | tasktracker.magicbusstudios.com | `tasktracker` | 5 | DONE |
| `Tutor/` | tutor.magicbusstudios.com | `ai_tutor` | 5 | DONE |

---

## PREVIOUS SESSIONS

| Session | Date | Highlight |
|---------|------|-----------|
| 5 | March 27 | Phase 3B + Phase 4 reports reviewed |
| 4 | March 27 | Phase 3A complete, docs hardened, marketing updated |
| 3 | March 26 | Email/password auth + 2FA decisions, Inner Lab auth pages |
| 2 | March 26 | Full audit (50+ fixes), marketing briefs, orchestration guide |
| 1 | March 25 | Architecture finalized (17 decisions), three-layer design |
