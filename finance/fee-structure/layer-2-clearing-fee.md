# Layer 2 - Clearing Fee

Fees that are paid from the broker to the Yellow Clearing House.

A Clearing Fee is due whenever Broker positions are cleared/settled on the Yellow Network. The fee is calculated as % of the settlement liability, converted into $YELLOW, accumulated, and invoiced monthly directly to the dedicated $YELLOW spending wallet of each broker.

There is a fixed clearing cycle that settles all cross-broker transactions daily at 0.00 am UTC.

The clearing fee is only applied to cross-broker exchanged volume.

Clearing fees are defined by Yellow Clearing House, and 100% of it goes to the Clearing House. There are dynamic fee levels with a floor fee that depends on the number of active (open) trading channels.

**Clearing Fee = Trade Nominal in  $YELLOW x Clearing Fee (%)**

Defined by:        Yellow Clearing House\
Beneficiary:       Yellow Clearing House\
Payment Ccy:    $YELLOW\
Periodicity:        Calculated every day at 0.00 am UTC based on final liabilities of each\
&#x20;                          broker and paid every day during the settlement.

| Broker Tier\* | $YELLOW Stacked | Clearing Fee |
| ------------- | --------------- | ------------ |
| 4 channels    | 250.000         | 1.00%        |
| 8 channels    | 500.000         | 0.80%        |
| 16 channels   | 1.000.000       | 0.40%        |
| 32 channels   | 2.000.000       | 0.20%        |
| 64 channels   | 4.000.000       | 0.10%        |

_\*The applicable Broker Tier is defined at the beginning of each clearing cycle. Brokers can not move tiers during a clearing cycle._

_Note: All positions are subject to a daily clearing at 0.00 am UCT. Brokers must ensure a sufficient $YELLOW token balance (as outlined in the Risk Management Section) to cover any outstanding Clearing Fees. Failure to meet the clearing fee payment will prevent the Broker from engaging in new positions until the payment for the previous cycle is satisfied._
