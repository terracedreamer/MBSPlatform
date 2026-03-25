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
- CWG database: `conversations_with_god` with 56 collections
- ~10 real users (friends testing)

## Target State
- Auth handled by MBS Platform at magicbusstudios.com (Google SSO + Nostr + LNURL)
- Billing handled by MBS Platform (Stripe + BTCPay)
- User identity stored in `mbs_platform` database
- CWG product data stored in `inner_lab` database with `cwg_` prefix
- Shared Inner Lab data stored in `inner_lab` database with `il_` prefix
- Old `conversations_with_god` database stays untouched as backup

## Migration Steps

### Step 1: Update Auth Flow
- Remove CWG's standalone auth (login, signup, password reset, TOTP)
- Add JWT verification middleware that validates tokens from MBS Platform
- Login button redirects to: `magicbusstudios.com/auth/login?redirect=conversationswithgod.ai&brand=innerlab`
- After login, MBS Platform redirects back with `?token={JWT}`
- Store JWT, send in Authorization header on all API calls

### Step 2: Remove Billing
- Remove CWG's Stripe integration (checkout, webhooks, portal)
- Remove CWG's BTCPay integration
- Upgrade/billing buttons redirect to MBS Platform billing page
- Check access via MBS Platform: `GET magicbusstudios.com/api/entitlements/cwg`
- Free tier: `{ hasAccess: true, reason: "free_tier" }`
- Premium: `{ hasAccess: true, reason: "product_pass" }` or `{ hasAccess: true, reason: "category_access" }`

### Step 3: Migrate Data
Run migration script (built in MBS/ project) that:

**Copies to `mbs_platform.users`:**
- email, name, picture, google_id, nostr_npub, lnurl_linking_key, auth_provider
- consent_preferences, is_admin, referral_code, referred_by, referral_count
- created_at, updated_at, last_login

**Copies to `inner_lab` with `cwg_` prefix (39 collections):**
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
- active_sessions, lnurl_challenges, nostr_challenges, nostr_events
- consent_audit_log, data_requests, user_encryption, encrypted_backups
- Friends and invites data (if any)
- Push subscriptions, feature flags, promo codes

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

## What NOT to Do
- Do NOT delete the `conversations_with_god` database — it's the backup
- Do NOT modify user data during migration — copy only
- Do NOT run migration until MBS Platform and Inner Lab Middleware are built and tested
- Do NOT keep standalone auth/billing code — remove it completely after migration
