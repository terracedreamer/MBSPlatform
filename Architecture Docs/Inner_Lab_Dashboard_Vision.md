# Inner Lab Dashboard — Product Vision & Build Guide

**Version**: 1.0
**Date**: April 2, 2026
**Purpose**: Guide for enhancing the Inner Lab dashboard (innerlab.ai) to become the primary user-facing experience for inner growth. Written for Codex.

---

## The Vision

Inner Lab should be the **star** of Magic Bus Studios. When a user logs in to innerlab.ai, they should see a unified, intelligent view of their inner growth journey — not just a list of links to separate modules.

Today, Inner Lab only has data from CWG (Conversations With God). But the architecture is built for 11 modules to feed into a shared intelligence layer. The dashboard should be built to handle this eventual richness, while being useful NOW with whatever data exists.

---

## Current State (April 2026)

### What Exists
- **innerlab.ai**: Marketing pages + auth pages + basic dashboard
- **api.innerlab.ai**: Express backend with all il_* API routes
- **Dashboard pages**: `/dashboard`, `/consciousness`, `/memories`, `/activity`
- **Auth pages**: `/auth/login`, `/auth/signup`, `/auth/forgot-password`, `/auth/reset-password`

### What Feeds Data Today
- **CWG** writes to: `il_user_memories`, `il_check_ins`, `il_activity_feed`, `il_consciousness_profiles`
- **FlowState** writes to: `il_user_wellness_profiles`, `il_activity_feed`
- **Dashboard** itself writes to: `il_check_ins` (via check-in widget)

### What's Missing
1. **Daily Briefing** — The most important missing feature. Should synthesize: today's check-in, recent memories, module activity, consciousness trends.
2. **Cross-Module Insights** — Pattern detection across data. Even with CWG-only data, can surface themes.
3. **Rich Module Launcher** — Currently just links. Should show: last session time, streak count, recent activity preview.
4. **Unified Timeline** — Merge activity feed + memories + check-ins into a single chronological view.
5. **Data Visualization** — Mood/energy/stress trends over time, practice frequency charts.

---

## Dashboard Pages — What Each Should Do

### /dashboard (Main Hub)
**Current**: Module launcher grid + check-in widget + activity feed.
**Should become**:

```
┌──────────────────────────────────────────────────┐
│  Good morning, {name}.                            │
│  Here's your daily briefing for April 2, 2026.   │
│                                                    │
│  [Quick Check-In Widget]                          │
│  mood ●●●●●●●○○○  energy ●●●●●○○○○○              │
│                                                    │
│  ┌─ Today's Insights ──────────────────────────┐  │
│  │ "You've explored themes of forgiveness in    │  │
│  │  your last 3 CWG conversations. Your energy  │  │
│  │  tends to be higher on days you journal."     │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  ┌─ Module Launcher ──────────┐                   │
│  │ CWG          Last: 2h ago  │                   │
│  │ FlowState    Last: 3 days  │                   │
│  │ BreathArc    Coming Soon   │                   │
│  └────────────────────────────┘                   │
│                                                    │
│  ┌─ Recent Activity ─────────────────────────┐    │
│  │ • Completed conversation with Rumi (CWG)  │    │
│  │ • New memory: "User values solitude" (CWG) │    │
│  │ • Check-in: mood 7, energy 6, stress 4     │    │
│  └────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────┘
```

### /consciousness (Profile)
**Current**: Shows archetype and orientation.
**Should become**:
- Archetype card with visual representation
- Assessment history timeline (snapshots)
- "Retake Assessment" button
- Orientation breakdown
- Connection to CWG guide preferences

### /memories (Memory Manager)
**Current**: List of memories with sharing toggle.
**Should become**:
- Grouped by module (CWG memories, FlowState memories, etc.)
- Search across all memories
- Filter by type (fact, preference, emotional_state, insight)
- Confidence meter for each memory
- Bulk sharing controls
- "What does Inner Lab know about me?" summary view

### /activity (Activity Feed)
**Current**: Chronological list of events.
**Should become**:
- Filterable by module, action type, date range
- Visual timeline (not just a list)
- Aggregated views: "This week you had 5 CWG sessions, 2 FlowState practices"
- Milestone highlights

---

## API Enhancements Needed

### Daily Briefing API
```
GET /api/today
```
Returns a synthesized daily briefing:
```json
{
  "success": true,
  "briefing": {
    "greeting": "Good morning, Abhinav.",
    "check_in_prompt": true,
    "latest_check_in": { "mood": 7, "energy": 6, "stress": 4, "created_at": "..." },
    "insights": [
      "You've explored themes of forgiveness in your last 3 conversations.",
      "Your energy tends to be higher on days you practice yoga."
    ],
    "module_stats": {
      "cwg": { "last_session": "2h ago", "sessions_this_week": 5 },
      "flowstate": { "last_session": "3 days ago", "sessions_this_week": 0 }
    },
    "recent_memories": [...],
    "recent_activity": [...]
  }
}
```

**Implementation**: This reads from multiple il_* collections and synthesizes. Initially rule-based (string templates + date math). Later, can use OpenAI to generate natural language summaries.

### Trends API
```
GET /api/trends?period=7d
```
Returns mood/energy/stress averages over time, activity frequency, module usage breakdown.

### Module Stats API
```
GET /api/module-stats
```
Returns per-module statistics: last session, total sessions, streak, active days.

---

## Data Flow: How Modules Feed the Dashboard

```
CWG Backend                    FlowState Backend
    │                               │
    ├─ Writes cwg_* collections     ├─ Writes yoga_* collections
    ├─ Writes il_user_memories      ├─ Writes il_user_wellness_profiles
    ├─ Writes il_check_ins          ├─ Writes il_activity_feed
    ├─ Writes il_activity_feed      │
    ├─ Writes il_consciousness_*    │
    │                               │
    └───────────┬───────────────────┘
                │
                ▼
        inner_lab database
                │
                ▼
    Inner Lab Middleware (api.innerlab.ai)
                │
    ┌───────────┼───────────────────────────┐
    │           │                           │
    ▼           ▼                           ▼
 /api/today  /api/memories  /api/consciousness
    │           │                           │
    └───────────┼───────────────────────────┘
                │
                ▼
    innerlab.ai Dashboard (frontend)
```

---

## Design Principles for the Dashboard

1. **Useful with one module** — The dashboard must feel complete even when only CWG has data. Don't show empty states for 9 coming-soon modules. Show what IS available, richly.

2. **Progressive disclosure** — As more modules come online, the dashboard naturally gets richer. No code changes needed — it reads from il_* collections and any module that writes there automatically appears.

3. **Privacy-first** — Always show users what data is being shared across modules. The memories page is the control center.

4. **Warm, not clinical** — This is inner growth, not a fitness tracker. Use language like "Here's what we're noticing" not "Your metrics indicate."

5. **Dark aesthetic** — Match the existing Inner Lab design: deep dark backgrounds (#050505), teal/sky accents, glass card panels, Framer Motion animations.

---

## Technical Notes for Codex

### The Dashboard Frontend
- Lives in `Innerlab/src/pages/` (DashboardPage.jsx, ConsciousnessPage.jsx, MemoriesPage.jsx, ActivityPage.jsx)
- API calls via `src/utils/dashboardApi.js` — sends `Authorization: Bearer {mbs_token}` to api.innerlab.ai
- Auth state managed by AuthContext — checks localStorage for `mbs_token`
- Protected routes redirect to `/auth/login` if not authenticated

### The Dashboard Backend
- Lives in `Innerlab/server/` — Express app at api.innerlab.ai
- Routes: `server/routes/` (checkins.js, consciousness.js, memories.js, activity.js, export.js, encryption.js, userData.js, forms.js)
- Models: `server/models/` — 11 Mongoose models for all il_* collections
- Uses ES modules (`import`/`export`, not `require`)

### What Codex Should NOT Touch
- The MBS Platform (`MBS/` folder) — that's Layer 1, not your concern
- CWG backend code — that module manages itself
- FlowState backend code — same
- The `mbs_platform` database — only MBS Platform writes there
- Other module's prefix collections (cwg_*, yoga_*)

### Git Workflow
- Inner Lab repo: push to `main` branch
- Coolify auto-deploys on push
- Two Coolify services: "Innerlab A" (frontend) + "Innerlab B" (backend)
