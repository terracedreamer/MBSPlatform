# Arcade Brand DNA Compliance Audit

**Date:** 2026-03-29
**Scope:** All 5 Arcade games audited against the bootstrap-arcade Brand DNA standard
**Mode:** Read-only audit — no changes made

---

## Legend

- **+** = Compliant
- **X** = Missing / Not implemented
- **~** = Partially implemented
- **!** = Fundamentally incompatible
- **N/A** = Not applicable (architecture doesn't support it)

---

## 1. TRIVIA ROAST — Most Deviant

**! Fundamentally different architecture.** This is a single-player pass-the-device party game, not a multiplayer room-based game.

**Location:** `~/Desktop/Codes/Trivia`
**Stack:** Express + Vanilla HTML/JS (single `index.html`)

| Deviation | Details |
|-----------|---------|
| ! No multiplayer rooms | All players share one device. No Socket.io, no room codes, no host/join flow |
| ! Vanilla HTML/JS frontend | Single `index.html` with `showScreen()` toggling. No React, no routing, no components |
| ! No page routing | No `/lobby/:code`, `/game/:code`, `/results/:code` — all screens in one file |
| X No Socket.io | Not installed |
| X No reconnection | N/A — single device |
| X No host migration | N/A — no host concept |
| X No AI fallback | Game fails if OpenAI is down |
| X No response helpers | Manual `res.json()` |
| X No centralized error handler | Per-route try/catch |
| X No express-validator | Manual input checks |
| X No game state machine | State lives in frontend JS object |
| ~ Framer Motion | Uses CSS animations instead |
| ~ Button sizes | Primary buttons ~30-36px (below 44px standard) |
| + Sound effects | Web Audio API, toggleable |
| + Confetti | Canvas-based confetti on win |
| + Dark theme + glass cards | Correct `#0a0a0f`, correct glass morphism |
| + Fonts | Space Grotesk + DM Sans correct |
| + Winston logger | No console.log |
| + Rate limiting | Implemented |
| + GDPR endpoint | Exists (no-op, correct for this game) |
| + Podium | 1st/2nd/3rd on results screen |

**Verdict:** Would require a ground-up rewrite to match Arcade standard.

---

## 2. BROKENCHAIN

**Location:** `~/Desktop/Codes/Brokenchain`
**Stack:** Express + React/Vite + Socket.io

| Deviation | Severity | Details |
|-----------|----------|---------|
| X Results/podium screen | **HIGH** | No podium (1st/2nd/3rd). RevealPage shows chain timeline only. `Leaderboard.js` model exists but is unused. Scores defined in schema but never calculated |
| X In-game header | **HIGH** | No persistent header showing room code + round + timer + player count during gameplay |
| X Phase timer server-auth | **HIGH** | Timer is client-side only (`countdown.start(RECORD_TIME)`). Server doesn't enforce timeout |
| X Confetti | Medium | Not implemented, no dependency |
| X Reconnection (30s) | Medium | No grace period on disconnect |
| X Host migration | Medium | No auto-promote on host disconnect |
| X AI fallback | Medium | No static fallback phrases if OpenAI fails |
| ~ Game mode selection | Medium | UI exists but not wired to API — cosmetic only |
| X Room TTL | Low | No 24-hour expiry on rooms |
| X Response helpers | Low | Manual `res.json()` per route |
| ~ Route naming | Low | `/play/:gameId` instead of `/game/:code`; `/reveal/:gameId` instead of `/results/:code` |
| ~ Button sizes | Low | `sm` variant is 36px (below 44px) in 2 places |
| + Room codes | | Correct 6-char, excludes ambiguous chars |
| + Lobby | | Full compliance (host controls, settings, avatars) |
| + Sound effects | | All 5 required sounds via Web Audio API |
| + Visual design | | Excellent — dark theme, glass cards, particles, aurora BG |
| + Fonts | | Space Grotesk + DM Sans correct |
| + Winston logger | | No console.log |
| + GDPR endpoint | | Implemented with cascade deletion |
| + Input validation | | express-validator on all routes |
| + Rate limiting | | Implemented |
| + Entitlement check | | With caching |
| + Centralized error handler | | App-level middleware |

---

## 3. MINDHACKER

**Location:** `~/Desktop/Codes/Mindhacker`
**Stack:** Express + React 19 + Socket.io

| Deviation | Severity | Details |
|-----------|----------|---------|
| X Sound effects | **HIGH** | No Web Audio API sounds at all |
| X Confetti | **HIGH** | No confetti on winner |
| ~ Results screen | Medium | Has leaderboard but no visual podium. Missing "Play Again" button — only "Back to Main" |
| ~ Landing page buttons | Medium | Create/Join buttons only on Home `/`, not on Landing page |
| X Room TTL | Medium | No 24-hour expiry mechanism |
| X Response helpers | Low | Manual `res.json()` per route |
| X Centralized error handler | Low | No Express error middleware |
| ~ Reconnection | Low | Socket re-join exists but no explicit 30-second hold |
| ~ Player avatars | Low | Simple gradient boxes with initials, not full avatar images |
| + Room codes | | Correct 6-char, excludes ambiguous chars |
| + Lobby | | Full compliance |
| + In-game header | | Room code, round number, phase, timer, role badge all visible |
| + Game state machine | | LOBBY -> PLAYING -> FINISHED with phase rotation |
| + Phase timer | | Server-authoritative |
| + Host migration | | `room:transferHost` handler implemented |
| + AI fallback | | Inline fallback tasks/narrations |
| + Fonts | | Space Grotesk + DM Sans correct |
| + Dark theme + glass | | Fully compliant |
| + Framer Motion | | Throughout |
| + Winston logger | | No console.log |
| + GDPR endpoint | | Cascade deletion |
| + Input validation | | express-validator + XSS protection |
| + Rate limiting | | API + auth limiters |
| + Entitlement | | effectivePremium pattern with cache |
| + Leaderboard | | PlayerProfile model + leaderboard API |

---

## 4. FAKE ARTIST

**Location:** `~/Desktop/Codes/Fakeartist`
**Stack:** Express + React + Socket.io

| Deviation | Severity | Details |
|-----------|----------|---------|
| X Sound effects | **HIGH** | No Web Audio API sounds at all |
| X Confetti | **HIGH** | No confetti on winner |
| X Fonts | **HIGH** | Uses `Inter` instead of Space Grotesk + DM Sans |
| ~ Phase timer | Medium | Client-authoritative (clock drift risk) |
| X Reconnection (30s) | Medium | Immediate player removal on disconnect |
| X Response helpers | Low | Manual `res.json()` |
| X Centralized error handler | Low | No Express error middleware |
| ~ express-validator | Low | Installed but never imported/used — manual validation only |
| ~ AI fallback file | Low | Inline `FALLBACK_WORDS` object, not separate `staticContent.js` |
| ~ Results buttons | Low | No "Play Again" in results — only "Back to Home" |
| ~ Mobile button sizes | Low | Some drawing toolbar buttons may be below 44px |
| + Room codes | | Correct 6-char, excludes ambiguous chars |
| + Lobby | | Full compliance with host controls, settings, kick/transfer |
| + In-game header | | Room code, round, timer ring, player count all visible |
| + Results/podium | | Podium with medals + all scores |
| + Dark theme + glass | | 4 theme options, all dark-compatible |
| + Framer Motion | | Page transitions, staggered animations, spring physics |
| + Game state machine | | Full phase rotation |
| + Room TTL | | 24-hour TTL with MongoDB index |
| + Winston logger | | No console.log |
| + GDPR endpoint | | Stub (correct — no persistent user data) |
| + Rate limiting | | Implemented |
| + Host migration | | Auto-assigns on disconnect |
| + Entitlement | | Endpoint exists |

---

## 5. WHISPERING HOUSE

**Location:** `~/Desktop/Codes/Whispering House`
**Stack:** FastAPI (Python) + React 19 + native WebSockets (not Socket.io)

| Deviation | Severity | Details |
|-----------|----------|---------|
| X Sound effects | **HIGH** | No Web Audio API sounds at all |
| X Room code chars | **HIGH** | Uses `ascii_uppercase + digits` — **includes** I, O, 0, 1, L |
| X Host migration | **HIGH** | No failover when host disconnects |
| ~ Results podium | Medium | Sorted score list with star for #1, but no 3-position podium visual |
| ~ In-game header | Medium | Timer not in a consistent pinned header across all views |
| X Room TTL | Medium | No 24-hour expiry on sessions |
| X PlayerProfile leaderboard | Medium | Per-session only, no cross-game stats |
| X Centralized error handler | Medium | No `@app.exception_handler` registered |
| X Rate limiting | Medium | slowapi installed but **not wired** (no `@limiter` decorators) |
| ~ Reconnection | Low | 3-second reconnect intervals, not 30-second hold |
| X Player avatars | Low | Players shown by name only, no avatar images/initials |
| ~ Route naming | Low | `/play/:code` + `/host/:code` instead of `/game/:code` |
| ~ Results buttons | Low | "Play Again" exists but no "Back to Lobby" |
| ~ Fonts | Low | Playfair Display + Inter (deliberate thematic override) |
| ~ Entitlement naming | Low | Uses `hasAccess` not `effectivePremium` |
| + Confetti | | Canvas confetti on resolution |
| + AI fallback | | Ashworth family static mystery |
| + Dark theme + glass | | Correct glass morphism |
| + Framer Motion | | Page transitions throughout |
| + Game state machine | | LOBBY -> PLAYING -> RESOLVING -> RESOLVED |
| + Phase timer | | Server-authoritative (`timerStartedAt` UTC) |
| + Winston/logger | | Zero print() statements |
| + GDPR endpoint | | DELETE /api/user-data exists |
| + Input validation | | Pydantic schemas with constraints |
| + Entitlement check | | Fail-open pattern |

---

## Cross-Game Summary Matrix

| Feature | BrokenChain | MindHacker | Trivia Roast | Fake Artist | Whispering House |
|---------|:-----------:|:----------:|:------------:|:-----------:|:----------------:|
| **Sound Effects** | + | X | + | X | X |
| **Confetti** | X | X | + | X | + |
| **Podium (1/2/3)** | X | X | + | + | X |
| **Room Codes (no I/O/0/1)** | + | + | N/A | + | X |
| **Fonts (SG+DM)** | + | + | + | X | X |
| **In-game Header** | X | + | N/A | + | ~ |
| **Server-auth Timer** | X | + | N/A | X | + |
| **Reconnection (30s)** | X | ~ | N/A | X | ~ |
| **Host Migration** | X | + | N/A | + | X |
| **AI Fallback** | X | + | X | ~ | + |
| **Room TTL (24h)** | X | X | N/A | + | X |
| **Response Helpers** | X | X | X | X | N/A |
| **Error Handler** | + | X | X | X | X |
| **express-validator** | + | + | X | X | N/A |
| **Rate Limiting** | + | + | + | + | X |
| **GDPR Endpoint** | + | + | + | + | + |
| **Winston Logger** | + | + | + | + | + |
| **Entitlement Check** | + | + | N/A | + | + |

---

## Top Takeaways

1. **Sound effects** are the #1 gap — only BrokenChain and Trivia Roast have them (3 games missing)
2. **Response helpers** (`sendSuccess`/`sendError`) — missing in ALL games
3. **Centralized error handler** — only BrokenChain has one
4. **Confetti** — only Trivia Roast and Whispering House have it (3 missing)
5. **Room TTL (24h)** — only Fake Artist implements it
6. **Trivia Roast** is architecturally incompatible (single-player, vanilla JS, no rooms)
7. **Whispering House** has ambiguous room codes (includes I/O/0/1/L)
8. No game has centralized response helpers — this is a universal gap

---

## Recommended Priority Order

### P0 — Blocking / High Impact
1. Add sound effects to MindHacker, Fake Artist, Whispering House (Web Audio API)
2. Fix Whispering House room codes (exclude ambiguous chars)
3. Add confetti to BrokenChain, MindHacker, Fake Artist
4. Fix Fake Artist fonts (Inter -> Space Grotesk + DM Sans)
5. Add results podium to BrokenChain, MindHacker, Whispering House
6. Add in-game header to BrokenChain

### P1 — High Priority
7. Make BrokenChain timer server-authoritative
8. Make Fake Artist timer server-authoritative
9. Add host migration to BrokenChain, Whispering House
10. Wire up Whispering House rate limiting (slowapi decorators exist but unused)
11. Add centralized error handlers to MindHacker, Fake Artist, Whispering House

### P2 — Medium Priority
12. Add response helpers to ALL games
13. Implement reconnection (30s hold) across all games
14. Add 24-hour room TTL to BrokenChain, MindHacker, Whispering House
15. Add AI fallback to BrokenChain, Trivia Roast
16. Wire BrokenChain game mode selection to API

### P3 — Low / Polish
17. Fix route naming (BrokenChain `/play` -> `/game`, Whispering House `/play` -> `/game`)
18. Add "Play Again" button to MindHacker and Fake Artist results
19. Add "Back to Lobby" button to Whispering House results
20. Audit all button sizes for 44px minimum
21. Add player avatars to Whispering House

### Trivia Roast — Separate Track
Requires architectural decision: rewrite as multiplayer room-based game, or accept as a different product category.
