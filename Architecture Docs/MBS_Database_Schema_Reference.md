# MBS Ecosystem — Database Schema Reference

**Version**: 1.0
**Date**: April 2, 2026
**Purpose**: Complete MongoDB schema reference for the MBS Platform ecosystem. Use this to understand data structures when building new modules or integrating with existing ones.

---

## Database Map

All applications share a single MongoDB cluster (self-hosted). Each concern gets its own database.

| Database | Owner | Connection |
|----------|-------|-----------|
| `mbs_platform` | MBS Platform (Layer 1) | Only MBS Platform backend connects |
| `inner_lab` | Inner Lab Middleware (Layer 2) + all IL modules | Multiple backends share this DB |
| `brokenchain` | BrokenChain | Own backend only |
| `mindhacker` | MindHacker | Own backend only |
| `triviaroast` | Trivia Roast | Own backend only |
| `fakeartist` | Fake Artist | Own backend only |
| `whispering_house` | Whispering House | Own backend only |
| `wildlens` | WildLens | Own backend only |
| `lazy_chef` | Lazy Chef | Own backend only |
| `moviepicker` | Movie Picker | Own backend only |
| `smartcart` | SmartCart | Own backend only |
| `tasktracker` | TaskTracker | Own backend only |
| `ai_tutor` | AI Tutor | Own backend only |

---

## 1. mbs_platform Database (Layer 1 — MBS Platform)

### User
**Collection**: `users`
**One document per registered user across the entire ecosystem.**

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `_id` | ObjectId | auto | | This is the canonical user ID across ALL apps |
| `email` | String | no | unique, sparse, lowercase, trim | Null for Nostr/LNURL pseudonymous users |
| `name` | String | yes | trim | |
| `avatar` | String | no | default null | URL to avatar image |
| `google_id` | String | no | unique, sparse | Google OAuth subject ID |
| `nostr_npub` | String | no | unique, sparse | Nostr public key |
| `lnurl_linking_key` | String | no | unique, sparse | LNURL-Auth linking key |
| `auth_methods` | [String] | no | default [] | e.g., `["email", "google", "nostr", "lnurl"]` |
| `password_hash` | String | no | default null | bcrypt hash |
| `email_verified` | Boolean | no | default false | |
| `email_verification_token` | String | no | default null | |
| `email_verification_expires` | Date | no | default null | |
| `password_reset_token` | String | no | default null | |
| `password_reset_expires` | Date | no | default null | |
| `totp_enabled` | Boolean | no | default false | 2FA enabled flag |
| `totp_secret` | String | no | default null | Encrypted TOTP secret |
| `totp_backup_codes` | [String] | no | default [] | bcrypt-hashed backup codes |
| `preferred_language` | String | no | default "en" | |
| `preferences` | Mixed | no | default {} | User preferences blob |
| `consent_preferences` | Mixed | no | default {} | GDPR consent settings |
| `is_admin` | Boolean | no | default false | Also checked via ADMIN_EMAILS env var |
| `stripe_customer_id` | String | no | unique, sparse | Stripe customer reference |
| `referral_code` | String | no | unique, sparse | User's referral code |
| `referred_by` | ObjectId | no | ref User | Who referred this user |
| `referral_count` | Number | no | default 0 | |
| `last_login` | Date | no | default Date.now | |
| `created_at` | Date | auto | Mongoose timestamps | |
| `updated_at` | Date | auto | Mongoose timestamps | |

**Critical**: The `_id` of a User document becomes the `userId` in JWT tokens and the `user_id` in all child app databases. This is the universal identity key.

---

### Entitlement
**Collection**: `entitlements`
**Tracks what products/categories a user has access to.**

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `user_id` | ObjectId | yes | ref User, indexed | |
| `category` | String | yes | enum: `innerlab`, `arcade`, `studioworks` | |
| `type` | String | yes | enum: `product_pass`, `category_access`, `mbs_all_access` | |
| `product` | String | no | default null | Product slug (null for category/all-access) |
| `status` | String | no | enum: `active`, `expired`, `cancelled`, `trial`; default `active` | |
| `stripe_subscription_id` | String | no | default null, indexed | |
| `purchased_at` | Date | no | default Date.now | |
| `expires_at` | Date | no | default null | |
| `trial_ends_at` | Date | no | default null | |

**Indexes**: `{user_id: 1, product: 1}`, `{user_id: 1, category: 1}`, `{stripe_subscription_id: 1}`

**How entitlements are checked**: `GET /api/entitlements/{product_slug}` returns `{ success, hasAccess, reason }` where reason is one of: `mbs_all_access`, `category_access`, `product_pass`, `free_tier`, `no_subscription`.

---

### Transaction
**Collection**: `transactions`

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `user_id` | ObjectId | yes | ref User, indexed |
| `type` | String | yes | enum: `stripe_subscription`, `stripe_one_time`, `btcpay_lightning` |
| `category` | String | no | `innerlab`, `arcade`, `studioworks` |
| `product` | String | no | Product slug |
| `amount` | Number | yes | In cents (USD) or sats (BTC) |
| `currency` | String | yes | `usd` or `btc` |
| `stripe_payment_id` | String | no | |
| `description` | String | no | |
| `promo_code` | String | no | If promotion was applied |
| `created_at` | Date | auto | |

---

### Other mbs_platform Collections

| Collection | Purpose | Key Fields |
|-----------|---------|-----------|
| `activesessions` | Active user sessions | `user_id`, `session_token` (unique), `ip_address` (hashed), `user_agent`, `last_active` |
| `activitylogs` | User action audit trail | `user_id`, `product`, `category`, `action`, `metadata` |
| `announcements` | System announcements | `title`, `message`, `type` (info/warning/promo/maintenance), `target` (all/innerlab/arcade/studioworks), `active`, `valid_from`, `valid_until` |
| `btcpayinvoices` | BTCPay Lightning invoices | `user_id`, `invoice_id` (unique), `amount_sats`, `status` (pending/settled/expired/invalid) |
| `consentauditlogs` | GDPR consent audit (IMMUTABLE) | `user_id` (String, hashed — NOT ObjectId), `consent_type`, `action`, `version`. **Never deleted — anonymized on account deletion.** |
| `datarequests` | GDPR data requests | `user_id`, `request_type` (access/deletion/portability/rectification), `status` |
| `emailpreferences` | Email notification prefs | `user_id` (unique), `marketing_emails`, `product_updates`, `transactional_emails`, `unsubscribe_token` (sparse unique) |
| `featureflags` | Feature availability | `feature_name` (unique), `enabled_for_plans`, `enabled_for_users`, `description` |
| `friends` | Friend relationships | `users` (array of exactly 2 ObjectIds), `status` (pending/accepted/blocked) |
| `invites` | Friend/product invites | `user_id`, `invite_code` (unique), `email`, `claimed_by_user_id`, `expires_at` |
| `lnurlchallenges` | LNURL-Auth challenges | `challenge_id` (unique), `challenge_value`, `status`. **TTL index: auto-expires.** |
| `nostrchallenges` | Nostr auth challenges | `challenge_id` (unique), `challenge_value`, `status`. **TTL index: auto-expires.** |
| `promotions` | Promo/discount codes | `code` (unique), `discount_type` (percentage/fixed), `discount_value`, `max_uses`, `current_uses`, `valid_from`, `valid_until` |
| `pushsubscriptions` | Web push subscriptions | `user_id`, `endpoint` (unique), `auth_key`, `p256dh_key` |
| `referrals` | Referral tracking | `referrer_user_id`, `referred_user_id`, `status`, `reward` |

---

## 2. inner_lab Database (Layer 2 — Inner Lab Middleware + Modules)

All Inner Lab module backends connect to this same database. Collections are namespaced by prefix. **All documents use `user_id` as a String (not ObjectId)** — the value is `userId` from the JWT (`User._id.toString()`).

### Shared Collections (il_* — owned by Inner Lab Middleware)

#### il_check_ins
**Daily mood/energy/stress tracking. Any module can write.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `source_module` | String | yes | default "dashboard" |
| `mood` | Number | yes | min 1, max 10 |
| `energy` | Number | yes | min 1, max 10 |
| `stress` | Number | yes | min 1, max 10 |
| `intention` | String | no | default "" |
| `notes` | String | no | default "" |
| `created_at` | Date | no | default Date.now |

**Indexes**: `{user_id: 1, created_at: -1}`

---

#### il_consciousness_profiles
**User's spiritual archetype and orientation. One per user.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | **unique**, indexed |
| `archetype` | String | no | default "" |
| `orientation` | String | no | default "" |
| `assessment_data` | Mixed | no | default {} |
| `source_module` | String | yes | default "dashboard" |
| `created_at` | Date | auto | |
| `updated_at` | Date | auto | Pre-save hook |

---

#### il_consciousness_snapshots
**Historical consciousness profile changes.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `archetype` | String | yes | |
| `orientation` | String | no | default "" |
| `assessment_data` | Mixed | no | default {} |
| `source_module` | String | yes | |
| `snapshot_reason` | String | no | default "profile_update" |
| `created_at` | Date | no | default Date.now |

**Indexes**: `{user_id: 1, created_at: -1}`

---

#### il_personal_histories
**User's life story for AI personalization. One per user.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | **unique**, indexed |
| `content` | Mixed | yes | default {} |
| `source_module` | String | yes | default "dashboard" |
| `created_at` | Date | auto | |
| `updated_at` | Date | auto | Pre-save hook |

---

#### il_user_memories
**AI-extracted facts about the user. Privacy-controlled cross-module sharing.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `source_module` | String | yes | |
| `memory_type` | String | yes | enum: `fact`, `preference`, `emotional_state`, `insight` |
| `content` | String | yes | |
| `confidence` | Number | no | min 0, max 1, default 0.8 |
| `shared` | Boolean | yes | default false |
| `shared_at` | Date | no | default null |
| `created_at` | Date | auto | |
| `updated_at` | Date | auto | Pre-save hook |
| `expires_at` | Date | no | default null |

**Indexes**: `{user_id: 1, source_module: 1}`, `{user_id: 1, shared: 1}`, `{expires_at: 1}` (TTL)

**Privacy rule**: When `shared: false`, only the module matching `source_module` can read this memory. When `shared: true`, all IL modules can read it. Users toggle sharing via innerlab.ai/memories.

---

#### il_user_wellness_profiles
**Health conditions, injuries, goals. One per user. Used by FlowState, BreathArc.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | **unique**, indexed |
| `health_conditions` | [String] | no | default [] |
| `injuries` | [String] | no | default [] |
| `goals` | [String] | no | default [] |
| `source_module` | String | yes | default "dashboard" |
| `created_at` | Date | auto | |
| `updated_at` | Date | auto | Pre-save hook |

---

#### il_activity_feed
**Cross-module activity events for the dashboard feed.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `source_module` | String | yes | |
| `action` | String | yes | e.g., "session_complete", "achievement_earned" |
| `title` | String | yes | Human-readable title |
| `description` | String | no | default "" |
| `metadata` | Mixed | no | default {} |
| `created_at` | Date | no | default Date.now |

**Indexes**: `{user_id: 1, created_at: -1}`

---

#### il_analytics_events
**User event tracking. Auto-expires after 90 days.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `event_type` | String | yes | |
| `source_module` | String | no | default "dashboard" |
| `metadata` | Mixed | no | default {} |
| `created_at` | Date | no | default Date.now |

**Indexes**: `{created_at: 1}` (TTL: 90 days), `{user_id: 1, event_type: 1}`

---

#### il_notifications
**Cross-module notifications.**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `user_id` | String | yes | indexed |
| `type` | String | yes | |
| `title` | String | yes | |
| `message` | String | no | default "" |
| `source_module` | String | no | default "system" |
| `read` | Boolean | no | default false |
| `read_at` | Date | no | default null |
| `action_url` | String | no | default null |
| `created_at` | Date | no | default Date.now |

**Indexes**: `{user_id: 1, read: 1, created_at: -1}`

---

#### Other il_* Collections (reserved)

| Collection | Purpose | Status |
|-----------|---------|--------|
| `il_blockchain_anchors` | OpenTimestamps data integrity proofs | Model exists, not yet populated |
| `il_sync_backups` | Local-first storage sync | Model exists, not yet populated |

---

### Module Collections (owned by individual module backends)

#### CWG (cwg_* prefix — 39+ collections)
Key collections:
- `cwg_chat_sessions` — AI chat sessions with guides
- `cwg_messages` — Individual chat messages
- `cwg_journal_entries` — User journal entries
- `cwg_practices` — Spiritual practices
- `cwg_guides` — Guide configurations (21 guides)
- Plus ~34 more collections for wisdom texts, community reflections, etc.

#### FlowState (yoga_* prefix — 7 collections)
- `yoga_user_profiles` — Yoga-specific user profiles
- `yoga_sessions` — Practice session records
- `yoga_achievements` — Achievement/badge tracking
- `yoga_community_flows` — Shared community flows
- `yoga_activity` — Module-specific activity log

---

## 3. Cross-Database Rules

### user_id Conventions
| Database | Field Name | Type | Value |
|----------|-----------|------|-------|
| `mbs_platform` | `user_id` | ObjectId | `ref: "User"` |
| `inner_lab` (all il_* collections) | `user_id` | **String** | `User._id.toString()` from JWT |
| `inner_lab` (module collections) | `user_id` | **String** | Same as above |
| Standalone apps | varies | ObjectId or String | Same user ID from JWT `userId` |

### GDPR Cascade
When MBS Platform deletes a user:
1. Platform deletes all documents in `mbs_platform` where `user_id` matches
2. Platform connects to `inner_lab` and deletes all documents where `user_id` matches (across ALL il_* collections)
3. Platform calls `DELETE /api/user-data` on each standalone app
4. `consentauditlogs.user_id` is replaced with SHA-256 hash (never deleted)

### Collection Ownership Rules
- Modules **write** to: their own prefix + `il_check_ins`, `il_user_memories`, `il_activity_feed`
- Modules **read** from: their own prefix + all `il_*` shared collections
- Modules **NEVER write** to another module's prefix
- Inner Lab Middleware **reads** from ALL prefixes, **writes** only to `il_*`
