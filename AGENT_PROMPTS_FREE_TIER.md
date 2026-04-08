# Agent Prompts — Free Tier Entitlement Gating

**Created**: April 8, 2026 (Session 30)
**Architecture**: See `FREE_TIER_ARCHITECTURE.md` for the full spec.

These are paste-ready prompts for each product's Claude Code agent. The MBS Platform build prompt comes first (builds the billing page + backend), then child app prompts.

---

## 1. MBS Platform Agent (build in MBS/ folder)

```
Task: Update billing page to show free vs premium tiers, and update WildLens to support free tier

Context:
The MBS Platform already has free tier infrastructure:
- POST /api/entitlements/subscribe-free — creates free entitlement (no trial by default; trial days configurable per product via admin)
- GET /api/entitlements/:product — returns hasAccess, isPremium, reason, limits
- products.js has freeTier: true and freeTierLimits for 22 of 23 products
- Trial-to-free downgrade already works

What needs to change:

1. products.js — Set WildLens freeTier to true:
   Change: freeTier: false → freeTier: true
   Change: freeTierLimits: {} → freeTierLimits: { scansPerDay: 3 }
   (Keep freeTierLimits empty {} if you prefer — module agent can set the actual value later)

2. BillingPage.jsx — Redesign to show free vs premium per product:
   - Group products by category (Inner Lab, Arcade, Studio Works) — already done
   - For each product, show TWO tiers side by side:
     - Free: list limits from freeTierLimits (e.g., "5 messages/day"), "Register Free" button, "Start using for free"
     - Premium: existing pricing, "Subscribe" button
   - State-aware badges per product:
     - Not registered: "Register Free" + "Subscribe" buttons
     - Free tier active: "Active (Free)" green badge + "Upgrade to Premium" button
     - Trial active: "Trial (X days left)" yellow badge + "Subscribe to Keep Premium"
     - Premium active: "Premium" purple badge + "Manage Subscription"
   - Fetch user's entitlements on page load: GET /api/entitlements (already returns all)
   - "Register Free" button calls POST /api/entitlements/subscribe-free { product: slug }
   - After registration, redirect to the product's URL with ?refresh=true

3. Deep-link support:
   - ?product=<slug> — scroll to that product's card, highlight it, optionally auto-open registration
   - ?register=<slug> — auto-call subscribe-free for that product, then redirect to product URL
   - ?register=innerlab — special case: register for Inner Lab category, redirect to innerlab.ai/dashboard

4. IndividualPlansPage.jsx — Update to show free tier info alongside pricing:
   - Each product card shows "Free: X/day" and "Premium: Unlimited + $X/mo"

5. Tests — Add tests for:
   - BillingPage renders free tier info for each product
   - "Register Free" button calls subscribe-free endpoint
   - Deep-link ?product= scrolls to correct card
   - State badges render correctly for free/trial/premium users

6. Admin Dashboard — Trial Campaign Config:
   - Add a section in AdminPage (or admin/AdminEntitlements) where admin can set defaultTrialDays per product
   - Show a table of all products with current defaultTrialDays value (default 0 = no trial)
   - Admin clicks "Edit" → sets trial days (e.g., 7) → saves to products config or a new TrialConfig collection
   - When defaultTrialDays > 0 for a product, the subscribe-free endpoint gives new free registrants a premium trial for that many days
   - This is a marketing campaign tool — admin turns trials on/off per product as needed

Do NOT change:
- Stripe or BTCPay integration
- The entitlement model or middleware
- Existing premium subscription flow

Design notes:
- Dark aesthetic, glass cards, Framer Motion animations (existing MBS style)
- Use Sonner toasts for success/error (never alert/prompt/confirm)
- Mobile-first layout
- Free tier card should feel welcoming, not restrictive
```

---

## 2. Inner Lab Agent (build in Innerlab/ folder)

```
Task: Add entitlement gating to Inner Lab — require free registration before accessing dashboard features

Context:
MBS Platform now has free tier registration. Every product requires explicit registration (even free) before access. Inner Lab needs to gate its dashboard behind entitlement checks.

The MBS Platform entitlement API:
- GET {PLATFORM_URL}/api/entitlements/category/innerlab — returns all IL products with access info
- Each product response includes: { hasAccess, isPremium, reason, limits }
- PLATFORM_URL is the MBS Platform backend (https://api.magicbusstudios.com)

What needs to change:

1. Entitlement check on dashboard load:
   - When user hits /dashboard (or any authenticated page), call GET {PLATFORM_URL}/api/entitlements/category/innerlab
   - If the user has NO entitlements for any IL product → redirect to magicbusstudios.com/billing?category=innerlab
   - If the user has at least one IL product entitlement → show dashboard with only registered modules

2. Module picker page (new page: /modules):
   - Shows all 12 IL modules in a grid
   - Each module card shows: name, description, free tier limits, premium features
   - "Register Free" button per module → calls POST {PLATFORM_URL}/api/entitlements/subscribe-free { product: slug }
   - Already registered: "Active (Free)" or "Premium" badge
   - After registration, module appears in dashboard sidebar

3. Dashboard sidebar:
   - Only show modules the user has registered for
   - "Add Module" link → /modules page
   - Modules without entitlement are hidden (not shown as locked)

4. Free vs premium feature gating:
   - You decide which IL dashboard features are free vs premium
   - Use the entitlement response: if isPremium is false for any IL product, show limits
   - Suggested split (adjust as you see fit):
     - Free: dashboard overview, check-ins (limited/day), basic journal
     - Premium: consciousness assessment, weekly review, daily briefing, cross-module insights, sharing
   - Show upgrade prompts for premium features: "Upgrade to Premium to unlock this feature"

5. Handle ?refresh=true query param:
   - When redirected back from MBS billing page after registration
   - Re-fetch entitlements to pick up the new registration

Environment:
- PLATFORM_URL env var already exists (https://api.magicbusstudios.com)
- JWT token is passed in Authorization header for all platform API calls
- Use existing auth context for the token

Do NOT change:
- The MBS Platform entitlement API (it already works)
- Authentication flow (SSO via MBS Platform stays the same)
- Existing il_* data models or collections
- Backend routes (check-ins, consciousness, reflections etc. stay the same)
```

---

## 3. CWG Agent (build in CWG/ folder)

```
Task: Add entitlement gating — CWG must require free registration before allowing access

Context:
MBS Platform now requires explicit registration (even free) before accessing any product. CWG needs to check for an entitlement on every authenticated page load and redirect to the billing page if none exists.

The MBS Platform entitlement API:
- GET {PLATFORM_URL}/api/entitlements/cwg — returns { hasAccess, isPremium, reason, limits }
- reason can be: "free_tier", "trial", "product_pass", "category_access", "mbs_all_access"
- limits for CWG free tier: { messagesPerDay: 5 }
- PLATFORM_URL = MBS_PLATFORM_URL env var (https://api.magicbusstudios.com)

What needs to change:

1. Backend entitlement check:
   - On every authenticated API request (or at minimum, on chat/message endpoints), check entitlement
   - Call GET {MBS_PLATFORM_URL}/api/entitlements/cwg with the user's JWT
   - If hasAccess is false → return 403 { success: false, message: "Registration required", redirectUrl: "https://magicbusstudios.com/billing?product=cwg" }
   - Cache the result for the session (don't call on every single request — cache for 5 minutes)

2. Frontend entitlement check:
   - On app load (after auth), call the CWG backend to check entitlement
   - If 403 with redirectUrl → redirect to magicbusstudios.com/billing?product=cwg
   - If free tier → store isPremium: false and limits in app state
   - If premium → store isPremium: true

3. Message limit enforcement (free tier):
   - Count messages sent today by this user
   - If count >= limits.messagesPerDay (default 5) and NOT premium → block with message: "You've reached your daily message limit. Upgrade to Premium for unlimited conversations."
   - Show remaining messages count in the UI (e.g., "3/5 messages today")

4. Premium feature gating:
   - premiumFeatures from products.js: ["unlimited_messages", "advanced_guides", "conversation_history", "export_conversations"]
   - Free users: basic guides only, no conversation history, no export
   - Show "Premium" lock icon on gated features with upgrade link to magicbusstudios.com/billing?product=cwg

5. Handle ?refresh=true:
   - When redirected back from billing page after registration
   - Re-check entitlement to pick up new access

Environment:
- MBS_PLATFORM_URL env var exists (defaults to https://api.magicbusstudios.com in config.py)
- CWG is FastAPI (Python). Use httpx for the platform API call.
- JWT is available from the auth dependency

Do NOT change:
- Authentication flow (SSO stays the same)
- Database models or collections
- Guide content or conversation logic
- The MBS Platform API (it already works)
```

---

## 4. FlowState Agent (build in YogaGhost/ folder)

```
Task: Add entitlement gating — FlowState must require free registration before allowing access

Context:
MBS Platform now requires explicit registration (even free) before accessing any product. FlowState needs to check for an entitlement and redirect to billing if none exists.

The MBS Platform entitlement API:
- GET {PLATFORM_URL}/api/entitlements/flowstate — returns { hasAccess, isPremium, reason, limits }
- limits for FlowState free tier: { sessionsPerDay: 3 }
- PLATFORM_URL = MBS_PLATFORM_URL env var (https://api.magicbusstudios.com)

What needs to change:

1. Frontend entitlement check (FlowState is Express + React):
   - On app load (after auth), call GET /api/entitlements/check (new backend route)
   - Backend route proxies to MBS Platform: GET {MBS_PLATFORM_URL}/api/entitlements/flowstate
   - If hasAccess is false → redirect to magicbusstudios.com/billing?product=flowstate
   - If free tier → store limits in app state

2. Session limit enforcement (free tier):
   - Count sessions completed today by this user
   - If count >= limits.sessionsPerDay (default 3) and NOT premium → block with: "You've reached your daily session limit. Upgrade to Premium for unlimited practice."
   - Show remaining sessions in the UI (e.g., "1/3 sessions today")

3. Premium feature gating:
   - premiumFeatures: ["unlimited_sessions", "custom_sequences", "progress_tracking", "offline_mode"]
   - Free: basic sessions, community flows
   - Premium: custom sequences, progress tracking, offline mode
   - Lock icon on premium features with upgrade link

4. Add backend route:
   - GET /api/entitlements/check — calls MBS Platform, returns entitlement info
   - Cache result for 5 minutes per user

5. Handle ?refresh=true after billing page redirect

Do NOT change: auth flow, session logic, database models, il_* integrations
```

---

## 5. Generic Prompt — All Arcade & Studio Works Apps

Use this prompt for: BrokenChain, MindHacker, Trivia Roast, Fake Artist, Whispering House, WildLens, Lazy Chef, TaskTracker, AI Tutor, SmartCart, Movie Picker

Replace `{PRODUCT_SLUG}`, `{PRODUCT_NAME}`, `{LIMIT_KEY}`, `{LIMIT_VALUE}`, and `{LIMIT_DESCRIPTION}` with the correct values for each app.

```
Task: Add entitlement gating — {PRODUCT_NAME} must require free registration before allowing access

Context:
MBS Platform now requires explicit registration (even free) before accessing any product. This app needs to check for an entitlement and redirect to the billing page if none exists.

The MBS Platform entitlement API:
- GET {PLATFORM_URL}/api/entitlements/{PRODUCT_SLUG} — returns { hasAccess, isPremium, reason, limits }
- Free tier limits for this app: { {LIMIT_KEY}: {LIMIT_VALUE} }
- PLATFORM_URL env var (https://api.magicbusstudios.com)

What needs to change:

1. Frontend entitlement check:
   - On app load (after auth), check entitlement
   - Call your own backend: GET /api/entitlements/check (new route)
   - Backend proxies to: GET {PLATFORM_URL}/api/entitlements/{PRODUCT_SLUG} with user JWT
   - If hasAccess is false → redirect to magicbusstudios.com/billing?product={PRODUCT_SLUG}
   - If free tier → store limits in app state
   - Cache entitlement for 5 minutes

2. Limit enforcement (free tier):
   - {LIMIT_DESCRIPTION}
   - Show usage counter in UI
   - When limit reached, show: "Daily limit reached. Upgrade to Premium for unlimited access."
   - Include link to magicbusstudios.com/billing?product={PRODUCT_SLUG}

3. Add backend route:
   - GET /api/entitlements/check — calls MBS Platform, returns { hasAccess, isPremium, limits }
   - Authenticated route (requireAuth)

4. Handle ?refresh=true — re-check entitlement after billing page redirect

Do NOT change: auth flow, core game/app logic, database models
```

### Variable Reference Table

| App | PRODUCT_SLUG | LIMIT_KEY | LIMIT_VALUE | LIMIT_DESCRIPTION |
|-----|-------------|-----------|-------------|-------------------|
| BrokenChain | brokenchain | minutesPerDay | 30 | Track play time per day. Block new games after 30 minutes. |
| MindHacker | mindhacker | minutesPerDay | 30 | Track play time per day. Block new games after 30 minutes. |
| Trivia Roast | triviaroast | minutesPerDay | 30 | Track play time per day. Block new games after 30 minutes. |
| Fake Artist | fakeartist | minutesPerDay | 30 | Track play time per day. Block new games after 30 minutes. |
| Whispering House | whisperinghouse | minutesPerDay | 30 | Track play time per day. Block new games after 30 minutes. |
| WildLens | wildlens | scansPerDay | 3 | Count AI scans today. Block after 3. |
| Lazy Chef | lazychef | recipesPerDay | 5 | Count recipes generated today. Block after 5. |
| Task Tracker | tasktracker | tasksTotal | 50 | Count total tasks in DB. Block creation after 50. |
| AI Tutor | tutor | sessionsPerDay | 3 | Count tutoring sessions today. Block after 3. |
| SmartCart | smartcart | listsTotal | 5 | Count total lists in DB. Block creation after 5. |
| Movie Picker | moviepicker | picksPerDay | 10 | Count AI picks today. Block after 10. |
