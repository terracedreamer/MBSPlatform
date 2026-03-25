You are updating the Yoga app to align with Inner Lab.

## OBJECTIVES

1. Integrate with Inner Lab identity system
2. Stop using separate database (yogaghost)
3. Use shared database: conversations_with_god

---

## DATABASE

- Use shared collections:
  - users
  - entitlements

- Create yoga-specific collections:
  - yoga_sessions
  - yoga_progress
  - yoga_preferences

- All documents must include user_id

---

## AUTH

- No standalone login
- Redirect to:
  /signup?source=yoga

- Grant yoga entitlement upon signup

---

## ACCESS CONTROL

- Check entitlements before allowing access

---

## OUTPUT

Refactor:
- DB usage
- Auth flow
- Routing
