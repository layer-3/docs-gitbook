---
description: >-
  How perpetual contracts work on Yellow, including funding, collateral in
  stablecoins, and the insurance fund.
hidden: true
---

# Perpetuals

Perpetual contracts are derivative instruments that let you take long/short exposure to an asset without an expiry date. You do not receive the underlying asset.

### Price tracking (funding mechanism)

Perpetual contract prices are kept close to the underlying spot/index price through funding.

Funding is a periodic payment exchanged between open position holders:

* If funding is positive: longs pay shorts
* If funding is negative: shorts pay longs

The purpose of funding is price alignment between the perpetual market and the reference price.

Funding is not the same as a trading fee.

_Screenshot: Perp market header showing funding rate + next funding time_

### Collateral and PnL currency

Perpetual positions are collateralized in a stablecoin (for example, USDC).

This means:

* margin is posted in the stablecoin
* profits and losses (PnL) are tracked and settled in the stablecoin
* account equity is measured in the collateral currency

**Screenshot:** Position panel showing margin, equity, and unrealized PnL in USDC

### Insurance fund

A portion of fees can be allocated to an insurance fund.

The insurance fund is used to absorb losses when positions are liquidated below bankruptcy price, with the goal of reducing the likelihood and impact of auto-deleveraging (ADL) events.

Yellow should document:

* how the insurance fund is funded
* when it is used
* how ADL is handled if insurance is insufficient

**Screenshot:** Risk/insurance info panel or help tooltip
