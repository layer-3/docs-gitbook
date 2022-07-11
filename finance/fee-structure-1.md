# Fee Structure

The Yellow Clearing House will charge fees and reward participants based on their activity within the Yellow Network. The Yellow Network will not charge fees for profit; all collected fees are used for redistribution and token liquidity management;

![Yellow Network Fee Structure](https://lh5.googleusercontent.com/fy9\_\_xhynB9xqT1ugu2ENxbcnv2urPREK8wjzLpHMkVeu0\_P5\_dSxGjDD-WlDOqsCA76NCfA28pPKMbozyHB9A7Em8IsdgAMM3RN9ry1KvWGYvxQs5J7zruZ1AgH99ukM2zvdUG5PMQXX2nc5A)

Brokers using the Yellow Network will encounter two fee levels;

### Level 1 - Market Maker/Taker Fee&#x20;

The Market Maker/Taker fee is agreed upon between the brokers. The Market Maker will receive 100% of the fees paid by the Market Taker. The Yellow Network will not charge additional fees to either side.

The Market Taker Fee is volume-based and calculated on a per trade level when using a Market Makers liquidity. The fee is calculated in % of the trade's base currency.&#x20;

_Example: Broker 1 is seeking 10 ETH vs. USDt of Liquidity from Broker 2. The trade pair is quoted ETH/USDt and the base currency is - also called "transaction currency" - in this case is ETH. Thus the fee is calculated as: 10 ETH x Fee in %_

### Level 2 - Clearing Fee&#x20;

Whenever a Broker clears his position through the Yellow Network, a Clearing Fee will apply. This fee is calculated as 0.05% of the cleared position volume and converted into Yellow Tokens.

The clearing fee is redistributed as follows:&#x20;

* 50% converted to Yellow Token and sent to a burn address&#x20;
* 50% converted to Yellow Token and forwarded to a Treasury Vault&#x20;

_Example: Broker 1 is clearing a total volume of 25 ETH vs. USTt and the clearing fee is set at 0.05% of the base currency (ETH). The price of Yellow Tokens at the time of clearing is quoted as ETH/YT 0.008. The broker thus pays: 25 ETH x 0.05% / 0.008 = **1.5625 YT**_

_The Yellow Network will split the YT 1.5625 equally:_&#x20;

* _YT 0.78125 to a burn address_&#x20;
* _YT 0.78125 to the Yellow Treasury Vault_

### Use of the Clearing Fee

#### Token Burn (50%)&#x20;

Each time a Broker clears a position a fee is levied, 50% will be converted into Yellow Token and sent to a dedicated burn address, ensuring a constant demand and burn of Tokens.&#x20;

#### Yellow Treasury Vault (50%)

The remaining 50% of the clearing fee is converted to Yellow Token and sent to a treasury vault. The Yellow Treasury Vault will act as a security fund for the Yellow Network in the case of a clearing dispute or failure of the Broker to meet his obligations.

### Clearing Window

All positions are subject to daily clearing at 12am CET. Brokers have to ensure that they have sufficient collateral (as outlined in the Risk Management Section) to cover any outstanding balance. Failure to meet the balance will prevent the Broker from engaging in new positions until the requirements are met.&#x20;
