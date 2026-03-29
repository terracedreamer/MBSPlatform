# The Arcade

**by Magic Bus Studios**

*Marketing Brief & Website Content Guide — March 2026*

> **Last Updated**: March 27, 2026. Platform decisions integrated, pricing marked as TBD, auth methods updated.
> **Note**: Arcade product information has been absorbed into `MagicBusStudios_Brand_And_Company.md` (the master doc). This file is kept as detailed reference for Arcade-specific marketing campaigns and content strategy. The master doc is the source of truth for product descriptions and pricing.

## Platform Context (SSO Migration Pending — Phase 5)
- The Arcade will use the centralized MBS Platform for auth and billing (no per-game payment handling)
- **Four auth methods** via MBS Platform: Google SSO, Email/Password (with optional 2FA/TOTP), Nostr identity, LNURL-Auth (Lightning wallet)
- Login redirects to magicbusstudios.com/auth/login with MBS branding (not Inner Lab)
- Arcade games do NOT share data between each other — SSO only via MBS Platform
- Each game keeps its own database (no migration to shared DB)
- No separate Arcade marketing folder — Arcade is marketed under MBS brand
- Tiered access: MBS All Access ($29.99/mo or $249.99/yr — includes everything). Individual game pricing TBD.

---

## 1. What is The Arcade?

The Arcade is a collection of multiplayer party games built by Magic Bus Studios. Social games designed to bring people together — built around skill, creativity, and quick thinking.

The Arcade sits alongside Inner Lab and Studio Works as one of three product lines under Magic Bus Studios.

## 2. The Games

| Game | Tagline | Description | URL |
|------|---------|-------------|-----|
| Broken Chain | Break free together | Break free through strategy and deception. A game of alliances, betrayal, and escape. | brokenchain.magicbusstudios.com |
| MindHacker | Outsmart your friends | Outsmart your friends with psychological strategy. Read minds, bluff, and decode hidden intentions. | mindhacker.magicbusstudios.com |
| Trivia Roast | Knowledge meets comedy | Answer trivia, roast your rivals, and climb the leaderboard. Knowledge meets comedy. | triviaroast.magicbusstudios.com |
| Whispering House | Trust no one | A social deduction game of secrets and suspicion. Share whispers, form alliances, and uncover the truth. | whisperinghouse.magicbusstudios.com |
| Fake Artist | One player is faking it | One player doesn't know the prompt — but has to fake it. Spot the impostor before it's too late. | fakeartist.magicbusstudios.com |

**Known issue:** Trivia Roast subdomain is misspelled as `trivaroast.magicbusstudios.com` (missing "i") in the live deployment. Needs DNS fix.

## 3. Target Audience

- Friend groups looking for online party games
- Remote teams wanting fun team-building activities
- Couples and families wanting shared entertainment
- Board game enthusiasts seeking digital alternatives
- Streamers and content creators

Age range: 16-40. Tech-savvy. Browser-based gaming.

## 4. Positioning & Tone

**Positioning:** Social games built around skill, creativity, and quick thinking.

**Tone:** Playful, energetic, competitive but inclusive. Game night with friends, not esports.

- No downloads — play instantly in your browser
- Works on any device — phone, tablet, laptop
- Designed for groups — best with 3-8 players
- Quick rounds — 5-15 minute sessions

## 5. Pricing Model

**Current status: All games are FREE.** Pricing below is planned/aspirational and has not been finalized or implemented.

| Option | Price (PLANNED) | What You Get |
|--------|----------------|-------------|
| Free Tier | Free | Limited daily play time (30 min/day) |
| Game Pass | $4.99 one-time (TBD) | Unlimited access to one game forever |
| All Access Monthly | $14.99/month (TBD) | All games unlimited |
| All Access Yearly | $119.99/year (TBD) | All games unlimited (save 33%) |
| 10hr Time Bank | $9.99 one-time (TBD) | 10 hours across any game |
| 25hr Time Bank | $19.99 one-time (TBD) | 25 hours across any game |

## 6. Website Content (magicbusstudios.com/arcade)

Current state: 5 games with links. Future additions (after platform is built):

- Login/signup button (Google SSO, Email/Password, Nostr, LNURL-Auth via MBS Platform)
- My Games section for logged-in users
- Pricing cards for passes and subscriptions
- Promo code input
- User profile and subscription management

**DO NOT add these to the website until the platform middleware is built.**

## 7. Marketing Campaigns

### 7.1 Launch

- Email blast to MBS subscribers
- Social media: gameplay clips for each game
- Launch promo: `ARCADE50` — 50% off All Access first month
- Referral: invite a friend, both get 2 free hours

### 7.2 Ongoing

- New game launch emails
- Seasonal events and tournaments
- Win-back emails (14+ days inactive)
- Weekly newsletter: top plays, tips, upcoming features

### 7.3 Content Ideas

- Blog: *Best Party Games for Remote Teams*, *How to Host a Virtual Game Night*
- Short-form video: 30s gameplay clips, reaction compilations

## 8. Differentiators

- Browser-based — no downloads
- Cross-platform — any device with a browser
- Quick rounds — 5-15 minutes
- Social-first — every game built for groups
- Part of MBS ecosystem — meaningful tools + fun games

## 9. Brand Connection

The Arcade is one of three MBS product lines. While Inner Lab focuses on inner growth and Studio Works provides practical tools, The Arcade represents the playful, social side of the studio.

The Arcade demonstrates MBS builds diverse products — from mindfulness to party games. The common thread: bringing people together and creating meaningful experiences.
