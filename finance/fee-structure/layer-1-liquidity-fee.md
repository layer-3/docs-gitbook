# Layer 1 - Liquidity Fee

The Liquidity Fee is agreed upon between the brokers when opening a channel. The broker acting as a market maker will receive the entirety (100%) of the fees paid by the broker acting as a market taker.

The Liquidity Fee is volume-based, calculated at position opening in % of the receivable currency of the market taker and added to, or subtracted from, the market taker's liability.

**Liquidity Fee = Trade Nominal in USDT x Broker Fee (%)**

Liquidity Fees are due on settlement, directly charged to the market taker's stablecoin wallet, and transferred to the market maker.

There is a minimum Liquidity Fee (Floor) to incentivize market making over market taking. Market Makers can set a higher fee for the use of their liquidity but can not go below the floor.

Defined by:       Brokers when they establish a trading channel\
Beneficiary:      Market Making Broker\
Payment Ccy:   Stablecoin\
Periodicity:       Calculated on every trade and charged on the settlement at 0.00 am UTC

| Broker Tier\* |           | Liquidity Fee (Floor) |
| ------------- | --------- | --------------------- |
| 4 channels    | 250.000   | 0.010%                |
| 8 channels    | 500.000   | 0.008%                |
| 16 channels   | 1.000.000 | 0.004%                |
| 32 channels   | 2.000.000 | 0.002%                |
| 64 channels   | 4.000.000 | 0.000%                |

_\*The applicable Broker Tier is defined at the beginning of each clearing cycle. Brokers can not move tiers during a clearing cycle._
