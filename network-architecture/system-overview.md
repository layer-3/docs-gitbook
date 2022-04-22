# The product



### The product <a href="#_qrz16kmo4nrn" id="_qrz16kmo4nrn"></a>

#### Overview[​](https://www.yellow.org/docs/litepaper/product#overview) <a href="#_9lqr4nsnpms7" id="_9lqr4nsnpms7"></a>

#### **The broker node** <a href="#_88o83bna5dhs" id="_88o83bna5dhs"></a>

Is built with OpenDAXTM WEB SDK which offers reusable UI components for common exchange platform interfaces, including widgets.

All components come with a simple, modern design and are customizable.

The library leverages React state management tools such as Providers, Hooks, and Utils to connect to the OpenDAXTM WEB SDK for JavaScript and pass data to the UI layer, simplifying state synchronization.

#### **Demo version** <a href="#_thuljtxfvgkg" id="_thuljtxfvgkg"></a>

**Link:**[ alpha.yellow.org](http://alpha.yellow.org)

**Functionality:**

1. Metamask Sign In
2. Deposit / Withdraw (custody contract)
3. Spot Trading (local node, network trading in progress)
4. Broker Admin Panel (Supabase Studio)

**Links:**

* npm: [https://www.npmjs.com/package/@openware/opendax-web-sdk](https://www.npmjs.com/package/@openware/opendax-web-sdk)
* docs: [https://storybook-odax-master.v4.uat.opendax.app/?path=/story/getting-started--page](https://storybook-odax-master.v4.uat.opendax.app/?path=/story/getting-started--page)

****

#### Modular architecture&#x20;

A great fit for exchange businesses, allows deep customization.

* Worldwide accessible assets of any type&#x20;
* Secure, customizable, and easily localized&#x20;
* Ultra-high-speed performance

****

**Matching engine\*\***[**​**](https://www.yellow.org/docs/litepaper/product#matching-engine)

* Each broker has their own matching engine, called “Finex”. It is a high performance trading engine of the OpenDAXTM crypto exchange software stack. It is written in GO language for high throughput of orders and trades. The Finex engine is the core component of the trading platform. It accepts or rejects orders, adding them to the order book, processes trades when orders match, and stores the results in the database. It prevents users from spending more funds than available on their balance.
  * Bulk order placement and bulk cancel orders.
  * Post-Only Order: post-only order is an advanced limit order that can be fully matched or fully canceled.
  * API Rate limit: a customizable rate limiter, can be configured on every API server or on the trading engine level.
  * Bi-directional websocket API: allows end-users and Brokers’ bots to create and cancel orders through this channel and be notified about trades in real time.

#### &#x20;<a href="#_x6p28ka0ouzh" id="_x6p28ka0ouzh"></a>

#### State channel technology <a href="#_gxnel8rqtpbo" id="_gxnel8rqtpbo"></a>

The state channel protocol keeps track of assets owned by brokers without involving blockchains in trade operations in an ultra-fast and secure manner.

State-channel technology creates a metadata index over the liquidity and digital assets of any blockchain and can add liabilities to it.

Two brokers open a trading channel using state channel technology, to share liquidity and order book.

They deposit collateral in a smart contract called the Adjudicator, which serves as an independent, neutral clearing house or intermediary; it also safeguards the collateral to ensure a fair outcome.

Brokers can close the trading channel or trigger a settlement process at any time to honor a withdrawal request by the end-user, or if the value of assets becomes unbalanced.

YELLOW tokens can be unlocked 30 days after the last trade conducted on the trading channel.

#### Multi-chain support <a href="#_o8p3zt8mxh94" id="_o8p3zt8mxh94"></a>

Pluggable custodians solutions

* Qredo
* Gnosis safe
* Cobo
* Fireblocks

**KYC (Know Your Customer)**

* End-user identity verification procedure is required to avoid participating in money laundering and terrorism financing activities.
* Each broker has the ability to change and integrate his KYC partner or solution.



**Blockchains**[**​**](https://www.yellow.org/docs/litepaper/product#blockchains)

* List of supported blockchain networks:
  * Ethereum
* List of networks to be supported later:
  * Solana
  * Polkadot
  * Binance Smart Chain

**Currencies and supported**[**​**](https://www.yellow.org/docs/litepaper/product#currencies-and-markets-supported) **markets**

* List of supported currencies is independent for each Broker. The Broker has an ability to add currencies and appropriate markets at his own discretion from the Admin panel.
* P2P trading is available to Brokers only for approved markets with assets matching from both sides.

**Decentralized support**[**​**](https://www.yellow.org/docs/litepaper/product#decentralized-support)

* Each Broker operates as an independent Exchange and is free to provide customer care for his end-users
* Best practice is to use Zendesk-like systems for operating activities.

### **Technologies Used and Created by the Project** <a href="#_5c6uz4n9qot6" id="_5c6uz4n9qot6"></a>

**Yellow Network:** state channels; P2P network

**Yellow nodes:** Jamstack, Golang, Next.js, gRPC
