# CHANGELOG — MBS Platform

## March 24, 2026 — Session 1: Architecture Planning

### What happened
- Full architecture discussion across MBS Platform + Inner Lab ecosystem
- Read and analyzed all reference documents (Technical Architecture, Product Briefs, ChatGPT architecture docs, Inner Lab Blueprint)
- Reviewed live innerlab.ai website (marketing site + modules page)

### Decisions made
1. **Three-layer architecture**: MBS Platform (SSO/billing) → Inner Lab ecosystem (shared DB) → Standalone products (own DBs)
2. **Inner Lab modules = separate containers, shared database** (`inner_lab`). Not one monolith — each module deploys independently but all connect to same DB.
3. **Fresh `inner_lab` database** — don't reuse existing `conversations_with_god` DB
4. **User identity ONLY in `mbs_platform`** — Inner Lab stores only spiritual/consciousness data
5. **B+C hybrid frontend**: modules work standalone at own domains + unified Inner Lab dashboard for All Access subscribers
6. **Tiered experience**: product_pass (single module) vs category_access (all Inner Lab + unified features)
7. **Inner Lab Core container built later** — when 2-3 modules exist
8. **Arcade & Studio Works stay fully independent** — SSO only through MBS Platform
9. **Admin = simple is_admin flag** for now

### No code written
- Project remains spec-only
- Next step: inspect CWG MongoDB database to understand existing data
