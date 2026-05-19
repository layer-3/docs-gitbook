# Buyback

Yellow runs a continuous buyback of the $YELLOW token funded by trading fees. This page explains how it works.

For the full fee allocation, see [Fee Split](fee-split.md). For current fee rates, see [Fees](fees.md).

### Source of Funds

The buyback is funded by **85% of every trading fee** collected on Yellow — across both spot and perpetual markets.

Trading fees on Yellow can be collected in different currencies depending on the market (for example, USDT for a BTC/USDT trade, or ETH for a USDT/ETH market). The buyback engine handles all of these.

### How a Buyback Run Works

{% stepper %}
{% step %}
### Phase 1 — Consolidation

Fees in the buyback bucket sit in their original currencies. The engine sells non-quote currencies into the configured buyback quote currency through standard spot orders on Yellow.

* small amounts may be aggregated across runs before consolidation, depending on market liquidity
* if a fee currency has no consolidation market, the balance is held until one is available
{% endstep %}

{% step %}
### Phase 2 — Purchase

The consolidated quote balance is used to buy $YELLOW on Yellow's spot order book.

Both phases use the same order types as any other trader. Buyback orders match against the open book — there is no separate matching engine or private pool.
{% endstep %}
{% endstepper %}

### Where Bought $YELLOW Goes

$YELLOW acquired through buyback is moved into a platform-owned **holding account**.

The holding account is custody only — it does not trade. Subsequent treasury actions (for example: long-term hold, distribution, burn) are governed separately by Yellow's treasury policy and are not part of the buyback engine itself.

### Schedule

The buyback runs on a periodic cadence. Each run processes the buyback bucket as it stood at the start of the run; fees collected during a run flow into the next one.

Specific cadence and trigger thresholds are operational parameters and may change over time.

### Auditability

Every buyback run is recorded end to end:

* the opening balance of the buyback bucket per currency
* the consolidation trades placed (with order IDs and fills)
* the $YELLOW purchase orders and fills
* the transfer of acquired $YELLOW into the holding account
* the closing balance and any residuals carried into the next run

The chain from an individual user's trading fee to the resulting $YELLOW purchase can be followed end to end through this record.

### Effect on the Market

Buyback orders interact with the order book like any other order. They contribute to volume on the markets they touch — both the consolidation markets (for example, ETH/USDT) and the YELLOW/quote market.

Because the buyback size scales directly with platform fee revenue, the buyback flow grows with trading activity on Yellow.

### What This Means in Practice

* every fee paid on Yellow contributes to a buyback of $YELLOW — 85% by policy
* the buyback uses ordinary spot orders on Yellow's own order book
* fees in any currency are first consolidated into the buyback quote, then used to buy $YELLOW
* acquired $YELLOW sits in a platform holding account; what happens next is a separate treasury decision
* buyback volume scales with platform activity

### Related Pages

* [Fee Split](fee-split.md) — the 15 / 85 allocation policy
* [Fees](fees.md) — current trading fee rates and VIP tiers
* [The $YELLOW Token](/broken/pages/J28UKSzcT5EIn6iV6Sgw) — token utility and supply mechanics
