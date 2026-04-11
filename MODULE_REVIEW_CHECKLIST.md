# Inner Lab Module — Coolify Instructions Review Checklist

> **Purpose**: When reviewing compile/deployment instructions from IL module agents, check every item below. This catches the recurring mistakes agents make. Created Session 32 (April 10, 2026) after reviewing Rituals, StarMap, InnerQuest, LifeMap, and Archetypes.
>
> **How to use**: Module agents generate Coolify deployment instructions. Owner pastes them to the MBS Platform agent (this repo) for review. This checklist is what the reviewer uses. When corrections are needed, a prompt is generated and pasted back to the module agent.

---

## Domain Convention (decided 2026-04-10)

| Check | Correct | Common Mistake |
|-------|---------|----------------|
| Frontend domain | `{slug}.innerlab.ai` | `{slug}.magicbusstudios.com` |
| Backend domain | `api.{slug}.innerlab.ai` | `api.{slug}.magicbusstudios.com` |
| DNS A records | Both point to VPS IP under `*.innerlab.ai` | Listed under `*.magicbusstudios.com` |
| CORS_ORIGINS | `https://{slug}.innerlab.ai,https://www.{slug}.innerlab.ai` | Missing www variant, or using magicbusstudios.com |
| VITE_BACKEND_URL | `https://api.{slug}.innerlab.ai` | Using magicbusstudios.com |
| Health check URL | `https://api.{slug}.innerlab.ai/health` | Using magicbusstudios.com |
| Post-deploy verify URLs | All under `*.innerlab.ai` | Leftover magicbusstudios.com |
| nginx www redirect | `www.{slug}.innerlab.ai` → `{slug}.innerlab.ai` | Wrong domain |

**Historical exceptions (keep as-is):**
- CWG: `conversationswithgod.ai`
- FlowState: `yoga.magicbusstudios.com`

**Do NOT change these — they still point to MBS Platform:**
- `PLATFORM_URL` = `https://api.magicbusstudios.com` (backend entitlement checks)
- `VITE_PLATFORM_URL` = `https://magicbusstudios.com` (frontend subscribe/billing redirects)

---

## JWT (Dual-Mode RS256 + HS256)

| Check | Correct | Common Mistake |
|-------|---------|----------------|
| Both keys listed | `JWT_PUBLIC_KEY` AND `JWT_SECRET` both present | Listed as "either/or" or "optional" |
| JWT note | "Both required. RS256 primary, HS256 fallback, dual-mode still active." | "Only one required" or "use either" |
| JWT_PUBLIC_KEY format | Note says: **Mark "Is Multiline?" in Coolify** | Says "`\n` for newlines" or "`\\n` escaped" |

---

## GDPR

| Check | Correct | Common Mistake |
|-------|---------|----------------|
| DELETE endpoint mentioned | `DELETE /api/user-data` (authenticated) | Missing entirely |
| Scope | Deletes all `{prefix}_*` data + `il_*` entries where `source_module: "{slug}"` | Only mentions module prefix, misses il_* |
| Post-deploy verification | Includes GDPR delete test | Not mentioned |

---

## Environment Variables

| Variable | Correct Value | Common Mistake |
|----------|---------------|----------------|
| `MONGO_URL` | Shared MongoDB connection string | - |
| `DB_NAME` | `inner_lab` | - |
| `JWT_PUBLIC_KEY` | PEM format, "Is Multiline?" in Coolify | Manual `\n` escaping |
| `JWT_SECRET` | Same as MBS Platform | Listed as optional |
| `PORT` | `4000` | - |
| `CORS_ORIGINS` | Both www and non-www `*.innerlab.ai` | Single origin or magicbusstudios.com |
| `PLATFORM_URL` | `https://api.magicbusstudios.com` | Changed to innerlab.ai |
| `PRODUCT_SLUG` | Module slug | - |
| `SERVICE_NAME` | `{slug}-api` | Missing |
| `OPENAI_MODEL` | `gpt-4o-mini` | Hallucinated models (gpt-5.4-mini, gpt-4o full) |
| `LOG_LEVEL` | `info` | - |

---

## Frontend Build Args

| Variable | Correct Value | Common Mistake |
|----------|---------------|----------------|
| `VITE_BACKEND_URL` | `https://api.{slug}.innerlab.ai` | magicbusstudios.com |
| `VITE_PLATFORM_URL` | `https://magicbusstudios.com` | Changed to innerlab.ai (this SHOULD be magicbusstudios.com) |
| `VITE_PRODUCT_SLUG` | `{slug}` | Missing ("defaults in code") |

**Critical distinction agents confuse**: `VITE_PLATFORM_URL` points to MBS Platform (for subscribe/billing), NOT to the module itself. It stays at `magicbusstudios.com` always.

---

## Other Checks

| Check | Correct | Common Mistake |
|-------|---------|----------------|
| No invented env vars | Only standard MBS env vars | `ALLOW_TRANSPORT_ONLY_ACCESS` and similar |
| Pre-deploy: register product | Admin dashboard → Products tab | - |
| Pre-deploy: NO "add to MBS CORS" | Module doesn't need MBS CORS entry | "Add module domain to MBS Platform CORS" |
| Auth delegation | Login redirects to `innerlab.ai/auth/login` | - |
| `VITE_GOOGLE_CLIENT_ID` | Not needed (auth delegated) | Included |

---

## Correction Prompt Template

When corrections are needed, use this structure:

```
## Deployment Instructions — Corrections Required

Your instructions have N issues. Apply all corrections and regenerate.

### 1. [Issue title]
[Explanation + correct value]

### 2. [Issue title]
...

### What to do next

Apply all N corrections and regenerate the complete Coolify deployment instructions 
in the same format. If anything is unclear or you have questions about MBS Platform 
conventions, respond with your questions instead. Your questions will be routed to 
the MBS Platform architecture agent, which is the source of truth. Don't guess — ask.
```

---

## Modules Reviewed (Session 32)

| Module | Slug | Prefix | Issues Found | Status |
|--------|------|--------|-------------|--------|
| Rituals | `rituals` | `ritual_*` | 5 (domain, JWT either/or, no CORS www, ALLOW_TRANSPORT_ONLY_ACCESS, no GDPR) | ✅ Corrected — v2 instructions approved |
| StarMap | `starmap` | `astro_*` | 5 (domain, hallucinated model, VITE_PLATFORM_URL, JWT_PUBLIC_KEY note, no GDPR) | ⏳ Correction sent (twice — agent ignored first) |
| InnerQuest | `innerquest` | `quest_*` | 4 (domain, JWT either/or, CORS single origin, JWT_PUBLIC_KEY escaping) | ⏳ Correction sent |
| LifeMap | `lifemap` | `lifemap_*` | 7 (domain, JWT either/or, JWT_PUBLIC_KEY escaping, model gpt-4o, no VITE_PRODUCT_SLUG, GDPR scope, MBS CORS line) | ⏳ Correction sent |
| Archetypes | `archetypes` | `archetype_*` | 5 (domain, JWT either/or, JWT_PUBLIC_KEY escaping, CORS single origin, no GDPR + MBS CORS line) | ⏳ Correction sent |

### Modules Not Yet Reviewed
- Bonds (`bonds_*`)
- AstroCompass (`astrocart_*`)
- Arcana (`tarot_*`)
- DreamLens (`dream_*`)
- Nexus (`nexus_*`)
