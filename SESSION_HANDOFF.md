# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: March 26, 2026 (Session 3 — FINAL)
**Git Branch**: main
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## WHERE WE ARE RIGHT NOW

### Phase 1: MBS Platform — COMPLETE + New Addendum Items
- **Built and deployed** at magicbusstudios.com: Google SSO, JWT auth, entitlements, Stripe checkout, BTCPay checkout, friends/invites, branded login page, GDPR delete cascade, health check
- **Phase 1 Addendum** (13 items) — ALL DONE:
  1. Login button in nav (**DONE**)
  2. Rate limiting on platform routes (**DONE**)
  3. Lightning payment toggle in billing UI (**DONE**)
  4. Nostr authentication routes (**DONE**)
  5. LNURL-Auth routes (**DONE**)
  6. Auth method linking (**DONE**)
  7. Email preferences + one-click unsubscribe (**DONE**)
  8. Transactional emails via SendGrid (6 types) (**DONE**)
  9. Basic admin panel (**DONE**)
  10. Promo code routes (**DONE**)
  11. Referral routes (**DONE**)
  12. Token refresh endpoint (**DONE**)
  13. Structured billing page with real CWG pricing (**DONE**)
  14. Email/Password auth + signup page (**DONE — deployed**)
  15. 2FA/TOTP with backup codes (**DONE — deployed**)
- **Phase 1 reports**: `phase-reports/PHASE_1_REPORT.md` and `phase-reports/PHASE_1_LEARNINGS.md`

### Known Issues (Open)
- **BTCPay 403 error**: API key has insufficient permissions. Lightning payments fail. Regenerate BTCPay API key with full store permissions.
- **Stripe bundle price IDs**: CWG prices work. IL All Access and MBS All Access price IDs need Stripe Dashboard creation.
- **MBS /auth/forgot-password missing**: MBS login page links to it but the React component was never built (known gap from Phase 1 Addendum). Low priority — Inner Lab has its own page.
- **GDPR cascade incomplete**: Phase 1 DELETE /api/auth/account only cleans up mbs_platform data. Does not cascade to il_* data in inner_lab. Needs a Phase 2 endpoint that Phase 1 can call. Future work.

### Phase 2: Inner Lab Middleware + Auth Pages — FULLY COMPLETE
- **Built and deployed** at innerlab.ai / api.innerlab.ai
- 11 Mongoose models, 22 API routes, 4 dashboard pages (original Phase 2)
- 4 auth pages live: innerlab.ai/auth/login, /signup, /forgot-password, /reset-password (Phase 2 Addendum)
- ProtectedRoute fixed — was sending users to magicbusstudios.com/login (404), now goes to /auth/login (local)
- **Bonus fixes caught during this work**: Coolify backend service had wrong Dockerfile (should be Dockerfile.server) and wrong domain (`.com` vs `.ai`) — both fixed and redeployed
- Google Cloud Console updated: innerlab.ai added as authorized JS origin for Google SSO
- **Phase 2 reports**: `phase-reports/PHASE_2_REPORT.md` + `phase-reports/PHASE_2_ADDENDUM_REPORT.md`
- **Dashboard unblocked** — full login loop works end-to-end

### Session 3 Architecture Decisions
- **Email/Password auth added** as 4th auth method (alongside Google SSO, Nostr, LNURL)
- **2FA/TOTP with backup codes** added
- **Login/signup pages on BOTH sites**: magicbusstudios.com AND innerlab.ai (each branded, both call MBS Platform auth APIs)
- **CWG users auto-become platform users** during migration (including password hashes + 2FA data)
- **Inner Lab modules** redirect to `innerlab.ai/auth/login` (not magicbusstudios.com)
- **Standalone products** redirect to `magicbusstudios.com/auth/login`

### Session 3 Code Changes (made from orchestrator, not from project agents)
- **MBS/**: Added "Sign Up" button to nav (Layout.jsx) — `ORCHESTRATOR_CHANGES.md` documents this
- **Innerlab/**: Added "Sign In" + "Sign Up" to nav, fixed ProtectedRoute redirect, branded auth page headings — `ORCHESTRATOR_CHANGES.md` documents this
- **Both projects pushed and deployed via Coolify auto-deploy**

### Phases 3-5: READY TO START
- All instructions audited and corrected this session (FlowState redirect fixed, new-modules template fixed)
- All synced to their target project folders
- CWG migration doc: users copied with password_hash + 2FA fields + auth_methods array
- FlowState migration doc: login redirects to innerlab.ai
- Phase 3-5 prompts in ORCHESTRATION_GUIDE.md are accurate and ready to use

---

## WHAT NEEDS TO HAPPEN NEXT

### All immediate work is DONE. Next actions:

1. **Phase 3: CWG Migration** — READY. All blockers cleared.
   - Step A: Run migration script from MBS/ (copies ~10 users + 56 collections to mbs_platform + inner_lab)
   - Step B: Refactor CWG/ (JWT middleware, new DB, remove standalone auth + billing)
   - See ORCHESTRATION_GUIDE.md Phase 3 section for exact prompts

2. **Phase 4: FlowState Migration** — READY. Can run in parallel with Phase 3.
   - Same two-step pattern, simpler (0 users, 7 collections)
   - See ORCHESTRATION_GUIDE.md Phase 4 section

3. **Phase 5: Standalone Products** — READY. Can run in parallel with Phase 3+4.
   - 11 independent products (5 Arcade + 6 Studio Works)
   - SSO migration only — no data migration, no il_* involvement
   - See ORCHESTRATION_GUIDE.md Phase 5 section for product list

### Before Phase 3 — one manual prerequisite:
- Create Stripe products for IL All Access + MBS All Access in Stripe Dashboard (CWG pricing already exists)
- Without this, those checkout buttons fail — but CWG access (free_tier + product_pass) still works

### Sync Status (verified this session)
| Project | Sync Status |
|---------|-------------|
| MBS/platform-instructions/ | **IN SYNC** (updated with addendum #14-15) |
| Innerlab/platform-instructions/ | **IN SYNC** (updated with login/signup page spec) |
| CWG/platform-instructions/ | **IN SYNC** (updated with user copy-across + auth_methods) |
| YogaGhost/platform-instructions/ | **IN SYNC** (updated with login redirect) |

---

## ORCHESTRATION WORKFLOW

1. Open Claude Code in project folder
2. Agent reads `platform-instructions/` and builds
3. Agent generates `PHASE_X_REPORT.md` when done
4. **Bring report to orchestrator session** (MBSPlatform/) before starting next phase
5. Orchestrator reviews, cascades learnings to downstream instructions if needed
6. Re-sync updated instructions to target folders
7. Proceed to next phase

---

## WHERE CODE GETS BUILT

| Folder | Domain | What gets added | Database | Phase |
|--------|--------|----------------|----------|-------|
| `MBS/` | magicbusstudios.com | SSO + billing + entitlements + login/signup | `mbs_platform` | 1 ✅ DONE |
| `Innerlab/` | innerlab.ai | il_* APIs + dashboard + login/signup pages | `inner_lab` | 2 ✅ DONE |
| `CWG/` | conversationswithgod.ai | Refactored to use platform auth + inner_lab DB | `inner_lab` (cwg_*) | 3 |
| `YogaGhost/` | yoga.magicbusstudios.com | Refactored to use platform auth + inner_lab DB | `inner_lab` (yoga_*) | 4 |
| 5 Arcade games | *.magicbusstudios.com | SSO migration (JWT middleware) | Own DBs | 5 |
| 6 Studio Works apps | *.magicbusstudios.com | SSO migration (JWT middleware) | Own DBs | 5 |
