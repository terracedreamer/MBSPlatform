You are updating the Conversations With God (CWG) app to align with Inner Lab architecture.

## KEY OBJECTIVES

1. Treat CWG as a module within Inner Lab
2. Use shared identity system
3. Refactor database structure to support:
   - shared collections
   - module-specific collections (prefixed with cwg_)
4. Do NOT create a new database

---

## DATABASE RULES

- Use existing MongoDB database: conversations_with_god
- Create/ensure shared collections:
  - users
  - entitlements
  - subscriptions
  - sessions

- Move CWG-specific data into prefixed collections:
  - cwg_conversations
  - cwg_messages
  - etc.

- Ensure every document uses `user_id`

---

## AUTH FLOW

- Remove standalone CWG auth
- Redirect signup/login to Inner Lab:
  /signup?source=cwg

- After auth:
  - grant cwg entitlement
  - redirect back to CWG

---

## UX

- Messaging should say:
  "Create your Inner Lab account to access Conversations with God"

---

## OUTPUT

Refactor:
- Auth
- DB structure
- Routing
- User handling

Maintain backward compatibility where possible.
