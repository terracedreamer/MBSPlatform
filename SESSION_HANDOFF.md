# SESSION HANDOFF — MBS Platform Architecture Think Tank

**Last Updated**: March 26, 2026 (Session 2 — ongoing)
**Git Branch**: main
**Last Commit**: c3ffe49
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git
**Repo Purpose**: Architecture think tank — no code here. Reference files get copied to actual projects.

---

## WHAT WAS DONE THIS SESSION (Session 2)

### Full Architecture Review (6 audit passes, 50+ issues found and fixed)
- Pass 1: Fixed 7 issues — stale folder names, wrong URLs, contradictory build-later/build-now
- Pass 2: Fixed 6 cross-document inconsistencies — redundant collections, memory schema mismatch, missing API endpoints
- Pass 3: Fixed 3 HIGH + 7 MEDIUM — open redirect protection, free_tier logic, dual write paths, /api/ prefixes
- Pass 4: Fixed 4 stale docs + 3 architectural specs — deep links, token refresh, entitlement resilience
- Pass 5: Fixed 11 field mismatches — avatar/picture, google_id/googleId, JWT payload undefined, CWG cutover
- Pass 6 (adversarial): Fixed 3 CRITICAL + 11 HIGH + 10 MEDIUM — GDPR cascade, il_* schema contracts, deployment checklist, JWT security, BTCPay→Entitlement flow, CORS enumeration

### Marketing Brief Enhancements
- Absorbed Arcade detail into MBS master doc (URLs, taglines, pricing, target audience)
- Expanded Studio Works from one-liners to full descriptions
- Added Inner Lab Dashboard section (module launcher, activity feed, cross-module intelligence, memory privacy)
- Enhanced FlowState description (breathwork, streaks, achievements, programs)
- Added design language section to IL brief (exact colors, typography from live site)
- Synced all briefs to Desktop/Marketing/Overview/

### New Files Created
- `platform-instructions-for-standalone-products/` — SSO migration guide for 11 Arcade + Studio Works apps
- `ORCHESTRATION_GUIDE.md` — Step-by-step guide with exact copy-paste prompts for each Claude Code session

### All Instruction Sets Now Include
- JWT payload specification (userId, email, name, avatar, isAdmin)
- il_* schema contracts (check-ins, consciousness, histories, wellness, activity feed)
- Open redirect protection spec
- GDPR deletion cascade (cross-database)
- Entitlement response spec with full reason enum and free tier logic
- BTCPay → Entitlement flow
- Deployment checklist
- CORS complete domain enumeration (18 domains)
- Entitlement cache spec (5min TTL, ?refresh=true invalidation)
- CWG cutover sequence (simplified for ~10 users)
- MongoDB-level JSON Schema validation recommendations
- **Completion report requirements** — each phase agent auto-generates a PHASE_X_REPORT.md when done

### Marketing Brief Updates
- All 4 briefs updated to March 26, 2026
- Arcade brief game descriptions synced with live magicbusstudios.com copy
- Trivia Roast URL typo note added to Arcade brief

---

## NEXT STEPS — BUILD PHASE 1

JWT_SECRET has been generated. Follow `ORCHESTRATION_GUIDE.md` for exact prompts.

### Orchestration Workflow
1. Open Claude Code in project folder, paste prompt from orchestration guide
2. Agent builds and pushes code (auto-deploys via Coolify)
3. Agent generates `PHASE_X_REPORT.md` in project root
4. **Bring report back to this orchestrator session** (MBSPlatform) before starting next phase
5. Orchestrator reviews, updates downstream instructions if needed
6. Proceed to next phase

### Build Sequence
1. **Phase 1: MBS Platform** → `MBS/` — SSO, billing, entitlements (START HERE)
2. **Phase 2: IL Middleware** → `Innerlab/` — il_* APIs + dashboard
3. **Phase 3: CWG Migration** → migration script from `MBS/`, then refactor `CWG/`
4. **Phase 4: FlowState Migration** → migration script from `MBS/`, then refactor `YogaGhost/`
5. **Phase 5: 11 Standalone Products** → can all run in parallel after Phase 1

### Manual Steps Between Phases
- Set env vars in Coolify for each service (JWT_SECRET, etc.)
- Test the deployed service works
- Bring the PHASE_X_REPORT.md back to the orchestrator for review

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
├── archive/                                    ← Old reference material
├── ORCHESTRATION_GUIDE.md                      ← Copy-paste prompts for each Claude Code session
├── CLAUDE.md, SESSION_HANDOFF.md, CHANGELOG.md, CURRENT_STATUS.md, FUTURE_WORK_TODO.md
└── .gitignore
```

## WHERE CODE GETS BUILT

| Folder | Domain | What gets added | Database |
|--------|--------|----------------|----------|
| `MBS/` | magicbusstudios.com | SSO + billing + entitlements + login + migration scripts | `mbs_platform` |
| `Innerlab/` | innerlab.ai | il_* APIs + dashboard + consciousness + memories | `inner_lab` |
| `CWG/` | conversationswithgod.ai | Refactored to use platform auth + inner_lab DB | `inner_lab` (cwg_*) |
| `YogaGhost/` | yoga.magicbusstudios.com | Refactored to use platform auth + inner_lab DB | `inner_lab` (yoga_*) |
| 5 Arcade games | *.magicbusstudios.com | SSO migration (JWT middleware, remove standalone auth) | Own DBs |
| 6 Studio Works apps | *.magicbusstudios.com | SSO migration (JWT middleware, remove standalone auth) | Own DBs |
| New IL modules | TBD | Born into architecture using starter kit | `inner_lab` ({prefix}_*) |
