You are building a new Inner Lab module.

## INPUT FILES

- INNER_LAB_MASTER.md → system architecture
- plot.md → app-specific concept

---

## YOUR TASK

1. Read plot.md and understand the app concept
2. Map it into Inner Lab architecture

---

## RULES

### Database
- Use existing DB: conversations_with_god
- Do NOT create a new database
- Create module-prefixed collections:
  - e.g. astro_*, breathwork_*

### Identity
- Use shared users collection
- Do NOT create new user tables

### Auth
- Use Inner Lab SSO
- Redirect signup:
  /signup?source=<app>

### Access
- Use entitlements

---

## OUTPUT

Design:
- DB schema (collections)
- API structure
- Auth flow
- Integration points

Ensure full compatibility with Inner Lab system.
