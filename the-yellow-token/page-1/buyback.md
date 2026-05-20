# buyback

We use **85%** of every trading fee on Yellow to buy back $YELLOW on the open market.

That creates buying pressure that scales with platform activity. More trades → more fees → more buyback.

## How it works

Two steps, no magic:

1. Fees in non-USDT currencies are sold into USDT on Yellow's own spot markets.
2. The USDT is used to buy $YELLOW on the open order book.

Bought $YELLOW goes into a platform-owned holding account.

## How often

Buybacks run on a regular cadence.

## On-chain proof

Every buyback run is recorded end to end — the fees collected, the consolidation trades, the $YELLOW purchases, and the destination holding account. The full chain from a user's trading fee to the resulting $YELLOW purchase is auditable.

## Impact

* every fee paid on Yellow contributes to buying $YELLOW
* buyback orders execute on Yellow's own books, contributing to volume
* buyback size scales 1-to-1 with platform fee revenue

## Related

* [Fee Split](/broken/pages/5f83c56dc4cb42e1fa30db2a4b50914e641b8c37) — where the other 15% goes
* [Fees](/broken/pages/cfb7591e77e25a1b8558bcd2efb3a7c705ffaf21) — what generates the buyback in the first place
