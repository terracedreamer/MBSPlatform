# SESSION HANDOFF — MBS Platform Architecture Planning

**Last Updated**: March 25, 2026 (Session 1 — complete)
**Session Type**: Architecture planning (no code written yet)
**Git Branch**: main
**GitHub Repo**: https://github.com/terracedreamer/MBSPlatform.git

---

## WHERE WE LEFT OFF

All architecture decisions are **finalized**. No open questions remain.

**Next steps (in order):**
1. Build MBS Platform backend (Phase 1 MVP) — Google SSO + Nostr/LNURL + entitlements + Stripe + BTCPay
2. Build Inner Lab Middleware backend (Layer 2) — shared il_* collections, consciousness profiles, user memories, check-ins, wellness profiles. **Build NOW** so migrations write shared data correctly from day one.
3. Write migration scripts for CWG (56 collections) and FlowState (7 collections) — shared data goes to il_* via middleware, product data to cwg_*/yoga_*
4. Refactor CWG and FlowState backends to use platform auth + middleware APIs
5. Build Inner Lab **frontend** (innerlab.ai dashboard) — LATER when 2-3 modules have enough data

**Migration analysis docs created by product agents:**
- `CWG/MBS_DATABASE_MIGRATION_PLAN.md` — 56 collections categorized into 3 buckets
- `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md` — 7 collections categorized into 3 buckets

---

## THREE-LAYER ARCHITECTURE (Finalized)

```
┌──────────────────────────────────────────────────────────────┐
│  Layer 1: MBS Platform  (DB: mbs_platform)                   │
│  SSO + billing + entitlements — serves ALL 22+ products      │
│  Domain: platform.magicbusstudios.com:3002                   │
│                                                               │
│  Auth: Google SSO + Nostr + LNURL-Auth                       │
│  Payments: Stripe + BTCPay (Lightning)                       │
│  Also: friends, invites, push subscriptions, feature flags,  │
│         promo codes, referrals, GDPR consent/data requests   │
└──────────┬──────────────────────────┬────────────────────────┘
           │                          │
┌──────────▼──────────┐       ┌───────▼──────────────────────┐
│  Layer 2: Inner Lab │       │  Layer 3: Standalone         │
│  Middleware          │       │  Products                    │
│  (DB: inner_lab)    │       │                              │
│                      │       │  Arcade (5 games) — own DBs  │
│  Shared:            │       │  Studio Works (6 tools)      │
│  - consciousness    │       │    — own DBs                 │
│  - user memories    │       │                              │
│    (opt-in)         │       │  SSO only via MBS Platform.  │
│  - check-ins        │       │  No shared data between them.│
│  - activity feed    │       │                              │
│  - daily briefing   │       └──────────────────────────────┘
│  - encryption/      │
│    data export      │
│                      │
│  Module data:       │
│  cwg_*, yoga_*,     │
│  breath_*, etc.     │
│                      │
│  Each module =      │
│  separate container │
│  Same database      │
└──────────────────────┘
```

---

## ALL ARCHITECTURE DECISIONS (Confirmed 2026-03-25)

| # | Decision | Details |
|---|----------|---------|
| 1 | Fresh `inner_lab` database | CWG data migrates with `cwg_` prefix. Old DB stays as backup. |
| 2 | User identity ONLY in `mbs_platform` | Single source of truth. No duplicate user tables. |
| 3 | Shared IL data starts as module-prefixed | Everything starts as `cwg_*` or `yoga_*`. Promote to `il_*` when second module needs it. |
| 4 | Inner Lab Middleware backend — build NOW (after MBS Platform) | So migrations write shared data to il_* from day one. Inner Lab FRONTEND comes later. |
| 5 | Frontend: B+C hybrid | Modules standalone at own domains. `innerlab.ai` becomes premium dashboard later. |
| 6 | Branded login | Inner Lab modules → IL branding. Arcade/SW → MBS branding. Via `?brand=` param. |
| 7 | Arcade/Studio Works | SSO only. No shared data. Independent. |
| 8 | Admin | Simple `is_admin` flag. |
| 9 | Nostr/LNURL auth | MBS Platform level — all products. |
| 10 | Payments | Stripe + BTCPay both at MBS Platform. No per-product payment handling. |
| 11 | Friends/Invites | MBS Platform level — with originating product context. |
| 12 | Activity feed | Inner Lab level. No feed for Arcade/SW. Built when IL frontend exists. |
| 13 | Push subscriptions | MBS Platform level. |
| 14 | Feature flags | MBS Platform level. |
| 15 | User memories (AI) | Inner Lab level. Option C: per-module by default, user opt-in to share across IL. |
| 16 | Encryption & data export | Inner Lab level — all IL modules are sensitive. |
| 17 | Modules as containers | Each = separate Coolify container. All share `inner_lab` DB. |

---

## CWG SOVEREIGNTY FEATURES (Move to Platform)

CWG has experimental (untested) sovereignty infrastructure:
- **Nostr identity**: `nostr_npub`, `nostr_challenges`, `nostr_events`
- **LNURL-Auth**: `lnurl_linking_key`, `lnurl_challenges`
- **Lightning payments**: `btcpay_invoices`, `btcpay_tips`, `btcpay_donations`, `btcpay_chat_sessions`
- **Client encryption**: `user_encryption`, `encrypted_backups`
- **Blockchain anchors**: OpenTimestamps data integrity proofs
- **GDPR**: `consent_audit_log`, `data_requests`

These move to MBS Platform (Layer 1) for auth/payments/GDPR, or Inner Lab middleware (Layer 2) for encryption/export.

---

## MODULE STATUS

| Module | Database | Collections | Real Users | Migration |
|--------|----------|-------------|------------|-----------|
| CWG | `conversations_with_god` | 56 | ~10 friends | Split: users→platform, product→inner_lab cwg_* |
| FlowState | `yogaghost` | 7 | 0 | Split: users→platform, product→inner_lab yoga_* |
| Others | Don't exist | — | — | Born into new architecture |

---

## INFRASTRUCTURE

- MongoDB: **self-hosted on Coolify** (NOT Atlas) — container `conversations-mongodb`
- All apps on one Coolify server (localhost)
- CWG backend: FastAPI + Motor (Python)
- FlowState backend: Express 5 + native MongoDB driver (Node.js)
- MBS Platform: Express + Mongoose (Node.js) — to be built

---

## FILES IN THIS REPO

| File/Folder | Purpose |
|-------------|---------|
| `CLAUDE.md` | Project instructions for Claude agents |
| `SESSION_HANDOFF.md` | This file — session continuity |
| `MBS_Platform_Technical_Architecture.docx` | Original Layer 1 spec |
| `ChatGPT-architecture/` | Inner Lab shared intelligence discussion |
| `marketing-docs/` | Product briefs (Inner Lab, CWG, Arcade, MBS brand) |
| `InnerLab-middleware/` | **Reference folder for building Layer 2** (copy to new project) |
| `InnerLab-middleware/CLAUDE.md` | Full CLAUDE.md for the Inner Lab middleware project |
