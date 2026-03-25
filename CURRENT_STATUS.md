# CURRENT STATUS — MBS Platform

**Last Updated**: March 25, 2026

## Project State: Architecture Complete — Ready for Phase 1 Build

All 17 architecture decisions are finalized. No open questions. No code written yet.

## What Exists

| Component | Status | Notes |
|-----------|--------|-------|
| MBS Platform backend | Not started | Spec complete. Ready to build. |
| Inner Lab Middleware | Not started | Reference CLAUDE.md created. Build when 2-3 modules exist. |
| CWG backend | Exists (separate repo) | 56 collections, ~10 users, sovereignty features. Migration plan in `CWG/MBS_DATABASE_MIGRATION_PLAN.md` |
| FlowState backend | Exists (separate repo) | 7 collections, 0 real users. Migration plan in `YogaGhost/MBS_DATABASE_MIGRATION_PLAN.md` |
| innerlab.ai | Live marketing site | No backend, no auth |
| Other Inner Lab modules | Not started | Will be born into new architecture |
| Arcade games (5) | Exist independently | No changes until Phase 4 |
| Studio Works tools (6) | Exist independently | No changes until Phase 4 |

## Infrastructure

| Component | Status |
|-----------|--------|
| MongoDB | Self-hosted on Coolify (`conversations-mongodb` container) |
| Coolify server | Running (localhost) |
| GitHub repo | https://github.com/terracedreamer/MBSPlatform.git |

## Key Decisions That Affect Implementation

- Auth: Google SSO + Nostr + LNURL-Auth (no email/password)
- Payments: Stripe + BTCPay (Lightning) — both platform-level
- Friends/Invites: Platform-level with originating product context
- Push subscriptions & feature flags: Platform-level
- Branded login: `?brand=innerlab` or `?brand=mbs` parameter
- Activity feed: Inner Lab level (not platform, not per-product)
- User memories: Inner Lab level with user opt-in sharing
- Encryption/export: Inner Lab level

## Environment Variables Needed (MBS Platform — when built)

- MONGODB_URI, DB_NAME (`mbs_platform`)
- JWT_SECRET, JWT_EXPIRY
- GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- BTCPAY_URL, BTCPAY_API_KEY, BTCPAY_STORE_ID
- SENDGRID_API_KEY, FROM_EMAIL
- PORT (3002), CORS_ORIGINS (all product URLs)

## Next Steps

1. Begin MBS Platform Phase 1 implementation (Express + Mongoose scaffolding)
2. Implement Google SSO + JWT issuance
3. Implement entitlements system
4. Implement Stripe checkout + webhooks
5. Implement BTCPay Lightning payments
6. Write CWG + FlowState migration scripts
7. Deploy to Coolify at platform.magicbusstudios.com
