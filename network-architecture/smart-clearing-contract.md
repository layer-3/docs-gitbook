# Smart Clearing Protocol

### Network access

To join the Yellow Network, the broker needs to stake a minimum amount of 250,000 $YELLOW tokens. This allows him to create 4 peer-to-peer trading channels and access the liquidity pool of those 4 peers.

Sample bellow:&#x20;

| $YELLOW token staked | Opened state channels |
| -------------------- | --------------------- |
| 250,000              | 4                     |
| 500,000              | 8                     |
| 1,000,000            | 16                    |

To unstake his funds a broker needs to close open trading channels, request the funds withdrawals and **wait for 30 days** for funds to be available.

### Collateral

Before opening a trading channel, brokers agree on a collateral amount to deposit on the channel. Those funds are used to reduce the risks of the trades between the two brokers, it might be used in case of dispute to compensate a good faith broker against a malicious one.

#### Trading liabilities

The state of the trading channel contains the liabilities that one broker owes to the other.

Considering the following trades:

* Broker A buys 1 BTC for 50,000$ to the Broker B
* Broker B buys 10 ETH for 30,000$ to the Broker A

Liabilities will be the following:

| Currency | Broker A    | Broker B |
| -------- | ----------- | -------- |
| ETH      | 10 ETH      |          |
| BTC      |             | 1 BTC    |
| USDC     | 20,000 USDC |          |

Considering the price of those assets didn't move until now the estimated value in USD would be the following:



| Currency  | Broker A    | Broker B    |
| --------- | ----------- | ----------- |
| ETH       | $30,000     |             |
| BTC       |             | $50,000     |
| USDC      | $20,000     |             |
| **Total** | **$50,000** | **$50,000** |

In this situation liabilities are balanced.

Now let's consider BTC price raises by 5% to $52,500 and ETH raises by 10% to $3,300, the estimated liabilities value in USD would become:



| Currency  | Broker A    | Broker B    |
| --------- | ----------- | ----------- |
| ETH       | $33,000     |             |
| BTC       |             | $52,500     |
| USDC      | $20,000     |             |
| **Total** | **$53,000** | **$52,500** |

In this situation liabilities are unbalanced, Broker A owes 500$ more of asset to Broker B. As long as the collateral locked by both brokers is higher to this amount, the Broker B position is not at any risk.

In the last example we can see that a small collateral can be used to cover a relatively large amount of trades even in the case of a significant market movement.

#### Risk factor

{% hint style="warning" %}
This section is a draft of risk factor calculation, it needs to be refined to make sure the risk is always low for brokers and settlements are not triggered too often.
{% endhint %}

The previous example was using ETH and BTC which are in the same class of assets volatility and whose prices are very correlated. Some assets would have uncorrelated prices and very different volatility over time, SHIB and USDC for instance.

We calculate the risk factor for a given asset as the median of the daily price change over the last year.

As the day of writing we got the following results:

| Currency | Risk factor |
| -------- | ----------- |
| BTC      | 2.09%       |
| ETH      | 2.68%       |
| USDC     | 0.027%      |
| SHIB     | 3.64%       |
| DOGE     | 2.63%       |

The risk factor is applied on the value of each asset, we apply factors positively to the broker with the highest estimated value and negatively to the other.

The latest example with the risk factor gives the following results:

| Currency  | Broker A    | Broker B    |
| --------- | ----------- | ----------- |
| ETH       | $33,884     |             |
| BTC       |             | $51,403     |
| USDC      | $20,005     |             |
| **Total** | **$53,890** | **$51,403** |

Risk = 53,890 - 51,403 = $2,487

Including the volatility risk factor the maximum risk becomes $2,487. Including the volatility risk the collateral needs to be a lot more significant to cover the days of high volatility of assets, or the settlement between the brokers will be triggered more often.&#x20;

### State channel protocol

#### Overview

#### State definition

#### Adjudicator smart contract

#### Create a channel

#### Close a channel

### Settlement

#### Settlement trigger

#### Hash Time Lock (HTL)
