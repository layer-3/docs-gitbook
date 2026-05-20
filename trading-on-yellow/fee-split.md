# Fee Split

Every trading fee on Yellow gets split into buckets. Here is where your fee goes.

## The split

For a standard trade (no referral attached):

| Bucket              | Share |
| ------------------- | ----- |
| Buyback of $YELLOW  | 85%   |
| Platform operations | 15%   |

For a trade by a referred user:

| Bucket                 | Standard | Referred |
| ---------------------- | -------- | -------- |
| Platform operations    | 15%      | 5%       |
| Rewards & competitions | —        | 10%      |
| Referral commission    | —        | 20–50%   |
| Buyback of $YELLOW     | 85%      | the rest |

## What each bucket does

* **Buyback** — used to buy $YELLOW on the open market. See [$YELLOW Buyback](buyback.md).
* **Platform operations** — funds Yellow's running costs.
* **Rewards & competitions** — funds trading competitions and reward campaigns paid out to users.
* **Referral commission** — paid to the partner who referred the user. The rate depends on the partner agreement. See [Referral Program](../getting-started/referral-program/).

## How it works in practice

The split happens automatically on every fill. You don't see it as a deduction — your fee is still your fee. What changes is where that fee goes after Yellow collects it.

A referred user pays the same fee a non-referred user does. The difference is internal: a portion of their fee is routed to their partner instead of to buyback.

## Why it's structured this way

The buyback bucket is the dominant share. It ties $YELLOW directly to platform activity — every trade generates buy pressure.

The referral bucket lets Yellow grow through partners without inflating fees on the user side. Partners are paid from the same fee, not on top of it.

## Related

* [$YELLOW Buyback](buyback.md) — how the 85% becomes $YELLOW
* [Referral Program](../getting-started/referral-program/) — how the referral bucket works
* [Fees](fees.md) — what you actually pay
