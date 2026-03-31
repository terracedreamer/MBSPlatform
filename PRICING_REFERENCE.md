# MBS Pricing Reference

**Last Updated**: March 31, 2026 (Session 12)

## Product Pricing (25% annual discount across all tiers)

| Product | Category | Monthly | Annual | Stripe Env (Monthly) | Stripe Env (Annual) | Status |
|---------|----------|---------|--------|---------------------|---------------------|--------|
| FlowState | Inner Lab | $5 | $45 | `STRIPE_YOGA_MONTHLY_PRICE_ID` | `STRIPE_YOGA_ANNUAL_PRICE_ID` | Not created |
| CWG | Inner Lab | $15 | $135 | `STRIPE_CWG_MONTHLY_PRICE_ID` | `STRIPE_CWG_ANNUAL_PRICE_ID` | Not created |
| Studio Works All Access | Studio Works | $10 | $90 | `STRIPE_SW_MONTHLY_PRICE_ID` | `STRIPE_SW_ANNUAL_PRICE_ID` | Not created |
| Arcade All Access | Arcade | $10 | $90 | `STRIPE_ARCADE_MONTHLY_PRICE_ID` | `STRIPE_ARCADE_ANNUAL_PRICE_ID` | Not created |
| Inner Lab All Access | Inner Lab | $20 | $180 | `STRIPE_IL_MONTHLY_PRICE_ID` | `STRIPE_IL_ANNUAL_PRICE_ID` | Not created |
| MBS Everything Bundle | All | $30 | $270 | `STRIPE_MBS_MONTHLY_PRICE_ID` | `STRIPE_MBS_ANNUAL_PRICE_ID` | Not created |

**Total: 6 products, 12 prices to create in Stripe Dashboard.**

## Lightning / Bitcoin (via BTCPay)

| Product | Monthly (sats) | Yearly (sats) | Status |
|---------|---------------|---------------|--------|
| CWG | 21,000 | 126,000 | BTCPay API key 403 -- needs regen |
| Other products | TBD | TBD | Not configured |

## Free Tier (current state)

All products are currently free. Premium gating is wired but not enforced (`effectivePremium = true` in every app). One-line toggle per app to enable gating.

## Setup Steps

1. Create 6 products in Stripe Dashboard (test mode)
2. For each product, create Monthly + Annual price
3. Copy 12 price IDs (`price_xxx` format)
4. Add 12 env vars to Coolify MBS Backend service
5. Redeploy MBS Backend
6. Test checkout flow at `magicbusstudios.com/billing`

**BTCPay**: Separately, regenerate the API key with full store permissions in the BTCPay dashboard.

## Pricing History

| Date | Change |
|------|--------|
| Session 9 (March 30) | Finalized all 6 products with 25% annual discount |
| Session 8 (March 29) | CWG pricing decided: $9.99/mo (later changed to $15) |
| Session 1 (March 25) | Three-tier subscription model designed |
