---
hidden: true
---

# Vip Tiers

Your trading fees on Yellow depend on your **VIP level**. Every account starts at Base and moves up through six VIP levels as your activity or $YELLOW holdings grow.

Lower fees on every trade. Maker rebates from VIP 5 on perpetuals.

## Three Ways to Level Up

There are three independent paths to any VIP level. **Meet any one** and you get the tier.

| Tier  | 30d Spot Volume | 30d Perp Volume | $YELLOW Held |
| ----- | --------------- | --------------- | ------------ |
| Base  | < $100K         | < $1M           | < 20,000     |
| VIP 1 | ≥ $100K         | ≥ $1M           | ≥ 20,000     |
| VIP 2 | ≥ $500K         | ≥ $5M           | ≥ 100,000    |
| VIP 3 | ≥ $2.5M         | ≥ $25M          | ≥ 350,000    |
| VIP 4 | ≥ $10M          | ≥ $100M         | ≥ 2,000,000  |
| VIP 5 | ≥ $50M          | ≥ $500M         | ≥ 5,000,000  |
| VIP 6 | ≥ $200M         | ≥ $2B           | ≥ 20,000,000 |

Your VIP level is the **highest tier you qualify for across all three tracks**. One track is enough.

## Fees by VIP Level

The same VIP level applies to both spot and perpetuals. Rates differ by product.

### Spot

| Tier  | Maker  | Taker  |
| ----- | ------ | ------ |
| Base  | 0.080% | 0.100% |
| VIP 1 | 0.060% | 0.080% |
| VIP 2 | 0.050% | 0.070% |
| VIP 3 | 0.040% | 0.055% |
| VIP 4 | 0.030% | 0.045% |
| VIP 5 | 0.020% | 0.035% |
| VIP 6 | 0.015% | 0.030% |

### Perpetuals

| Tier  | Maker                | Taker  |
| ----- | -------------------- | ------ |
| Base  | 0.010%               | 0.035% |
| VIP 1 | 0.008%               | 0.030% |
| VIP 2 | 0.005%               | 0.025% |
| VIP 3 | 0.002%               | 0.020% |
| VIP 4 | 0.000%               | 0.015% |
| VIP 5 | **−0.005%** (rebate) | 0.012% |
| VIP 6 | **−0.010%** (rebate) | 0.010% |

A negative maker fee means Yellow **pays you** for providing liquidity. Rebates are credited in the quote currency of the trading pair, settled per trade.

## How Each Track Works

### Spot volume

Rolling 30-day USD volume on spot markets. Updated in real time. Your tier can drop when older trades roll out of the 30-day window.

### Perp volume

Same as spot, but on perpetual markets. Tracked separately.

The perp thresholds are roughly 10× the spot thresholds. This reflects the leverage built into perp trading — $1M in perp notional is closer in capital terms to $100K in spot.

### $YELLOW holding

The $YELLOW balance held in your Yellow trading account counts toward this track.

What counts:

* free $YELLOW in your Yellow trading account

What does **not** count:

* staked $YELLOW
* locked $YELLOW
* $YELLOW used as perp collateral
* $YELLOW held in external wallets

Your tier is evaluated against your balance at the time of each trade.

## Worked Examples

**Retail spot trader.** Holds 22,000 $YELLOW. Did $50K spot, no perps. Spot track → Base. Perp track → Base. Holding track → VIP 1. **You're VIP 1.** Spot 0.060% / 0.080%, perp 0.008% / 0.030%.

**Active perp trader.** No $YELLOW. $200K spot volume, $30M perp volume. Spot → VIP 2. Perp → VIP 3. Holding → Base. **You're VIP 3.** Spot 0.040% / 0.055%, perp 0.002% / 0.020%.

**Market maker.** $600M perp volume, no spot, no $YELLOW. Perp track alone → VIP 5. **You're VIP 5.** Perp maker fees are now a rebate: you earn 0.005% on every maker fill.

## When Does My Tier Update?

* **Volume tracks:** continuously, as trades fill and as old trades age out of the 30-day window.
* **$YELLOW holding:** evaluated at each trade against your current balance. Deposit more $YELLOW mid-session, and your next trade is at the new tier.
* **Downgrades:** automatic. If your 30-day volume drops below a threshold or you withdraw $YELLOW, your tier moves down at the next trade.

## Where to Check Your Tier

Your current VIP level, the track that qualified you, and your effective maker/taker rates are visible in your account fee summary in the app.

The fee you actually paid is also visible on every fill in your trade history.

## Related Pages

* [Fees](fees.md) — how trading fees are calculated and where to check them
* [Spot](markets/spot.md) — spot trading basics
* [Perpetuals](markets/perpetuals.md) — perp trading basics
* [Referral Program](../getting-started/referral-program.md) — partners can grant a starting VIP tier to invited users
* [The $YELLOW Token](/broken/pages/J28UKSzcT5EIn6iV6Sgw) — token utility, including the holding-based fee discount
