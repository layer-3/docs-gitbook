# Fee Split

Yellow charges trading fees when orders are filled (see [Fees](fees.md) for current rates and VIP tiers). This page explains what happens to those fees after they are collected.

### Where Fees Go

Every collected trading fee is automatically split into two buckets:

* **15% — Platform operations.** Funds ongoing platform costs.
* **85% — $YELLOW buyback.** Used to buy $YELLOW on the open market.

The split is applied to every fee, on every trade, in both spot and perpetual markets.

### How the Buyback Works

The buyback share is held in a dedicated platform account and processed periodically.

The process runs in two phases:

1. **Consolidation.** Fees collected in non-quote currencies (for example, ETH from an ETH/USDT trade) are sold into the buyback quote currency on Yellow's own spot markets.
2. **Purchase.** The consolidated quote balance is used to buy $YELLOW on the open market through standard spot orders.

Acquired $YELLOW moves to a platform-owned holding account.

The buyback runs on a regular cadence; specific timing is operational and may change.

### Why a Buyback

The buyback creates a continuous demand signal for $YELLOW that scales with platform activity. Higher trading volume means more fees, which means a larger buyback.

This is a direct economic link between using Yellow and the $YELLOW token, independent of any speculative activity.

### Auditability

Each step in the flow is recorded:

* the fee collected on the original trade
* the split allocation into operations and buyback buckets
* the consolidation trades that convert fees into the quote currency
* the $YELLOW purchase orders

The full chain from a single trade fee to the resulting $YELLOW purchase is preserved end to end.

### Extensibility

The split mechanism is designed to support additional buckets without redesign. A future split — for example, adding a referral reward bucket — is a policy change rather than a platform change.

The current policy is 15% operations / 85% buyback. Any future change to the policy will be announced before it takes effect.

### What This Means in Practice

* every trading fee you pay on Yellow is split 15 / 85 — operations vs. $YELLOW buyback
* the buyback share is converted into $YELLOW through normal market orders on Yellow's spot books
* the higher the platform's trading volume, the larger the buyback flow into $YELLOW
* the policy can be extended in the future (for example, to include referral rewards) without changing the underlying mechanism

### Related Pages

* [Fees](fees.md) — current fee rates, VIP tiers, and $YELLOW discounts
* [The $YELLOW Token](/broken/pages/J28UKSzcT5EIn6iV6Sgw) — token utility and supply mechanics
