# INNER LAB MASTER ARCHITECTURE

## CORE PRINCIPLE
Inner Lab is a unified system with:
- One identity (users)
- One database (currently: conversations_with_god)
- Multiple modules (CWG, Yoga, future apps)
- Access controlled via entitlements

---

## DATABASE ARCHITECTURE

### Single Database
- Use ONE MongoDB database: `conversations_with_god`
- This is now the Inner Lab master database

---

### Shared Collections (Core System)
- users
- auth_identities
- subscriptions
- entitlements
- sessions
- profiles (optional unified profile)

---

### Module Collections

#### CWG
- cwg_conversations
- cwg_messages
- cwg_journal_entries
- cwg_consciousness_profile
- cwg_personal_history

#### Yoga
- yoga_sessions
- yoga_progress
- yoga_preferences

---

### Rules
- Every document must include `user_id`
- No duplication of user data
- Prefix all module collections
- Shared collections must remain reusable

---

## AUTHENTICATION MODEL

- Single Sign-On via Inner Lab
- Modules rely on shared identity
- Authentication handled centrally

---

## SIGNUP FLOW (SOFT REDIRECT)

1. User lands on module (e.g., CWG)
2. Clicks Sign Up
3. Redirected to Inner Lab:
   `/signup?source=cwg`
4. Account created in `users`
5. Entitlement granted
6. Redirect back to module

---

## ACCESS CONTROL

- Controlled via `entitlements`
- Example:
  - cwg_access = true
  - yoga_access = false

---

## FUTURE EXPANSION

- Add new modules with:
  - prefixed collections
  - no schema conflicts
  - shared identity

---

## AGENT INSTRUCTION

All apps must:
- Use shared identity
- Follow collection prefix rules
- Use entitlements for access
- Avoid creating new databases
