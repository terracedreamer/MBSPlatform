# Inner Lab

## Product Marketing Brief

**Confidential — For Marketing Agent Use**

**innerlab.ai** | A MagicBusStudios Product

> **Last Updated**: March 25, 2026. Platform decisions integrated.

## Platform Context (New)
- Inner Lab modules share a database (inner_lab) with shared il_* collections for consciousness profiles, user memories (opt-in), check-ins, wellness profiles, and activity feed
- Each module is a separate container but reads/writes shared data through the Inner Lab Middleware at innerlab.ai
- innerlab.ai is being upgraded from a marketing site to also serve as the Inner Lab Dashboard for subscribers
- Dashboard features: daily briefing, unified consciousness profile, cross-module insights, activity feed, module launcher
- User memory privacy model: memories are per-module by default, users opt-in to share across modules
- Shared wellness profiles: health conditions and injuries from FlowState are readable by BreathArc and other modules
- Data sovereignty: Nostr identity, Lightning payments, client-side encryption, GDPR compliance
- Tiered access: product_pass (single module), category_access: innerlab (all 11 modules + dashboard + cross-module intelligence), mbs_all_access (everything in MBS)

---

## Section 1: Product Identity

| Field | Value |
|-------|-------|
| **Product Name** | Inner Lab |
| **Product Type** | Unified system for inner growth |
| **Website** | innerlab.ai |
| **Parent Company** | MagicBusStudios (magicbusstudios.com) |
| **Category** | Health & Wellness / Personal Growth / Self-Discovery |
| **Pricing Model** | Freemium — Free tier available, Premium subscriptions for unlimited access |
| **Total Modules** | 11 (2 Live, 9 Coming Soon) |

---

## Section 2: What is Inner Lab?

Inner Lab is a unified system for reflection, practice, and awareness that adapts to you over time. It brings together multiple modalities of inner growth — from spiritual dialogue to yoga, breathwork, astrology, dream interpretation, and more — into a single connected platform.

> **Critical Principle:** Inner Lab is NOT a collection of apps. It is a unified system.

### Key Characteristics

- One system built from multiple tools/modules — not separate standalone apps
- Each module serves a different aspect of inner growth (dialogue, movement, breathwork, astrology, dreams, etc.)
- The system learns from user engagement over time, personalizing the experience
- Modules share context and insights — what you learn in one informs the others
- Belief-inclusive: works across all spiritual traditions and none
- Sovereignty-focused: user data belongs to the user, never sold

---

## Section 3: How It Works (4-Step Framework)

The Inner Lab experience follows a consistent 4-step cycle designed to deepen self-awareness over time:

### Step 1: Observe

Check in with your mood, energy, and intention. Capture your current state without judgment. The system begins by meeting you where you are.

### Step 2: Interpret

The system reads your signals and surfaces relevant content. It helps you understand patterns — what keeps showing up, what's shifting, what needs attention.

### Step 3: Practice

Engage in a guided exercise tailored to your current state. This could be a conversation, a breathing session, a meditation, a journal prompt, or a movement practice. Take guided action.

### Step 4: Integrate

Reflect on what shifted. Help the system learn from your experience so future suggestions become more relevant. This is where growth compounds.

---

## Section 4: The Inner Lab Loop

Beyond the 4-step framework, Inner Lab operates on a continuous loop that deepens with every interaction:

```
Theme  →  Practice  →  Prompt  →  Action  →  Reflection  →  Insight
```

This cycle repeats and deepens over time as the system adapts to the user. Each pass through the loop builds on previous insights, creating a compounding effect where the system becomes increasingly personalized and effective.

---

## Section 4B: A Day With Inner Lab (Website Marketing Content)

How the daily experience is presented on innerlab.ai (Morning / Midday / Evening tabs). This is aspirational marketing — actual implementation details live in the platform-instructions docs:

| Step | What Happens |
|------|-------------|
| 1. 30-Second Check-in | Mood, energy, stress level, and your intention for the day |
| 2. Signals + Context | Transits, lunar phase, recent journal themes, and your personal goals |
| 3. Theme Interpretation | A plain-English summary of what this period is about for you |
| 4. 3-10 Min Practice | Breathwork, a short yoga flow, reflection exercise, or mantra |
| 5. One Real-World Action | A boundary to set, a conversation to have, or a focus step to take |
| 6. Reflection + Feedback | Rate accuracy and helpfulness -- the system learns from you |

---

## Section 4C: The Intelligence Layer (Website Marketing Content)

How the intelligence layer is presented on innerlab.ai. This is the marketing vision — actual collection schemas and API designs live in `platform-instructions-for-innerlab/CLAUDE.md`. Visualized on the website as concentric layers with the user at the center:

| Layer | Components |
|-------|-----------|
| **You** (center) | At the center of every layer |
| **Identity + State** | Identity Model, State Awareness |
| **Memory + Orchestrator** | Memory + Insights, Orchestrator |
| **Safety + Adaptive Learning** | Safety + Boundaries, Adaptive Learning |

### Where AI Fits (5 Capabilities)

Intelligence that enhances your practice -- never replaces your intuition:

1. **Guide Conversations** -- Personalized dialogue using your identity, recent entries, and current state
2. **Daily Briefing** -- Synthesizes signals into your Theme, Practice, Prompt, and Action
3. **Journal Intelligence** -- Pattern extraction, gentle reframes, and prompt generation from your entries
4. **Ritual Builder** -- Assembles rituals from a library based on your goals and time available
5. **Symbol Interpretation** -- Suggests meanings for dreams and synchronicities, learning your personal dictionary

> "AI is a tool in your toolkit -- you always stay in control."

---

## Section 5: The Lab Model

Inner Lab is positioned as a personal laboratory for self-discovery. The metaphor of a lab is intentional:

```
Input  →  Interpretation  →  Experiment  →  Outcome  →  Learning
```

- Users bring their own inputs (feelings, questions, states of mind)
- The system interprets and suggests relevant experiments (practices, conversations, reflections)
- Each experiment produces outcomes that generate learning
- Learning feeds back into the system, refining future suggestions
- There are no failures — only data points for growth

---

## Section 6: Complete Module Catalog

Inner Lab consists of 11 modules spanning multiple modalities of inner growth. Each module serves a distinct purpose while connecting to the broader system.

### Active Modules

#### 1. Conversations With God (CWG)

| Field | Value |
|-------|-------|
| **Purpose** | Reflect through dialogue |
| **Status** | Live |
| **URL** | conversationswithgod.ai |

**Description:** Explore meaning through guided AI conversations with 21 spiritual guides from across all major traditions. Available 24/7 in 36 languages. Includes guided meditations, spiritual journaling, consciousness mapping, and sacred text library.

**What it helps users do:** Process grief, find life purpose, explore spirituality, practice forgiveness, reduce anxiety through meaningful dialogue.

#### 2. FlowState

| Field | Value |
|-------|-------|
| **Purpose** | Practice with guidance |
| **Status** | Live |
| **URL** | yoga.magicbusstudios.com |

**Description:** Structured yoga, breathwork, and meditation sessions designed for your level and goals. Includes curated flows across categories (strength, flexibility, balance, relaxation), guided breathwork patterns with personal bests tracking, and meditation sessions. Features daily streaks, achievements, community-created flows, and an adaptive program system that adjusts to your progress.

**What it helps users do:** Build a consistent movement practice, improve flexibility and strength, connect body and mind, track progress with streaks and achievements, discover community-created flows.

**Key features:** Category-based flow library, breathwork sessions with timing and personal records, meditation library, daily streak tracking, achievement badges, community flows (user-created and shared), program enrollment (multi-week structured progressions), health condition and injury awareness (adapts recommendations).

### Coming Soon Modules

#### 3. BreathArc

**Tagline:** Guided breathing for grounding and clarity

**Description:** Breathwork practices calibrated to mood and energy levels. From box breathing for anxiety to energizing breath patterns for lethargy.

**What it helps users do:** Reduce stress, improve focus, regulate nervous system, build a daily breathwork habit.

#### 4. StarMap

**Tagline:** Explore cycles and patterns in your astrological chart

**Description:** Personal astrological chart analysis that connects planetary cycles to inner experiences. Not predictive — exploratory and reflective.

**What it helps users do:** Understand personal cycles, explore astrological archetypes, connect cosmic patterns to inner states.

#### 5. AstroCompass

**Tagline:** Discover how location influences your energy (astrocartography)

**Description:** Mapping tool that overlays astrological data with geography. Explore how different locations resonate with your chart.

**What it helps users do:** Plan travel with intention, understand place-based energy, explore relocation insights.

#### 6. Arcana

**Tagline:** Use archetypal tarot frameworks for reflection

**Description:** Tarot-inspired reflection tool using archetypal frameworks. Not fortune-telling — structured self-inquiry through symbolic imagery.

**What it helps users do:** Access subconscious patterns, use symbolism for reflection, explore decision-making from new angles.

#### 7. Archetypes

**Tagline:** Explore inner voices through Warrior, Sage, Healer, and other archetypes

**Description:** Jungian-inspired exploration of inner sub-personalities. Identify which archetypes are active and what they need.

**What it helps users do:** Understand inner conflicts, develop neglected parts of self, build psychological wholeness.

#### 8. DreamLens

**Tagline:** Decode dreams, track synchronicities, build personal symbol library

**Description:** Dream journaling with AI-powered interpretation. Build a personal symbol dictionary over time.

**What it helps users do:** Remember dreams more clearly, find recurring patterns, connect dream insights to waking life.

#### 9. Rituals

**Tagline:** Curated rituals for anxiety, confidence, grief, focus, and relationships

**Description:** Pre-built and customizable ritual sequences for specific life situations. Combines elements from multiple modalities.

**What it helps users do:** Navigate difficult emotions, mark transitions, build meaningful daily practices.

#### 10. InnerQuest

**Tagline:** Structured multi-day programs for shadow work, self-trust, and inner growth

**Description:** Guided multi-day and multi-week programs with daily practices, reflections, and milestones.

**What it helps users do:** Commit to deeper work, build sustained practice, achieve specific personal growth goals.

#### 11. Nexus

**Tagline:** Connect with practitioners and guided experiences (marketplace)

**Description:** Marketplace connecting users with verified practitioners — therapists, coaches, astrologers, healers.

**What it helps users do:** Find human guidance, book sessions with practitioners, complement digital tools with personal support.

---

## Section 6B: The Inner Lab Dashboard (innerlab.ai/dashboard)

The Inner Lab Dashboard is the unified hub for subscribers with Inner Lab All Access. It's the place where all module data comes together to reveal patterns you wouldn't see from any single module alone.

### Dashboard Features

| Feature | Description |
|---------|-------------|
| **Module Launcher** | Quick access to all active modules with status indicators (active, coming soon). One-click launch into any module. |
| **Daily Check-In** | Quick mood, energy, stress, and intention capture. Takes 30 seconds. Feeds into all modules for personalized recommendations. |
| **Activity Feed** | Cross-module timeline showing recent activity across all modules — conversations in CWG, sessions in FlowState, journal entries, achievements, check-ins. See your whole inner growth journey in one place. |
| **Consciousness Profile** | Unified view of your spiritual assessment, archetype, and orientation. Based on CWG's 12-archetype assessment, enriched by data from all modules over time. Historical snapshots show how your profile evolves. |
| **Memory Manager** | View AI-extracted insights about you from each module. Control which memories are shared across modules and which stay private. Full transparency and sovereignty over what the system knows. |
| **Daily Briefing** (future) | AI-synthesized summary: "Here's what we noticed across your modules today." Connects check-in data, conversation themes, practice patterns, and astrological transits into a personalized morning briefing. |
| **Cross-Module Insights** (future) | Pattern detection across modules. Example: "You've been exploring grief in CWG and doing calming breathwork in BreathArc — here's what we notice." Requires 2-3 active modules with enough data. |

### How Modules Connect (Cross-Module Intelligence)

The real power of Inner Lab is that modules talk to each other through shared data:

- **CWG conversation about anxiety** → BreathArc suggests a calming breathing pattern
- **FlowState yoga session focused on hip opening** → CWG's guides know this often surfaces stored emotions
- **StarMap shows a challenging transit** → Rituals suggests a relevant practice pack
- **DreamLens captures a recurring dream symbol** → Archetypes connects it to an active inner voice
- **Check-in shows low energy for 3 days** → Dashboard flags the pattern and suggests actions

This cross-module intelligence is opt-in — users control what's shared. But for those who opt in, it creates a compounding effect where each module becomes more valuable because of the others.

### User Memory Privacy Model

Inner Lab handles deeply personal data. The privacy model is built on explicit user consent:

- Each module's AI creates memories in its own namespace (private by default)
- Users can explicitly share specific memories across all Inner Lab modules
- Shared memories are visible to all modules; private memories are visible only to the source module
- Users can revoke sharing at any time
- Full data export and deletion available (GDPR/CCPA compliant)

---

## Section 7: Key Differentiators

- **Unified system, not separate apps** — modules share context and learn together. Insights from one module inform suggestions in another.
- **Belief-inclusive** — works across all spiritual traditions and none. No dogma, no prescriptions. Users find what resonates.
- **Sovereignty-focused** — user data belongs to the user. Private, encrypted, never sold. Growth should be personal.
- **Adaptive** — the system learns from user engagement and personalizes over time. Each interaction makes the next one more relevant.
- **Modular** — users engage with what resonates and skip the rest. No forced paths, no required sequences.
- **Multi-modal** — combines dialogue, movement, breathwork, astrology, dreams, ritual, and more. No other platform spans this range.
- **Premium design** — dark aesthetic, glass UI, particle effects. The experience feels as considered as the content.

---

## Section 7B: Privacy & Data Commitments (Website Marketing Content)

Privacy promises displayed on innerlab.ai. Implementation details (encryption, GDPR, data export) are in the platform-instructions docs:

1. **Encrypted by Default** -- AES-256, TLS. Journal entries never stored in plain text.
2. **Privacy-First AI** -- Minimal context sent to AI, no persistent profiles shared with AI providers.
3. **Your Data, Your Choice** -- Export or delete everything anytime. JSON or CSV download.
4. **Minimal Collection** -- No analytics trackers, no ad profiles, no third-party data sharing.
5. **No Third-Party Selling** -- Data never sold, shared, or used to train external AI models.
6. **Transparent Practices** -- Privacy policy in plain language, not legalese.

---

## Section 8: User Journey

| Stage | Description |
|-------|-------------|
| **1. Discovery** | User finds MagicBusStudios studio site or Inner Lab directly through search, social media, or word of mouth. First impression is the premium design and clear messaging. |
| **2. Exploration** | User browses modules, reads about the system, understands how Inner Lab works as a unified platform. The 4-step framework and module catalog communicate scope and depth. |
| **3. First Engagement** | User tries CWG (free, no account needed — 5 messages per day) or FlowState. This is the lowest-friction entry point with highest value demonstration. |
| **4. Deepening** | User signs up, explores more features within active modules. Begins to see the system adapt. Engages with journaling, meditation, or assessments. |
| **5. Commitment** | User joins waitlist for coming-soon modules, subscribes to studio updates, considers premium subscription. They see Inner Lab as part of their routine. |
| **6. Advocacy** | User shares with friends, provides feedback, participates in beta testing for new modules. They become part of the community. |

---

## Section 9: Marketing Hooks

Approved messaging themes and taglines for marketing use:

- "Platform for Inner Growth" (website hero H1)
- "A product ecosystem for reflection, practice, and holistic well-being" (website hero subheadline)
- "A unified system for inner growth"
- "Observe. Reflect. Grow."
- "Technology that serves the soul"
- "Find what resonates. Leave the rest."
- "Your growth. Your pace. Your path."
- "One system. Many tools. Your journey."
- "Built for seekers, not followers"
- "11 modules. One connected experience."
- "A personal laboratory for self-discovery"

### Messaging Reminders

- Always frame as ONE system, never as separate apps
- Use branded module names (FlowState, BreathArc — never generic descriptions)
- Avoid "calm" in marketing copy
- Never use "Operating System" to describe Inner Lab
- Tone: grounded, inclusive, clear — not preachy, not clinical

---

## Section 10B: Design Language (innerlab.ai)

### Colors
- Background: near-black (`rgb(5, 5, 5)`)
- Body text: off-white zinc-100 (`rgb(244, 244, 245)`)
- Muted text: zinc-400 (`rgb(161, 161, 170)`)
- Primary accent: teal-500 (`rgb(20, 184, 166)`)
- Secondary accent: sky-500 (`rgb(14, 165, 233)`)
- Accent gradient: teal-500 to sky-500 (used on italic hero text and highlights)
- Card backgrounds: dark at 30-50% opacity with borders at 6-10% white

### Typography
- Body: **DM Sans** (Variable), 14-16px
- Headings: **Space Grotesk** (Variable), bold 700, 36-72px
- Italic accent: **Instrument Serif**, italic, with gradient text effect

### UI Patterns
- Dark theme throughout (no light mode)
- Glass-morphism cards (semi-transparent bg + subtle borders)
- Horizontal scrolling marquee tickers with dot separators
- Aurora drift animations (4 layers) for background ambiance
- Concentric ring diagrams for system architecture
- Tab-based filtering (All/Active/Coming Soon)
- Morning/Midday/Evening time-of-day tabs
- Numbered step indicators ("01", "02" format)
- Green dot status badge for "live" modules

### Current Website State (Pre-Platform)
- `/` — Homepage (hero, how it works, daily flow, intelligence layer, AI capabilities, privacy, module stats)
- `/modules` — Module catalog with filter tabs (All/Active/Coming Soon)
- `/about` — Redirects to magicbusstudios.com
- `/waitlist`, `/subscribe`, `/contact` — Form pages
- No login/dashboard UI yet (comes with platform build)
- No pricing visible (comes with platform build)
- No individual module detail pages
