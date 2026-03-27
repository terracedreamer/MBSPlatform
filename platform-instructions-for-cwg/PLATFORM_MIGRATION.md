# CWG — MBS Platform Migration Instructions

## Prerequisites — Do NOT Start Until These Are Built
1. **MBS Platform (Layer 1)** at magicbusstudios.com must be live with SSO + entitlements API
2. **Inner Lab Middleware (Layer 2)** at innerlab.ai must be live with il_* collections created
3. Both must be tested and working before ANY CWG migration begins

## What's Happening
CWG (Conversations With God) is being migrated to use the centralized MBS Platform for auth/billing and the Inner Lab shared database for data. This document tells the CWG agent what to do.

## Important Technical Notes
- **CWG is Python (FastAPI + Motor)** — JWT validation middleware must be written in Python, not JavaScript
- **CWG uses UUID strings for _id** (not MongoDB ObjectId) — migration must map CWG UUIDs to platform ObjectIds. Store a `platform_user_id` field in CWG user profiles linking to the platform user.
- **JWT_SECRET must match the MBS Platform** — use the same secret so JWTs issued by the platform validate in CWG
- **Migration scripts run FROM the MBS/ project** (`MBS/server/scripts/`), not from CWG. The CWG agent should NOT build its own migration script — just prepare the backend for the new database structure.

## Current State
- CWG has its own auth (Google SSO + email/password + Nostr + LNURL)
- CWG has its own Stripe integration (test keys)
- CWG has its own BTCPay integration (experimental)
- CWG database: `conversations_with_god` with 28 collections (originally estimated 56 — actual count confirmed during migration)
- ~10 real users (friends testing)

## Target State
- Auth handled by MBS Platform at magicbusstudios.com (Google SSO + Email/Password + Nostr + LNURL + 2FA/TOTP)
- All ~10 CWG users automatically become MBS Platform users (including their password hashes, so email/password login keeps working)
- Billing handled by MBS Platform (Stripe + BTCPay)
- User identity stored in `mbs_platform` database
- CWG product data stored in `inner_lab` database with `cwg_` prefix
- Shared Inner Lab data stored in `inner_lab` database with `il_` prefix
- Old `conversations_with_god` database stays untouched as backup

## Migration Steps

### Step 1: Update Auth Flow
- Remove CWG's standalone auth (login, signup, password reset, TOTP)
- Add JWT verification middleware that validates tokens from MBS Platform
- Login button redirects to: `https://innerlab.ai/auth/login?redirect=https://conversationswithgod.ai`
  - CWG is an Inner Lab module, so login goes through the Inner Lab login page (which calls MBS Platform auth APIs)
  - Alternative: CWG can build its OWN login page following the same pattern (calls MBS Platform APIs directly) — but using Inner Lab's is simpler
- After login, user is redirected back to CWG with `?token={JWT}`
- Store JWT, send in Authorization header on all API calls
- IMPORTANT: After extracting `?token={JWT}` from URL, immediately call `history.replaceState(null, '', window.location.pathname + window.location.hash)` to remove the token from the URL (prevents token leakage via browser history and referrer headers)

### Step 2: Remove Billing
- Remove CWG's Stripe integration (checkout, webhooks, portal)
- Remove CWG's BTCPay integration
- Upgrade/billing buttons redirect to MBS Platform billing page
- Check access via MBS Platform: `GET https://magicbusstudios.com/api/entitlements/cwg`
- Free tier: `{ hasAccess: true, reason: "free_tier" }`
- Premium: `{ hasAccess: true, reason: "product_pass" }` or `{ hasAccess: true, reason: "category_access" }`

### Step 3: Migrate Data
Run migration script (built in MBS/ project) that:

**Copies to `mbs_platform.users` (ALL ~10 CWG users become platform users):**
- email, name, picture → **rename to `avatar`** (platform model uses `avatar`, not `picture`)
- **password** (hashed) → copy to `password_hash` field. CWG uses bcrypt — platform also uses bcrypt, so hashes are compatible. No re-hashing needed.
- google_id, nostr_npub, lnurl_linking_key
- **auth_methods**: build array from what exists — e.g. if user has password + google_id → `["email", "google"]`, if user has password + nostr_npub → `["email", "nostr"]`, etc.
- **email_verified**: copy from CWG's `email_verified` field (default `true` for Google SSO users)
- **totp_enabled, totp_secret, totp_backup_codes**: copy from CWG user if they have 2FA enabled
- consent_preferences, is_admin, referral_code, referred_by, referral_count
- preferred_language (set to `"en"` if not present), preferences (set to `{}` if not present)
- stripe_customer_id (from CWG's Stripe integration, if exists)
- created_at, updated_at, last_login

**Field normalization rules:**
- CWG `picture` → platform `avatar`
- All field names must be snake_case in mbs_platform (CWG already uses snake_case ✓)
- Fields not present in CWG data get sensible defaults: `preferred_language: "en"`, `preferences: {}`, `referral_count: 0`

**Copies to `inner_lab` with `cwg_` prefix (39 product collections + 3 infrastructure collections = 42 total):**
- cwg_chat_sessions, cwg_messages, cwg_journal_entries, cwg_journal_insights
- cwg_practices, cwg_meditation_completions, cwg_user_meditations, cwg_meditation_sessions
- cwg_program_enrollments, cwg_user_challenges, cwg_badge_history
- cwg_daily_wisdom, cwg_daily_quotes_history, cwg_wisdom_subscribers, cwg_reading_list, cwg_sacred_text_chunks
- cwg_blog_posts, cwg_blog_comments, cwg_community_reflections, cwg_community_comments
- cwg_bookmarks, cwg_favorites, cwg_saved_wisdom
- cwg_shared_quotes, cwg_shared_consciousness, cwg_shared_milestones
- cwg_btcpay_invoices, cwg_btcpay_tips, cwg_btcpay_donations, cwg_btcpay_chat_sessions
- cwg_trial_sessions, cwg_wisdom_certificates
- cwg_email_campaigns, cwg_feedback, cwg_flagged_content, cwg_client_errors
- cwg_roundtable_sessions, cwg_shared_conversations, cwg_conversation_summaries

**Copies to `inner_lab` with `il_` prefix (shared data):**
- il_consciousness_profiles (from consciousness_profile + consciousness_profile_structured)
- il_consciousness_snapshots
- il_personal_histories (from personal_history + personal_history_structured — normalize dual formats)
- il_check_ins (from checkins + daily_checkins)
- il_user_memories (from user_memories — with `source_module: "cwg"`, `shared: false` by default)
- il_analytics_events
- il_notifications

**Moves to `mbs_platform` (platform-level):**
- active_sessions → `ActiveSession` model
- lnurl_challenges → `LnurlChallenge` model
- nostr_challenges → `NostrChallenge` model
- consent_audit_log → `ConsentAuditLog` model
- data_requests → `DataRequest` model
- Friends and invites data (if any) → `Friend` and `Invite` models
- Push subscriptions → `PushSubscription` model
- Feature flags → `FeatureFlag` model
- Promo codes → `Promotion` model

**CWG-specific collections that stay as cwg_* (not platform-level):**
- nostr_events → `cwg_nostr_events` (CWG-specific Nostr event log, not a platform concern)
- user_encryption → `cwg_user_encryption` (CWG's client-side encryption keys — platform will build its own encryption infrastructure)
- encrypted_backups → `cwg_encrypted_backups` (CWG-specific backup data)

### Step 4: Update CWG Backend
- Change database connection from `conversations_with_god` to `inner_lab`
- Update all collection references to use `cwg_` prefix
- Read shared data from `il_*` collections
- Use `platformUserId` (from JWT) instead of local user IDs
- Remove auth routes, Stripe routes, BTCPay routes

### Step 5: Update CWG Frontend
- Remove login/signup pages — redirect to MBS Platform
- Remove billing/pricing pages — redirect to MBS Platform
- Remove account deletion flow — redirect to MBS Platform
- Store MBS Platform JWT instead of local token
- Update API calls to use new collection structure

## Dual-Format Fields to Normalize
CWG has legacy + structured formats for two fields:
- `personal_history` (string) + `personal_history_structured` (object with 26 fields) → normalize into `il_personal_histories`
- `consciousness_profile` (Q&A object) + `consciousness_profile_structured` (assessment) → normalize into `il_consciousness_profiles`

Migration script must handle users who have one format, the other, or both.

## User Memory Privacy
When migrating `user_memories` to `il_user_memories`:
- Set `source_module: "cwg"` on every memory
- Set `shared: false` on every memory (private by default)
- User must explicitly opt-in to share memories across Inner Lab modules

## Cutover Approach

~10 friends testing. Data loss is acceptable if it happens. Keep it simple:

1. Run migration script — copies data from `conversations_with_god` to `mbs_platform` + `inner_lab`
2. Deploy new CWG (new DB, JWT auth, no standalone auth)
3. Old tokens break — users re-login through MBS Platform. That's fine.
4. Old database stays untouched as a backup. Don't delete it.

## User Dedup / Merge Logic
The migration script MUST use **upsert on email** when creating `mbs_platform.users` records. If a user already exists (e.g., they were migrated from another product first), MERGE fields — do not overwrite. CWG data takes priority for fields like `nostr_npub` and `lnurl_linking_key` that FlowState doesn't have.

## What NOT to Do
- Do NOT delete the `conversations_with_god` database — it's the backup
- Do NOT modify user data during migration — copy only
- Do NOT run migration until MBS Platform and Inner Lab Middleware are built and tested
- Do NOT keep standalone auth/billing code — remove it completely after migration

---

## Completion Report (REQUIRED)

When you finish the migration and refactor, generate a file called `PHASE_3_REPORT.md` in the project root. The orchestrator will fetch it from here. The report must contain:

1. **What was built/changed** — Every file created, modified, or deleted, grouped by backend/frontend
2. **What changed from the plan** — Any deviations from this document. Why?
3. **Migration results** — How many users migrated, collections copied, any errors or skipped records
4. **Collection mapping** — Old collection name → new collection name (cwg_* and il_*) as actually created
5. **Field renames** — Every field that was renamed during migration (old → new)
6. **Env vars required** — Complete list for the refactored CWG app
7. **Code removed** — List of removed auth/billing routes, pages, and files
8. **JWT integration** — How JWT middleware was implemented (Python library used, header format, fields extracted)
9. **Assumptions made** — Anything you had to decide that wasn't explicitly in the spec
10. **Known gaps** — Anything deferred or issues discovered
11. **Gotchas for the orchestrator** — Anything that affects other phases or the platform architecture
12. **Testing steps** — How to verify the migration worked and the refactored app functions correctly

This report is critical — the orchestrator session (MBSPlatform repo) uses it to track the migration and update downstream instructions if needed.

---

## Phase 1 Learnings (Added by Orchestrator — 2026-03-26)

These are real-world implementation details from the MBS Platform build that affect this migration:

### Env Var Name
- The MBS Platform uses **`MONGO_URL`** (not `MONGODB_URI`). Use `MONGO_URL` for consistency.

### JWT Details (as actually implemented)
- Header: `Authorization: Bearer <token>`
- Payload: `{ userId, email, name, avatar, isAdmin, iat, exp }`
- `userId` is a **string** (ObjectId.toString()). CWG uses UUID strings — the migration maps CWG UUIDs to platform ObjectId strings. In the refactored CWG backend, use `req.user.userId` from the JWT.
- Python JWT library: use `PyJWT` (`import jwt`), algorithm `HS256`, verify with the shared `JWT_SECRET`.

### Frontend Token Storage
- After login redirect, the JWT arrives as `?token=<JWT>` in the URL
- CWG frontend must: extract token, store as `localStorage.setItem("mbs_token", token)`, store user as `localStorage.setItem("mbs_user", JSON.stringify(user))`, then `history.replaceState` to remove from URL
- All API calls use header: `Authorization: Bearer ${localStorage.getItem("mbs_token")}`

### First Platform User (Your Test Account)
- `1984.abhinav@gmail.com` → platform ID: `69c53401fe8f1763b9046ae5`
- This user exists in `mbs_platform` but has NO link to CWG data yet
- During migration, match on `email` or `google_id` to link existing CWG users to platform accounts

### Category Values
- Categories are lowercase no-separator: `innerlab` (not `inner_lab`)

### Entitlement Check
- `GET https://magicbusstudios.com/api/entitlements/cwg` with `Authorization: Bearer <JWT>`
- Returns `{ success, hasAccess, reason }` — reason `free_tier` means free access, `no_subscription` means blocked
- Cache response for 5 minutes in-memory

### GDPR
- MBS Platform cascade delete reaches into `inner_lab` database. All cwg_* and il_* collections must use `user_id` field consistently.

### BTCPay
- BTCPay entitlements are 30-day non-recurring. CWG's existing BTCPay integration should be REMOVED — all payments go through the platform.

---

## Phase 1 Learnings (SUPPLEMENT — confirmed from live MBS Platform)

### JWT Format (use this for Python middleware)
```python
# JWT payload from MBS Platform:
# { "userId": "ObjectId string", "email": "..." or null, "name": "...", "avatar": "..." or null, "isAdmin": false }
# Verify: jwt.decode(token, JWT_SECRET, algorithms=["HS256"])
# Access user ID: payload["userId"]  (camelCase, string)
```

### CRITICAL: email can be null
Nostr/LNURL users have no email. CWG code that assumes `user.email` exists will break. Guard with `if user.get("email"):` everywhere.

### Token extraction pattern (frontend)
```javascript
// After redirect back from MBS Platform login:
const token = new URLSearchParams(window.location.search).get("token");
if (token) {
  localStorage.setItem("mbs_token", token);
  window.history.replaceState(null, '', window.location.pathname);
}
```

### Stripe price IDs for CWG
CWG's existing Stripe price IDs (`STRIPE_MONTHLY_PRICE_ID`, `STRIPE_ANNUAL_PRICE_ID`) are already configured in the MBS Platform backend. When CWG removes its Stripe integration, users who click "Upgrade" get redirected to `magicbusstudios.com/billing` which handles checkout using those same price IDs.

### Entitlement reason values (complete list)
`"product_pass"`, `"category_access"`, `"mbs_all_access"`, `"free_tier"`, `"no_subscription"` — confirmed from live MBS Platform code.

### Rate limiting
MBS Platform rate limits: 100 req/15min. Cache entitlement checks for 5 min to avoid hitting the limit.

### First platform user
Abhinav Gupta (`1984.abhinav@gmail.com`), platform user ID: `69c53401fe8f1763b9046ae5`. Use this for testing migration — this user exists in `mbs_platform.users` already.
