---
description: >-
  How perpetual contracts work on Yellow, including funding, collateral in
  stablecoins, and the insurance fund.
hidden: true
---

# Perpetuals

Perpetual contracts are derivative instruments that let you take long or short exposure to an asset without an expiry date. You do not receive the underlying asset.

### Price tracking (funding mechanism)

Perpetual contract prices are kept close to the underlying spot/index price through funding.

Funding is a periodic payment exchanged between open position holders:

* if funding is positive: longs pay shorts
* if funding is negative: shorts pay longs

The purpose of funding is price alignment between the perpetual market and the reference price.

Funding is not a trading fee. It is a payment between traders, not to the platform.

Funding rate history is available per market.

_Screenshot: Perp market header showing funding rate + next funding time_

### Collateral and PnL currency

Perpetual positions are collateralized in **USDC**.

This means:

* margin is posted in USDC
* profits and losses (PnL) are tracked and settled in USDC
* account equity is measured in USDC

_Screenshot: Position panel showing margin, equity, and unrealized PnL in USDC_

### Margin modes

Yellow supports both **cross margin** and **isolated margin**, configurable **per market**. The same account can run BTC perps in isolated mode and ETH perps in cross mode at the same time.

* **Cross margin** — your full perp account equity backs all positions in that account
* **Isolated margin** — a specific amount of margin is allocated to a single position; losses on that position can only consume the allocated margin

In isolated mode you can add or reduce the allocated margin on an open position, which moves the liquidation price.

### Position modes

Two position modes are available, configurable per market:

* **One-way mode** — a market has a single net position (long or short)
* **Hedge mode** — a market can hold simultaneous long and short positions, each tracked independently

Hedge mode combined with isolated margin keeps the long and short legs' allocated margin fully separate.

### Leverage and risk tiers

Each perpetual market uses a **tiered margin** system. Position size is divided into tiers, and each tier defines:

* maximum leverage
* initial margin rate
* maintenance margin rate
* maximum position size
* minimum maintenance margin

Maximum leverage is highest on the smallest tier and decreases as position size grows. As your position size crosses tier boundaries, the maintenance margin rate and position limit update automatically.

Example: BTCUSDT-PERP tiers (illustrative — current values are shown in the trading-parameters page in the app):

| Tier | Max Leverage | Initial Margin Rate | Maintenance Margin Rate | Max Position Size (BTC) |
| ---- | ------------ | ------------------- | ----------------------- | ----------------------- |
| 1    | 100x         | 1.00%               | 0.5%                    | 5                       |
| 2    | 50x          | 2.00%               | 1.0%                    | 10                      |
| 3    | 25x          | 4.00%               | 3.0%                    | 30                      |
| 4    | 20x          | 5.00%               | 4.0%                    | 50                      |
| 5    | 10x          | 10.00%              | 5.0%                    | 75                      |
| 6    | 5x           | 20.00%              | 6.0%                    | 100                     |

Other markets follow the same tier model with their own parameters.

_Screenshot: Trading parameters page showing leverage and margin tiers_

### Order types specific to perps

In addition to the standard order types (see [Order Types](/broken/pages/2ec36c52e57665fe70ef57157f3b138ac726429e)), perps support:

* **Post-only** — the order is only accepted if it would rest in the order book as a maker order
* **Reduce-only** — the order can only reduce, not increase, the size of an open position (one-way mode)

Take Profit and Stop Loss are placed manually through the order form.

### Liquidation

When a position's margin falls below the maintenance margin requirement, it is liquidated. Liquidation runs in stages and tries to preserve as much of the position as possible before closing it fully.

**Stage 0 — partial position reduction.** Instead of closing the full position immediately, the system first reduces the position to the upper limit of a lower risk tier. This lowers the maintenance margin requirement and may bring the position back to a healthy state. The reduction is filled at the bankruptcy price against the insurance fund.

* mild excess risk (margin ratio < 110%): reduce by 1 tier (conservative)
* severe excess risk (margin ratio ≥ 110%): reduce by 2 tiers (aggressive)

If the position is already in the smallest tier, there is no partial step — liquidation goes straight to full closure.

**Stage 1 — full liquidation.** If partial reduction is not sufficient, the remaining position is closed.

**Stage 2 — auto-deleveraging (ADL).** If the insurance fund cannot absorb the position and an IOC order finds no market depth at the bankruptcy price, ADL closes the position against opposing counterparties.

Open orders on a liquidated position are cancelled automatically. Liquidation fills are tagged in trade history as `LIQUIDATION` or `ADL`.

_Screenshot: Position card showing liquidation price, margin ratio, and bankruptcy price_

### Insurance fund

A portion of fees can be allocated to the insurance fund. The insurance fund acts as the counterparty for partial-liquidation fills at the bankruptcy price, reducing the likelihood and impact of ADL events.

Insurance fund activity is published as a history feed.

_Screenshot: Insurance fund history feed_

### Fees

Perp trading fees follow the account's VIP level. See [Fees](/broken/pages/588bff933a91fa777fc02205a244994553b08fab) for the full schedule, $YELLOW discount mechanics, and rebate tiers.

Funding payments are separate from trading fees.

### Position and funding history

The following histories are available per account:

* **Position history** — closed positions with entry, exit, allocated margin, and realized PnL
* **Funding rate history** — funding rate and payment per market over time
* **Trade history** — all fills, including those tagged `LIQUIDATION` or `ADL`

_Screenshot: Position history table_

### What This Means in Practice

* you trade perps with USDC margin, and PnL settles in USDC continuously against the mark price
* leverage and maintenance margin requirements change as your position size moves between risk tiers
* if margin runs low, the system tries partial reduction before full liquidation, and only uses ADL as a last resort
* use isolated margin to cap downside on a single position; use cross margin to share equity across positions
