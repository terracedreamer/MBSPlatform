# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: March 25, 2026 (Session 1 — complete)
**Git Branch**: main
**Last Commit**: pending (end-of-session push)
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## WHAT WAS DONE THIS SESSION

### Architecture Decisions (17 total, all finalized)
- Three-layer architecture: MBS Platform (Layer 1) → Inner Lab Middleware (Layer 2) → Standalone Products (Layer 3)
- MBS Platform merges into existing MBS/ website (magicbusstudios.com) — 2 containers
- Inner Lab Middleware merges into existing Innerlab/ website (innerlab.ai) — 2 containers
- Auth: Google SSO + Nostr + LNURL (all passwordless)
- Payments: Stripe + BTCPay (Lightning) — both platform-level
- Branded login: IL modules → Inner Lab branding, Arcade/SW → MBS branding
- Friends/invites at platform level, activity feed at Inner Lab level
- User memories: per-module private by default, user opt-in to share across IL
- Fresh inner_lab database, old DBs stay as backups
- All Inner Lab modules = separate containers, shared database
- Reviewed CWG (56 collections) and FlowState (7 collections) schemas and migration plans

### Files Created
- `platform-instructions-for-mbs/` — Layer 1 build instructions (copied to MBS/)
- `platform-instructions-for-innerlab/` — Layer 2 build instructions (copied to Innerlab/)
- `platform-instructions-for-cwg/` — CWG migration instructions (copied to CWG/)
- `platform-instructions-for-yogaghost/` — FlowState migration instructions (copied to YogaGhost/)
- `platform-instructions-for-new-modules/` — Starter kit for any new Inner Lab module
- 4 marketing briefs converted from .docx to .md with platform context integrated
- Marketing README.md with agent instructions
- Global CLAUDE.md updated with end-session protocol and branch safety

### Issues Found & Fixed
- JWT_SECRET must be shared across all projects
- Migration scripts run from MBS/, not from individual products
- CWG uses UUID strings (not ObjectId) — migration must map IDs
- CWG is Python (FastAPI) — JWT middleware needs Python implementation
- Inner Lab HAS a backend (corrected from earlier "no backend" error)
- Added explicit prerequisites to CWG/FlowState migration docs

---

## NEXT SESSION TASKS

### Immediate (start with these)
- [ ] **Enhance marketing briefs** — check innerlab.ai and magicbusstudios.com for missing product info. Absorb Arcade brief into MBS master doc. Add Inner Lab dashboard/module connection details to IL brief. Focus on PRODUCT INFORMATION, not marketing strategy.
- [ ] **Convert CWG marketing plan .docx to .md** in Desktop/Marketing/Inner Lab/Conversations with God/ (currently old .docx)
- [ ] **Begin MBS Platform build** — open Claude Code in MBS/ folder, agent reads platform-instructions/CLAUDE.md

### Implementation Order (strict sequence)
1. Build MBS Platform (MBS/ folder) — SSO + billing + entitlements
2. Build Inner Lab Middleware (Innerlab/ folder) — il_* collections + APIs + dashboard
3. CWG Migration — scripts from MBS/, then CWG agent refactors
4. FlowState Migration — scripts from MBS/, then FlowState agent refactors
5. New modules — use platform-instructions-for-new-modules/ starter kit

---

## REPO STRUCTURE

```
MBSPlatform/
├── platform-instructions-for-mbs/          ← Copied to MBS/platform-instructions/
├── platform-instructions-for-innerlab/     ← Copied to Innerlab/platform-instructions/
├── platform-instructions-for-cwg/          ← Copied to CWG/platform-instructions/
├── platform-instructions-for-yogaghost/    ← Copied to YogaGhost/platform-instructions/
├── platform-instructions-for-new-modules/  ← Starter kit for future modules
├── marketing-docs/                         ← Product briefs (synced to Desktop/Marketing/Overview/)
├── archive/                                ← Old reference material
├── CLAUDE.md, SESSION_HANDOFF.md, CHANGELOG.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md
└── .gitignore
```

## WHERE CODE GETS BUILT

| Folder | Domain | What gets added | Database |
|--------|--------|----------------|----------|
| `MBS/` | magicbusstudios.com | SSO + billing + entitlements + login + admin | `mbs_platform` |
| `Innerlab/` | innerlab.ai | il_* APIs + dashboard + consciousness + memories | `inner_lab` |
| `CWG/` | conversationswithgod.ai | Refactored to use platform auth + inner_lab DB | `inner_lab` (cwg_*) |
| `YogaGhost/` | yoga.magicbusstudios.com | Refactored to use platform auth + inner_lab DB | `inner_lab` (yoga_*) |
| New modules | TBD | Born into architecture using starter kit | `inner_lab` ({prefix}_*) |
