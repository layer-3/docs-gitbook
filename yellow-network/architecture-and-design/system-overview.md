---
description: Technologies used and created by the Yellow Network team
---

# System Overview

![](<../../.gitbook/assets/Technologies Used and Created by Yellow.png>)

**Yellow Network:** state channels; P2P network

**Yellow Nodes:** [Jamstack](https://jamstack.org/), [Golang](https://go.dev/), [Next.js](https://nextjs.org/), [gRPC](https://grpc.io/)

## **The Broker Node** <a href="#_88o83bna5dhs" id="_88o83bna5dhs"></a>

The broker node is built with [OpenDAX™ WEB SDK](https://web-sdk.openware.com/?path=/story/getting-started--page), which offers reusable UI components for common exchange platform interfaces, including widgets.

All components come with a simple, modern design, and are customizable.

The library leverages React state management tools such as providers, hooks, and utils to connect to the [OpenDAX™ WEB SDK](https://web-sdk.openware.com/?path=/story/getting-started--page) for JavaScript and pass data to the UI layer, simplifying state synchronization.

Demo Version - [alpha.yellow.org](http://alpha.yellow.org/)

**Functionality**

1. MetaMask Sign In.
2. Deposit /Withdraw (custody contract).
3. Spot Trading (local node, network trading in progress).
4. Broker Admin Panel (Supabase Studio).

#### Modular Architecture&#x20;

A great fit for exchange businesses allows deep customization.

* Worldwide accessible assets of any type.
* Secure, customizable, and easily localized.
* Ultra-high-speed performance.

## **Matching Engine**[**​**](https://www.yellow.org/docs/litepaper/product#matching-engine)

Each broker has its own matching engine, called _Finex,_ a high-performance trading engine of the [OpenDAX™ crypto exchange software stack](https://www.openware.com/product/opendax). It is written in GO language for high throughput of orders and trades. The _Finex_ engine is the core component of the trading platform. It accepts or rejects orders, adds them to the order book, processes trades when orders match, and keeps a persistent state in the database. It prevents users from spending more funds than are available on their balance.

Features supported in _Finex_ spot matching engine:

* **Bulk orders** to post or cancel many orders in a single request
* **Post-only order:**  advanced limit order which will not match immediately; it guarantees that it will be filled or canceled
* **Fill or kill order:** an advanced limit order which must be fully filled or gets automatically canceled
* **Stop-loss:** advanced market order used to buy or sell a specific asset once the market reaches the limit price. Once the predefined limit price is achieved, the system creates a spot market order.&#x20;
* **Stop-limit:** advanced limit order used to buy or sell a specific asset at a predefined limit.  Contrary to the stop-loss order, a stop-limit order creates a limit order instead of buying the asset outright once the limit price is achieved.&#x20;
* **API Rate limit**: a rate limiter configurable depending on user roles; it can be configured at the API server level and the trading engine level
* **Bi-directional WebSocket API**: allows end-users and brokers' bots to create and cancel orders and be notified about trades in real-time
* [**gRPC API** with JS and go libraries to speed up the development of bots](https://docs.openware.com/opendax/developers-guides/api-documentation/grpc-protocol)

## State Channel Technology <a href="#_gxnel8rqtpbo" id="_gxnel8rqtpbo"></a>

The state channel protocol keeps track of assets owned by brokers, without involving blockchains in trade operations, in an ultra-fast and secure manner.

The state channel technology creates a metadata index over the liquidity and digital assets on any blockchain, and can add liabilities to it.

Two brokers open a trading channel using state channel technology, to share liquidity and order books.

They deposit collateral in a smart contract called the adjudicator, which serves as an independent, neutral clearing house or intermediary; it also safeguards the collateral to ensure a fair outcome.

Brokers can close the trading channel or trigger a settlement process at any time to honor a withdrawal request by the end-user, or if the value of the underlying assets becomes unbalanced.

Brokers are required to lock $YELLOW tokens to open channels with other participants. Once a channel is closed any $YELLOW tokens can be unlocked 30 days after the last trade conducted on the trading channel.

[Read more about how we use state channel technology for our smart clearing protocol.](smart-clearing-protocol.md)

## Electronic Communication Network (ECN)

Yellow Network implements a mesh and decentralized ECN.

An electronic communication network (ECN) is a computerized system that automatically matches buy and sell orders for securities in the market. ECN trading is especially helpful when participants in different geographical areas wish to complete a secure transaction without the use of a third party.

The ECN aggregates the books of brokers and market makers in different locations. This leads to deeper liquidity and larger order books and thus a tighter bid-ask spread and faster order execution. Traders can get more attractive spreads and better order flow.&#x20;

An additional benefit of the ECN's order aggregation is better price transparency for all participating brokers and market makers.&#x20;

Using an ECN counters market fragmentation issues, while maintaining the decentralization aspect of the crypto trading market.&#x20;

_\[INSERT: ECN Deal Flow Graphic (in preparation)]_

ECN provides brokers access to global financial markets; the Yellow Network protocol forms a decentralized ECN for digital assets.

## &#x20;<a href="#_o8p3zt8mxh94" id="_o8p3zt8mxh94"></a>
