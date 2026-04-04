# Codex Review Guide — MBS Platform Architecture Reviews

**Last Updated**: April 4, 2026
**Purpose**: Reference guide for running architecture review sessions between Claude Code and the Innerlab Incubator (Codex's planning repo).

---

## What This Is

Codex (OpenAI) builds Inner Lab module plans in the **Innerlab Incubator** repo. Claude Code (Anthropic) builds the live platform, middleware, and module code in the actual repos. Periodically, Codex's planning assumptions need to be validated against the live runtime to prevent architectural drift.

This document captures:
- The review process and how it works
- Where documents live
- What's been reviewed so far
- A ready-to-use prompt for starting new review sessions

---

## Key Locations

| What | Path |
|------|------|
| **Incubator docs (Codex's plans)** | `C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab Incubator\docs\` |
| **Review brief (questions from Codex)** | `...\Innerlab Incubator\docs\27-claude-cross-reference-pack.md` |
| **Response log (Claude's answers)** | `...\Innerlab Incubator\docs\28-claude-response-log.md` |
| **Touchpoint protocol** | `...\Innerlab Incubator\docs\29-claude-touchpoint-protocol.md` |
| **Live repos Claude checks against** | `Desktop\Codes\MBS\`, `Desktop\Codes\Innerlab\`, `Desktop\Codes\CWG\`, `Desktop\Codes\YogaGhost\`, `Desktop\Codes\DreamLens\` |
| **Architecture docs** | `Desktop\Marketing\Architecture Docs\` |
| **This guide** | `Desktop\Codes\MBSPlatform\CODEX_REVIEW_GUIDE.md` |

### Incubator Doc Index (40 docs)

The Incubator has a structured document series. Key docs for reviews:

| Doc | Title | Relevance |
|-----|-------|-----------|
| 00 | Source of Truth Hierarchy | Which docs override which |
| 05 | Inner Lab System Rules | Core architecture rules |
| 06 | Module Template | How new modules should be structured |
| 11 | Current Architecture Snapshot | Live state reference |
| 16 | Inner Lab Shared Surface Spec | Shared UI/UX contracts |
| 17 | CWG to Inner Lab Boundary | Migration boundary |
| 18 | Shared Journal Collection Contract | il_reflections spec |
| 23 | Inner Lab Platform Capability Architecture | What IL platform provides |
| 24 | Exact Module Capability Blueprints | Per-module feature plans |
| 25 | Cross-Module Coherence Review | Module overlap checks |
| 26 | Architecture Doc Alignment & Revalidation | Live vs proposed status |
| **27** | **Claude Cross-Reference Pack** | **THE review brief — start here** |
| **28** | **Claude Response Log** | **Append-only answer log** |
| 29 | Claude Touchpoint Protocol | When to ask Claude vs proceed independently |
| 30 | Assumption Status Model | Track confirmed/provisional assumptions |
| 33 | Agent Build Readiness Standard | Module graduation criteria |
| 40 | Implementation Agent Handoff Protocol | Codex → build agent handoff |

---

## Review Process

### How It Works

1. **Codex adds questions to doc 27** — tagged with IDs like `LAYER-01`, `DREAM-DB-PRIORITY-01`, `ASTRO-05`, etc.
2. **You start a Claude Code session** pointing at the Desktop folder
3. **Claude reads doc 27**, inspects linked incubator docs, compares assumptions against live code
4. **Claude appends answers to doc 28** in a new dated round — never overwrites prior rounds
5. **Codex reads doc 28** and reconciles corrections back into its planning docs

### Answer Format

Each answer uses this structure:
```
- ID: `QUESTION-ID`
  Status: `confirmed` | `corrected` | `safe enhancement` | `risky / not recommended` | `needs live-code check`
  Answer: ...
  Why: ...
  Sources checked: ...
  Recommended incubator update: ...
```

### Touchpoint Tiers (from doc 29)

**Tier 1 — Immediate check required:** New il_* collections, shared APIs, auth/token changes, Layer ownership boundaries, deletion/privacy/consent rules, cross-module write permissions, migration assumptions.

**Tier 2 — Batched review recommended:** New module blueprints, launch order changes, dashboard UX concepts, shared journal UX, migration sequencing.

**Tier 3 — Codex proceeds independently:** Naming decisions within module scope, internal feature design, UX copy, competitive positioning, marketing angles.

---

## Review History

### Round 1 — Session 16 (April 2, 2026)
- **Scope**: Full 30-question cross-reference review + 7 follow-ups
- **3 critical findings**:
  1. **FlowState is NOT movement-only** — it's yoga + breathwork + meditation (BreathArc overlap)
  2. **CWG creates memories silently** via AI extraction (not user-consented suggestions)
  3. **Nexus premiumFeatures conflict** — products.js lists intelligence-layer features, not support-bridge features
- **Key decisions**: BreathArc provisionally removed, Layer 2 contract documented, module guardrails established
- **Build ownership split confirmed**: Claude Code builds platform/middleware, Codex plans modules

### Round 2 — Session 17 (April 3, 2026)
- **Scope**: Architecture review — 14 decisions confirmed
- **Key confirmations**: Identity data always shared (no toggle), module activity data gets per-module toggle (Option B retroactive), GDPR app-level filters by source_module, dashboard requires any IL subscription
- **Critical bugs found**: CWG GDPR missing collections, FlowState zero il_* writes

### Round 3 — Session 18 (April 3, 2026)
- **Scope**: Full module review (all 11 modules against 10 architecture rules) + RS256 correction + birth data ownership
- **Key decisions**: Birth data owned by Inner Lab at Layer 2 (il_birth_profiles), RS256 dual-mode is correct
- **Answered**: AUTH-04, JOURNAL-09, JOURNAL-10, BUILD-03, NEXUS-05, ASTRO-05, ASTRO-06
- **All follow-ups resolved** — no remaining open questions

### Session 19 Updates (April 4, 2026) — NOT YET IN DOC 28
These changes happened after the last review round. Doc 27 has new DreamLens questions that need answering:
- **Birth Profile dashboard UI built** — `/birth-profile` page live at innerlab.ai
- **Sharing toggle implemented** — `il_sharing_preferences` collection, `/api/sharing` routes, `/sharing` page
- **CWG bidirectional sync** — CWG now reads identity from il_* first (commit `49803ba`)
- **GDPR updated to 14 collections** (added il_sharing_preferences)
- **DreamLens questions pending** — `DREAM-DB-PRIORITY-01` through `DREAM-DB-PRIORITY-04` need answering

---

## Ready-to-Use Prompt

Copy this entire block into a new Claude Code session. Set the working directory to `C:\Users\1984a\OneDrive\Desktop` so Claude has access to all repos.

```
Review Codex's architecture questions and provide responses.

Working directory: Desktop (you need access to Codes/MBS, Codes/Innerlab, Codes/CWG, Codes/YogaGhost, Codes/DreamLens, and Codes/Innerlab Incubator)

Step 1: Read the review brief:
- C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab Incubator\docs\27-claude-cross-reference-pack.md

Step 2: Read the existing response log to understand what's already been answered:
- C:\Users\1984a\OneDrive\Desktop\Codes\Innerlab Incubator\docs\28-claude-response-log.md
(Read the last round to see where we left off — do NOT overwrite prior rounds)

Step 3: For context on the review process:
- C:\Users\1984a\OneDrive\Desktop\Codes\MBSPlatform\CODEX_REVIEW_GUIDE.md

Step 4: Read any linked incubator docs that doc 27 references for context.

Step 5: Compare Codex's assumptions against the LIVE CODE in:
- Codes/MBS/ (MBS Platform — auth, billing, entitlements)
- Codes/Innerlab/ (Inner Lab — middleware, dashboard, il_* collections)
- Codes/CWG/ (Conversations with God — test branch)
- Codes/YogaGhost/ (FlowState)
- Codes/DreamLens/ (DreamLens — new module)

Step 6: Append a NEW dated round to docs/28-claude-response-log.md with your answers.

For each answer, include:
- ID (from doc 27)
- Status: confirmed | corrected | safe enhancement | risky / not recommended | needs live-code check
- Answer
- Why
- Sources checked
- Recommended incubator update

IMPORTANT RULES:
- NEVER delete or overwrite earlier rounds in doc 28 — append only
- Check LIVE CODE, not just docs — docs can be stale
- If doc 27 has no new unanswered questions, say so and ask if there's anything specific to review
- The DreamLens DB_NAME question (DREAM-DB-PRIORITY-01 through 04) is the current highest priority if it hasn't been answered yet

Key architecture context (Session 19 updates):
- Inner Lab now has 14 il_* collections (added il_sharing_preferences)
- Birth Profile dashboard UI is LIVE at innerlab.ai/birth-profile
- Sharing toggle is LIVE at innerlab.ai/sharing (per-module, retroactive Option B)
- CWG reads identity data from il_* first, falls back to cwg_user_profiles (bidirectional sync)
- CWG is on test branch (not merged to development/main)
- FlowState still writes zero il_* data (deferred)
```

---

## Tips for Running Reviews

1. **Start from Desktop level** — the session needs access to multiple repo folders under `Codes/`
2. **Don't use MBSPlatform as working directory** — it's a docs-only repo, the review needs access to live code repos
3. **One review session can handle multiple question batches** — Codex can add questions to doc 27 anytime, and you can point Claude at it
4. **Check doc 27's "Active" section first** — that's where the most urgent questions live
5. **This session and the build session can run in parallel** — as long as the review session only writes to Incubator docs and the build session works in the actual repos, there are no conflicts
