# CURRENT STATUS — MBS Platform

**Last Updated**: March 24, 2026

## Project State: Architecture Planning (Pre-Development)

No code has been written. All architecture decisions are documented in SESSION_HANDOFF.md.

## What Exists

| Component | Status |
|-----------|--------|
| MBS Platform backend | Not started — spec only |
| Inner Lab Core | Not started — will build later |
| CWG backend | Exists (separate repo/deployment) — needs refactoring |
| CWG database | `conversations_with_god` DB exists with ~10 users |
| FlowState/Yoga | Exists (separate repo/deployment) — minimal data |
| innerlab.ai | Live marketing site — no backend, no auth |
| Other Inner Lab modules | Not started |
| Arcade games | Exist independently — no changes needed yet |
| Studio Works tools | Exist independently — no changes needed yet |

## Known Issues

- CWG currently has its own auth system — needs migration to MBS Platform
- CWG has its own Stripe integration being built — must move to MBS Platform
- CWG database schema needs inspection before migration planning
- No shared data schema designed yet for Inner Lab

## Environment Variables Needed (MBS Platform — when built)

- MONGODB_URI, DB_NAME (`mbs_platform`)
- JWT_SECRET, JWT_EXPIRY
- GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- SENDGRID_API_KEY, FROM_EMAIL
- PORT (3002), CORS_ORIGINS (all product URLs)

## Next Steps

1. Inspect CWG MongoDB database — understand existing collections and fields
2. Design data migration plan (CWG → mbs_platform users + inner_lab cwg_* tables)
3. Begin MBS Platform Phase 1 implementation
