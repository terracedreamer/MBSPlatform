# Proactive Prompt — For Remaining Inner Lab Module Agents

> **Context**: We reviewed 5 IL module agent sessions (Rituals, StarMap, InnerQuest, LifeMap, Archetypes) and found every single one hit the same deployment failures and Coolify instruction mistakes. This prompt prevents the remaining 5 modules from repeating those errors.
>
> **How to use**: Paste the prompt section below into each remaining IL module agent session. Replace `{slug}`, `{prefix}`, and `{ModuleName}` with the module's actual values before pasting.

## Module Values (replace before pasting)

| Module | `{ModuleName}` | `{slug}` | `{prefix}` |
|--------|-----------------|-----------|-------------|
| Bonds | Bonds | `bonds` | `bonds_` |
| AstroCompass | AstroCompass | `astrocompass` | `astrocart_` |
| Arcana | Arcana | `arcana` | `tarot_` |
| DreamLens | DreamLens | `dreamlens` | `dream_` |
| Nexus | Nexus | `nexus` | `nexus_` |

---

## Prompt (copy everything below this line)

---

## MBS Platform Architecture — Deployment Standards + Workflow for {ModuleName}

I've just finished reviewing deployment instructions from 5 other Inner Lab module agents. Every single one made the same mistakes — incorrect domains, broken Dockerfiles, wrong env vars. This message gives you the correct standards so you can get it right the first time.

**Read all 7 parts before generating any Coolify instructions or Dockerfiles.**

---

### Part 1: Domain Convention

**All Inner Lab modules deploy to `*.innerlab.ai`**, NOT `*.magicbusstudios.com`.

| What | Correct | Wrong |
|------|---------|-------|
| Frontend domain | `{slug}.innerlab.ai` | `{slug}.magicbusstudios.com` |
| Backend domain | `api.{slug}.innerlab.ai` | `api.{slug}.magicbusstudios.com` |
| DNS A records | Under `*.innerlab.ai` | Under `*.magicbusstudios.com` |
| CORS_ORIGINS | `https://{slug}.innerlab.ai,https://www.{slug}.innerlab.ai` | Single origin, or magicbusstudios.com |
| VITE_BACKEND_URL | `https://api.{slug}.innerlab.ai` | magicbusstudios.com |
| Health check URL | `https://api.{slug}.innerlab.ai/health` | magicbusstudios.com |

**Critical exception — these still point to MBS Platform (do NOT change to innerlab.ai):**
- `PLATFORM_URL` = `https://api.magicbusstudios.com` (backend entitlement checks)
- `VITE_PLATFORM_URL` = `https://magicbusstudios.com` (frontend subscribe/billing redirects)

`PLATFORM_URL` and `VITE_PLATFORM_URL` are the MBS Platform's own addresses for billing and entitlements. Your module *calls* them, but your module *lives at* `*.innerlab.ai`. Every module so far confused this — don't repeat it.

---

### Part 2: JWT — Both Keys Required

MBS Platform uses **dual-mode JWT verification**: RS256 primary, HS256 fallback. Both are still active across all 15+ services.

| Env Var | Required? | Notes |
|---------|-----------|-------|
| `JWT_PUBLIC_KEY` | **YES** | PEM format. In Coolify, check **"Is Multiline?"** — do NOT manually escape `\n` characters |
| `JWT_SECRET` | **YES** | HS256 fallback — same value as MBS Platform |

Do NOT present these as "either/or" or "optional". Both are required, period.

---

### Part 3: Dockerfile.backend — Three Gotchas (Every Module Hit These)

If your project is a TypeScript monorepo with npm workspaces (root `package.json` + `backend/package.json` + `frontend/package.json`), you WILL hit these three failures unless you handle them. This is not theoretical — StarMap, InnerQuest, LifeMap, Archetypes, and Rituals all hit all three in sequence.

**Gotcha 1 — `npm ci --prefix backend` fails (lockfile not found)**

`package-lock.json` lives at the monorepo root, not inside `backend/`. The `--prefix` flag looks for a lockfile inside `backend/` and fails. Use workspace install instead:
```dockerfile
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
RUN npm ci --workspace=backend
```

**Gotcha 2 — `tsc: not found` (Coolify strips devDependencies)**

Coolify injects `NODE_ENV=production` as a Docker build arg. This makes `npm install` / `npm ci` skip devDependencies — but `typescript` is a devDependency. The TypeScript compiler never gets installed. Override NODE_ENV in the **build stage only**:
```dockerfile
RUN NODE_ENV=development npm ci --workspace=backend
```

The runtime stage should still use `NODE_ENV=production`.

**Gotcha 3 — `Cannot find module dist/server.js` (wrong output path)**

If `tsconfig.json` has `rootDir: "."` (the default), TypeScript preserves folder structure in output: `src/server.ts` → `dist/src/server.js`, NOT `dist/server.js`. Either:
- **Option A** — Use the correct CMD: `CMD ["node", "backend/dist/src/server.js"]`
- **Option B** — Create `tsconfig.build.json` with `rootDir: "src"` so output goes to `dist/server.js`

**Complete Dockerfile.backend example (known working):**
```dockerfile
# ---- Build Stage ----
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
# Force development to install devDeps (typescript, etc.)
RUN NODE_ENV=development npm ci --workspace=backend
COPY backend/ ./backend/
RUN npx tsc -p backend/tsconfig.json

# ---- Runtime Stage ----
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
# Production deps only
RUN npm ci --workspace=backend --omit=dev
COPY --from=builder /app/backend/dist ./backend/dist
EXPOSE 4000
USER node
HEALTHCHECK --interval=30s --timeout=5s CMD wget -q --spider http://localhost:4000/health || exit 1
CMD ["node", "backend/dist/src/server.js"]
```

Adjust the CMD path based on your tsconfig's `rootDir` setting (see Gotcha 3).

---

### Part 4: Coolify Secrets — Don't Expose at Build Time

In Coolify, env vars can be marked "Available at Buildtime." **Do NOT check this for secrets.** Only `VITE_*` vars need to be build-time (they're baked into the frontend bundle).

| Variable | Build-time? | Why |
|----------|-------------|-----|
| `VITE_BACKEND_URL` | **Yes** (build arg) | Baked into frontend JS |
| `VITE_PLATFORM_URL` | **Yes** (build arg) | Baked into frontend JS |
| `VITE_PRODUCT_SLUG` | **Yes** (build arg) | Baked into frontend JS |
| `JWT_SECRET` | **No** (runtime only) | Secret — don't leak into Docker layer |
| `JWT_PUBLIC_KEY` | **No** (runtime only) | Key material — don't leak into Docker layer |
| `OPENAI_API_KEY` | **No** (runtime only) | Secret — don't leak into Docker layer |
| `MONGO_URL` | **No** (runtime only) | Secret — don't leak into Docker layer |
| All other backend vars | **No** (runtime only) | Backend reads them at startup |

If you see Docker warnings like `SecretsUsedInArgOrEnv: Do not use ARG or ENV instructions for sensitive data (ARG "JWT_SECRET")`, it means secrets are leaking into the build layer. Fix by unchecking "Available at Buildtime" in Coolify for those vars.

---

### Part 5: Environment Variables (Complete List)

**Backend env vars (Coolify → backend container, runtime only):**

| Variable | Value |
|----------|-------|
| `MONGO_URL` | (shared MongoDB connection string) |
| `DB_NAME` | `inner_lab` |
| `JWT_PUBLIC_KEY` | PEM format — check "Is Multiline?" in Coolify |
| `JWT_SECRET` | Same as MBS Platform |
| `PORT` | `4000` |
| `CORS_ORIGINS` | `https://{slug}.innerlab.ai,https://www.{slug}.innerlab.ai` |
| `PLATFORM_URL` | `https://api.magicbusstudios.com` |
| `PRODUCT_SLUG` | `{slug}` |
| `SERVICE_NAME` | `{slug}-api` |
| `LOG_LEVEL` | `info` |
| `OPENAI_API_KEY` | (shared MBS key — only if module uses AI) |
| `OPENAI_MODEL` | `gpt-4o-mini` (do NOT invent model names like gpt-5.4-mini) |

**Frontend build args (Coolify → frontend container, build-time only):**

| Variable | Value |
|----------|-------|
| `VITE_BACKEND_URL` | `https://api.{slug}.innerlab.ai` |
| `VITE_PLATFORM_URL` | `https://magicbusstudios.com` (NOT innerlab.ai — this is for billing/subscribe redirects) |
| `VITE_PRODUCT_SLUG` | `{slug}` |

Do NOT add invented env vars (e.g., `ALLOW_TRANSPORT_ONLY_ACCESS`). Only use the standard MBS env vars listed above. Do NOT include `VITE_GOOGLE_CLIENT_ID` — auth is delegated to innerlab.ai, your module never renders a Google sign-in button.

---

### Part 6: GDPR

Your module MUST implement `DELETE /api/user-data` (authenticated). It must delete:
1. **All `{prefix}_*` collection data** for the user
2. **All `il_*` entries** where `source_module: "{slug}"` and `user_id` matches

Do NOT delete il_* entries from other modules. Do NOT delete identity singletons (il_consciousness_profiles, il_personal_histories, il_user_wellness_profiles) — the MBS Platform cascade handles those.

---

### Part 7: Workflow — How to Proceed

Here's what I need you to do:

1. **Read your project's `platform-instructions/CLAUDE.md`** (if it exists) for the full module template with architecture context, database conventions, auth flow, and coding rules.

2. **Generate complete Coolify deployment instructions** applying all corrections from Parts 1-6 above. This should include:
   - Dockerfile.frontend (multi-stage: build → nginx)
   - Dockerfile.backend (multi-stage: build with tsc → runtime)
   - nginx.conf
   - Complete env var list for both containers
   - DNS setup
   - Pre-deploy checklist
   - Post-deploy verification steps

3. **Before presenting the instructions, self-check against these common mistakes** (every module so far made at least 3 of these):
   - [ ] All domains use `*.innerlab.ai`, NOT `*.magicbusstudios.com`
   - [ ] `VITE_PLATFORM_URL` = `magicbusstudios.com` (NOT innerlab.ai)
   - [ ] `PLATFORM_URL` = `api.magicbusstudios.com` (NOT innerlab.ai)
   - [ ] Both `JWT_PUBLIC_KEY` AND `JWT_SECRET` listed as required
   - [ ] JWT_PUBLIC_KEY says "Is Multiline?" not "escape \n"
   - [ ] CORS includes both www and non-www
   - [ ] OPENAI_MODEL = `gpt-4o-mini` (not invented)
   - [ ] No invented env vars
   - [ ] GDPR covers `{prefix}_*` AND `il_*` where source_module
   - [ ] Dockerfile uses `NODE_ENV=development` in builder stage
   - [ ] No secrets as build-time args (only VITE_* are build-time)
   - [ ] No "add module to MBS CORS" line (not needed)

4. **If anything is unclear**, respond with your questions instead of guessing. Your questions will be routed to the MBS Platform architecture agent, which is the source of truth for all deployment conventions. **Don't guess — ask.**

5. **If everything is clear**, present the complete Coolify deployment instructions. I will review them and send corrections if needed.
