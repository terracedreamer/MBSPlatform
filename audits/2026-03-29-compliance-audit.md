# MBS Ecosystem — 17 Project Compliance Audit Report

**Date**: 2026-03-29
**Audited by**: Claude Opus 4.6 (automated)
**Scope**: All 17 product folders in `~/Desktop/Codes/` (excluding Claude Setup and MCP Servers)
**Method**: Read each project's `FUTURE_WORK_TODO.md` compliance checklist, scanned actual source code to verify each item, updated checklists with findings, committed and pushed.

> **IMPORTANT — MBS vs MBSPlatform Clarification (added March 30, 2026):**
> MBSPlatform is a **docs-only repo** (zero server code). All MBS Platform server code (auth, billing, entitlements, GDPR cascade) lives in the **MBS** folder at `~/Desktop/Codes/MBS/server/`. The "MBSPlatform" section below audited the MBS server code but was mislabeled. When fixing items listed under "MBSPlatform," work in the **MBS** folder. The 155 console.log calls appear in both MBS and MBSPlatform sections — this is the same codebase counted once, not twice.

---

## Summary

| Metric | Value |
|--------|-------|
| **Total projects audited** | 17 |
| **Total checklist items compliant** | 126 |
| **Total checklist items remaining** | 78 |
| **Overall compliance rate** | 62% |
| **Cleanest project** | AI Tutor (6/7 compliant — only 1 gap) |
| **Most work needed** | YogaGhost, CWG, WildLens (7 items remaining each) |

---

## Per-Project Results

| Project | Stack | Compliant | Remaining | Branch | Compliance % |
|---------|-------|:---------:|:---------:|--------|:------------:|
| MBSPlatform | Express | 17 | 3 | main | 85% |
| MBS | Express | 7 | 6 | main | 54% |
| Brokenchain | Express | 9 | 5 | main | 64% |
| CWG | FastAPI | 8 | 7 | test | 53% |
| Consciousness | Express | 5 | 3 | main | 63% |
| Fakeartist | Express | 10 | 6 | main | 63% |
| Innerlab | Express | 9 | 4 | main | 69% |
| LazyChef | FastAPI | 4 | 3 | main | 57% |
| Mindhacker | Express | 9 | 4 | main | 69% |
| Movie Picker | Express | 10 | 4 | main | 71% |
| SmartCart | Express | 6 | 5 | main | 55% |
| TaskTracker | Express | 2 | 5 | main | 29% |
| Trivia Roast | Express | 4 | 5 | main | 44% |
| AI Tutor | FastAPI | 6 | 1 | main | 86% |
| Whispering House | FastAPI | 7 | 3 | main | 70% |
| WildLens | Next.js | 6 | 7 | main | 46% |
| YogaGhost | Express | 7 | 7 | main (dev) | 50% |

---

## Detailed Findings Per Project

### MBSPlatform (Layer 1 — SSO, Billing, Entitlements)
- **Compliant (17)**: CORS order, canonical env vars, response format, rate limiting, input validation, route patterns, Sonner toasts, no alert/prompt/confirm, correct fonts, safe array mapping, req.user shape in middleware
- **Remaining (3)**:
  1. **Winston Logger** — 155 `console.log/warn/error` calls across 15 server files. **CRITICAL**: `gdprCascade.js` imports `require("../utils/logger")` but the file `server/utils/logger.js` does not exist — this will crash at runtime when GDPR cascade is invoked.
  2. **Centralized Error Handler** — No `app.use((err, req, res, next))` middleware. No `sendSuccess`/`sendError`/`sendNotFound` helpers.
  3. **Auth Route Responses** — 7 auth route response bodies return `user: { id: user._id }` instead of `user: { userId: ... }`.

### MBS (Main Website + Platform Frontend)
- **Compliant (7)**: Sonner toasts, no alert/prompt/confirm, array safety, Google SSO above forms, req.user shape, response format, CORS order, rate limiting
- **Remaining (6)**:
  1. **Winston Logger** — 155 console calls across 15 files, no Winston installed
  2. **Centralized Error Handler** — Missing entirely
  3. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`
  4. **Input Validation** — Only `forms.js` uses express-validator
  5. **Test Suite** — None exists
  6. **Chrome Verification** — Pending

### Brokenchain (Arcade Game)
- **Compliant (9)**: req.user shape (`userId`), response format, Winston logger (zero console.log), Sonner toasts, CORS before helmet, rate limiting, GDPR endpoint, input validation (most routes), array safety (most)
- **Remaining (5)**:
  1. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`; routes use inline `res.json()`
  2. **Input Validation** — Missing on 3 routes: `PUT /api/users/me`, `POST /api/users/friends/add`, `POST /api/upload/audio`
  3. **Array Safety** — Unguarded `game.highlights.map()` in `client/src/pages/RevealPage.tsx:160`
  4. **Test Suite** — No Jest/supertest files
  5. **Chrome Verification** — Pending redeploy

### CWG (Conversations with God — FastAPI)
- **Compliant (8)**: Timezone-aware datetimes, env vars (with fallbacks), auth dependency injection, `get_database()` sync, DB fields snake_case, rate limiting, response format (majority)
- **Remaining (7)**:
  1. **GDPR Endpoint** — `DELETE /api/user-data` missing entirely. Admin delete logic exists in `admin_routes.py` but not exposed as required route.
  2. **Frontend Dialogs** — 12 `confirm()`/`prompt()` calls need modal replacement
  3. **Pydantic Config** — Deprecated `class Config` in 3 models (should be `model_config = SettingsConfigDict(...)`)
  4. **Production Logging** — 7 `print()` calls in `share_routes.py`, `journal_insights_routes.py`, `wisdom_book_routes.py`, `roundtable_routes.py`
  5. **Input Validation** — Partial coverage
  6. **Array Safety** — Pattern rarely used
  7. **Test Suite** — Needs httpx AsyncClient modernization

### Consciousness (Teaser Sites — Monorepo)
- **Compliant (5)**: Winston logger (zero console.log), response format, error handler, input validation, rate limiting
- **Remaining (3)**:
  1. **Sonner Toasts** — Installed but never wired up. `<Toaster>` not rendered, `toast()` never called. EmailCapture uses inline state UI.
  2. **Test Suite** — None exists
  3. **Chrome Verification** — Pending

### Fakeartist (Arcade Game)
- **Compliant (10)**: Winston logger (zero console.log), Sonner toasts, response format, rate limiting, AI service layer, CORS before helmet, GDPR endpoint, req.user shape (`userId`), safe array mapping, correct branch
- **Remaining (6)**:
  1. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`
  2. **Error Handler** — No centralized error-handling middleware
  3. **Input Validation** — `express-validator` installed but never imported (dead dependency)
  4. **Test Suite** — None exists
  5. **Env Vars** — Non-canonical: `MONGODB_URI` (internal), `CLIENT_URL` instead of `CORS_ORIGINS`
  6. **Fonts** — Uses Inter instead of Space Grotesk/DM Sans

### Innerlab (Layer 2 — Middleware)
- **Compliant (9)**: Winston logger (zero console.log), response format, req.user.userId, CORS before helmet, rate limiting, input validation, Sonner toasts, no alert/prompt/confirm, array safety
- **Remaining (4)**:
  1. **GDPR Endpoint** — No `DELETE /api/user-data` exists
  2. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound` abstraction
  3. **Test Suite** — None exists
  4. **Chrome Verification** — Pending

### LazyChef (FastAPI — Gold Standard for Testing)
- **Compliant (4)**: Timezone-aware datetimes, `SettingsConfigDict` config, rate limiting, auth pages/Google SSO placement
- **Remaining (3)**:
  1. **Input Validation** — 1 `PATCH` endpoint accepts raw `dict` instead of Pydantic model (`server.py:932`)
  2. **Array Safety** — 3 unguarded `.map()` calls (`RecipePage.jsx`, `RecipeDetailPage.jsx`)
  3. **Frontend Dialog** — 1 `prompt()` call in `BarcodeScanner.jsx` (should be modal input)
- **Skipped**: Sonner migration (LOW PRIORITY)

### Mindhacker (Arcade — Layer 3)
- **Compliant (9)**: GDPR endpoint, Winston logger, Sonner toasts, req.user shape, response format, input validation, rate limiting, CORS order, deployment quirk
- **Remaining (4)**:
  1. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`; routes use inline `res.json()`
  2. **Error Handler** — No centralized middleware; each route has own try/catch
  3. **Test Suite** — None exists
  4. **Chrome Verification** — Pending
- **Note**: Helmet is not installed (should be added)

### Movie Picker (Single Container)
- **Compliant (10)**: GDPR endpoint, Winston logger, Sonner toasts, response format, error handler, input validation, rate limiting, CORS order, array safety
- **Remaining (4)**:
  1. **req.user Shape** — `req.user._id` used throughout routes (`movies.js`, `watchlists.js`, `users.js`, `resolveUser.js`) instead of `req.user.userId`
  2. **Response Helpers** — Missing `sendNotFound`; routes inline the 404 pattern
  3. **Test Suite** — None exists
  4. **Chrome Verification** — Pending

### SmartCart (Single Container)
- **Compliant (6)**: Response format, error handler, input validation, rate limiting, CORS order, auth pages (N/A)
- **Remaining (5)**:
  1. **req.user Shape** — All routes use `req.user.id` instead of `req.user.userId`
  2. **Winston Logger** — Custom `console.log` wrapper, not actual Winston
  3. **Sonner Toasts** — Uses `react-hot-toast` across 11 files
  4. **Array Safety** — Insufficient guards
  5. **Test Suite** — None exists

### TaskTracker (Nixpacks)
- **Compliant (2)**: GDPR endpoint, Winston logger
- **Remaining (5)**:
  1. **CORS Order** — `helmet()` applied before `cors()` in `server/index.js`
  2. **req.user Shape** — `parentJoin.js` uses `req.user.id` in 6 places; ~40 responses lack `success` field
  3. **Response Helpers** — `sendSuccess`/`sendError`/`sendNotFound` exist in `responseHelpers.js` but are unused by any of the 15 route files
  4. **Sonner Toasts** — Custom toast system instead of Sonner
  5. **Array Safety** — Most `.map()` calls lack `(array || []).map()` guards
- **Note**: Uses `REACT_APP_*` (known legacy exception for CRA)

### Trivia Roast (Arcade Game)
- **Compliant (4)**: Winston logger (zero console.log), response format, input validation, rate limiting
- **Remaining (5)**:
  1. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`
  2. **Error Handler** — No centralized middleware
  3. **AI Service Layer** — OpenAI call inline in route handler, not behind service layer
  4. **Test Suite** — None exists
  5. **Chrome Verification** — Pending

### AI Tutor (FastAPI)
- **Compliant (6)**: Sonner toasts/no alerts, timezone-aware datetimes, `SettingsConfigDict` config, rate limiting, auth pages/SSO, array safety
- **Remaining (1)**:
  1. **Input Validation** — 7 endpoints accept `body: dict` instead of typed Pydantic `BaseModel` subclasses: `create_learning_path`, `update_learning_path_progress`, `update_chat_session`, `add_friend`, `submit_weekly_challenge`, `create_room`, `send_room_message`

### Whispering House (FastAPI)
- **Compliant (7)**: GDPR endpoint, Sonner toasts, timezone-aware datetimes, `SettingsConfigDict` config, input validation, auth pages/SSO, array safety
- **Remaining (3)**:
  1. **Rate Limiting** — `slowapi` installed as dependency but zero configuration; no `Limiter` instance or decorators anywhere
  2. **Test Suite** — No `tests/` directory exists
  3. **Chrome Verification** — Pending
- **Note**: CORS is hardcoded to `frontend_url` + localhost instead of using `CORS_ORIGINS` env var pattern

### WildLens (Next.js)
- **Compliant (6)**: GDPR endpoint (thorough with file cleanup), Winston logger (zero console.log), response format via helpers, centralized error handler, rate limiting, auth pages (N/A)
- **Remaining (7)**:
  1. **Sonner Migration** — Custom `ToastContext` used across 23 files
  2. **req.user Shape** — Auth middleware exposes both `id` and `userId`, but 95 usages across 12 route files use `req.user.id`
  3. **Input Validation** — `express-validator` only on 3 of 13 route files
  4. **CORS Order** — `helmet()` registered before `cors()` in `index.js`
  5. **Array Safety** — Only 12 guarded `.map()` out of 124 total
  6. **Test Suite** — None exists
  7. **Chrome Verification** — Pending

### YogaGhost / FlowState (Express)
- **Compliant (7)**: Winston logger, input validation, rate limiting, CORS order, auth pages (N/A — uses Inner Lab SSO), AI service layer, no alert/prompt/confirm
- **Remaining (7)**:
  1. **GDPR Endpoint** — `DELETE /api/user-data` missing entirely
  2. **req.user Shape** — Uses flat `req.userId` instead of `req.user = { userId, email, name, avatar }`
  3. **Response Format** — 47 error responses use `{ error: '...' }` not `{ success: false, message: '...' }`
  4. **Response Helpers** — No `sendSuccess`/`sendError`/`sendNotFound`
  5. **Error Handler** — No centralized middleware
  6. **Array Safety** — Only 2 of 160 `.map()` calls guarded
  7. **Test Suite** — Zero test files
- **Note**: Sonner marked N/A — YogaGhost uses custom CSS Modules toast system (valid thematic exception)

---

## Top Systemic Issues (Ranked by Frequency)

| # | Issue | Projects Affected | Count |
|---|-------|-------------------|:-----:|
| 1 | No test suite | All except LazyChef, CWG (partial), AI Tutor | 14/17 |
| 2 | No response helpers (`sendSuccess`/`sendError`/`sendNotFound`) | Brokenchain, CWG, Fakeartist, Innerlab, Mindhacker, Movie, MBS, MBSPlatform, SmartCart, Trivia, YogaGhost, TaskTracker (has but unused) | 12/17 |
| 3 | No centralized error handler middleware | Brokenchain, Fakeartist, Mindhacker, MBS, MBSPlatform, Trivia, YogaGhost, TaskTracker, SmartCart, CWG | 10/17 |
| 4 | `req.user.id`/`_id` instead of `userId` | TaskTracker, SmartCart, Movie, WildLens, MBSPlatform (auth routes), YogaGhost | 6/17 |
| 5 | Missing GDPR `DELETE /api/user-data` | CWG, Innerlab, YogaGhost, Whispering House | 4/17 |
| 6 | Winston logging absent | MBS, MBSPlatform, SmartCart | 3/17 |
| 7 | Helmet before CORS | TaskTracker, WildLens | 2/17 |
| 8 | Non-Sonner toast library | SmartCart (react-hot-toast), TaskTracker (custom), WildLens (custom) | 3/17 |

---

## Critical / Urgent Items

### P0 — Runtime Crash Risk
- **MBSPlatform**: `server/services/gdprCascade.js` imports `require("../utils/logger")` but `server/utils/logger.js` does not exist. This **will crash** when any user triggers account deletion through the GDPR cascade flow.

### P1 — GDPR Compliance Broken
- **CWG, Innerlab, YogaGhost, Whispering House**: Missing `DELETE /api/user-data` endpoint means the MBS Platform cascade deletion cannot reach these apps. Users who delete their account will have orphaned data in these four databases.

### P2 — Security
- **TaskTracker, WildLens**: `helmet()` is registered before `cors()`, which can cause CORS headers to be stripped or misconfigured.

---

## Recommendations (Priority Order)

1. **Fix MBSPlatform logger crash** — Create `server/utils/logger.js` with Winston, or change the import. This is a production crash waiting to happen.
2. **Add GDPR endpoints to CWG, Innerlab, YogaGhost, Whispering House** — Required for legal compliance and cascade deletion.
3. **Standardize `req.user.userId`** across TaskTracker, SmartCart, Movie, WildLens, MBSPlatform auth routes, YogaGhost — 6 projects use inconsistent naming.
4. **Create a shared response helpers template** — Copy from Movie Picker or Brokenchain (which have partial implementations) and standardize across all Express projects.
5. **Add Winston to MBS, MBSPlatform, SmartCart** — The 3 projects still using console.log.
6. **Fix CORS/helmet ordering** in TaskTracker and WildLens.
7. **Migrate SmartCart from react-hot-toast to Sonner** — 11 files affected.
8. **Build test suites** — Start with MBSPlatform (most critical) and expand using LazyChef as the template.

---

*Report generated 2026-03-29. Each project's `FUTURE_WORK_TODO.md` has been updated with detailed per-item findings and committed to its respective branch.*
