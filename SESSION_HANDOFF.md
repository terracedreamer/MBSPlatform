# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: March 26, 2026 (End of Session 2)
**Git Branch**: main
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## WHERE WE ARE RIGHT NOW

### Phase 1: MBS Platform — MOSTLY COMPLETE
- **Built and deployed** at magicbusstudios.com: Google SSO, JWT auth, entitlements, Stripe checkout, BTCPay checkout, friends/invites, branded login page, GDPR delete cascade, health check
- **Phase 1 Addendum** (13 items) added to `platform-instructions-for-mbs/CLAUDE.md` — these are additional items to build in MBS/ before moving on:
  1. Login button in nav (**DONE** — pushed to MBS/)
  2. Rate limiting on platform routes (**DONE** — already existed)
  3. Lightning payment toggle in billing UI (**DONE**)
  4. Nostr authentication routes (**DONE** — needs real-world testing)
  5. LNURL-Auth routes (**DONE** — needs real-world testing)
  6. Auth method linking (**DONE**)
  7. Email preferences + one-click unsubscribe (**DONE**)
  8. Transactional emails via SendGrid (6 types) (**DONE**)
  9. Basic admin panel (**DONE**)
  10. Promo code routes (**DONE**)
  11. Referral routes (**DONE**)
  12. Token refresh endpoint (**DONE**)
  13. Structured billing page with real CWG pricing (**DONE**)
- **Phase 1 report**: `MBS/phase-reports/PHASE_1_REPORT.md` and `MBSPlatform/phase-reports/PHASE_1_LEARNINGS.md`

### Known Issues from Phase 1
- **BTCPay 403 error**: API key has insufficient permissions. Lightning payments fail. Need to regenerate BTCPay API key with full store permissions. Does NOT block Phase 2.
- **Stripe bundle price IDs**: CWG prices work ($9.99/mo, $79.99/yr). IL All Access and MBS All Access price IDs need to be created in Stripe Dashboard before those checkout buttons work.
- **package-lock.json out of sync**: After installing nostr-tools + @noble/secp256k1, the lock file needs regeneration (`npm install` in MBS/server/) before next deploy.

### Phase 2: Inner Lab Middleware — NOT STARTED
- Instructions are ready and synced at `Innerlab/platform-instructions/CLAUDE.md`
- Phase 1 learnings already baked into the instructions
- **Start this in a separate Claude Code session** pointing to `Innerlab/` folder

### Phases 3-5: Not started
- All instructions synced to their target project folders
- All have Phase 1 learnings baked in

---

## WHAT THE NEXT SESSION NEEDS TO DO

### Option A: Continue as Orchestrator (new session in MBSPlatform/)
1. Open separate Claude Code sessions for each phase (one at a time)
2. Phase 2: `Innerlab/` — paste prompt from `ORCHESTRATION_GUIDE.md`
3. After Phase 2 finishes, review `PHASE_2_REPORT.md`, update downstream instructions if needed
4. Phase 3: `CWG/` migration
5. Phase 4: `YogaGhost/` migration
6. Phase 5: 11 standalone products (can parallel)

### Option B: Build directly from orchestrator (if context allows)
- The orchestrator session CAN read/write all project folders
- Good for small tasks, reviews, and syncing
- Not recommended for large builds (Phase 2 is a large build — backend + dashboard)

### Sync Status (verified just now)
| Project | Sync Status |
|---------|-------------|
| MBS/platform-instructions/ | IN SYNC |
| Innerlab/platform-instructions/ | IN SYNC |
| CWG/platform-instructions/ | IN SYNC |
| YogaGhost/platform-instructions/ | IN SYNC |
| Desktop/Marketing/Overview/ | IN SYNC |

---

## ORCHESTRATION WORKFLOW (established this session)

1. Open Claude Code in project folder
2. Agent reads `platform-instructions/` and builds
3. Agent generates `PHASE_X_REPORT.md` when done
4. **Bring report to orchestrator session** (MBSPlatform/) before starting next phase
5. Orchestrator reviews, cascades learnings to downstream instructions if needed
6. Re-sync updated instructions to target folders
7. Proceed to next phase

---

## REPO STRUCTURE

```
MBSPlatform/
├── platform-instructions-for-mbs/              ← Copied to MBS/platform-instructions/
├── platform-instructions-for-innerlab/         ← Copied to Innerlab/platform-instructions/
├── platform-instructions-for-cwg/              ← Copied to CWG/platform-instructions/
├── platform-instructions-for-yogaghost/        ← Copied to YogaGhost/platform-instructions/
├── platform-instructions-for-standalone-products/ ← Copied to each Arcade/SW project
├── platform-instructions-for-new-modules/      ← Starter kit for future IL modules
├── marketing-docs/                             ← Product briefs (synced to Desktop/Marketing/Overview/)
├── phase-reports/                              ← Phase completion reports (copied from project folders)
├── archive/                                    ← Old reference material
├── ORCHESTRATION_GUIDE.md                      ← Copy-paste prompts for each Claude Code session
├── CLAUDE.md, SESSION_HANDOFF.md, CHANGELOG.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md
└── .gitignore
```

## WHERE CODE GETS BUILT

| Folder | Domain | What gets added | Database | Phase |
|--------|--------|----------------|----------|-------|
| `MBS/` | magicbusstudios.com | SSO + billing + entitlements + login | `mbs_platform` | 1 (mostly done) |
| `Innerlab/` | innerlab.ai | il_* APIs + dashboard + consciousness + memories | `inner_lab` | 2 (next) |
| `CWG/` | conversationswithgod.ai | Refactored to use platform auth + inner_lab DB | `inner_lab` (cwg_*) | 3 |
| `YogaGhost/` | yoga.magicbusstudios.com | Refactored to use platform auth + inner_lab DB | `inner_lab` (yoga_*) | 4 |
| 5 Arcade games | *.magicbusstudios.com | SSO migration (JWT middleware) | Own DBs | 5 |
| 6 Studio Works apps | *.magicbusstudios.com | SSO migration (JWT middleware) | Own DBs | 5 |
