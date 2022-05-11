# Background

Crypto-currency trading is now a reality and is enjoying a fast rate of user adoption all over the world. Since Bitcoin many alternative projects were launched and users have now a large choice of different blockchains and many decentralized applications. New challenges appear to simplify the user journey in this wild wide web 3.0. Bridges are ones of several technologies brought to interconnect blockchains, it happened to suffer from a lack of decentralization, ERC20 native connection from one to another chain and a need of large liquidity for many tokens on every chain to satisfy the user demand.

### Related Work

#### Centralized Exchange

**Security concerns**

Centralized exchanges are fully in charge of the deposited users assets. It means users have to fully trust them to secure correctly platform wallets and process trades in fairly manner. Security has been taken seriously by big exchanges recently but it comes at a great cost, however we can still read news of exchanges being hacked and users funds being drained by attackers. Most of small exchange can't afford the cost of securing correctly users funds.

**Support of blockchains**

To keep a centralized exchange running, a lot of operations are needed. We just mentioned the security aspect which require a full-time focus by a dedicated team. The support of many blockchains is also very complicated and leads to high cost, each of them needs to be monitored specifically to make sure nodes and platform applications stay synchronized and process blocks in real time. The load of some chains sometime jumps quickly with the cost of transactions, this can lead to withdrawals not being proceed or very slowly, overall the gas price for withdrawals transactions and deposits processing might have to be adjusted frequently.

**Compliance**

For an exchange to comply with local regulations can be very complicated. Small exchanges will probably prefer to target a single market and comply with a single regulator. An other approach is to simply register the company in a country where they can operate without any regulation, this solution expose users to the goodwill of the platform operator, regulator rules being usually made to protect customers.

**Market making & Access to liquidity**

Running an exchange with many markets imposes to maintain orderbooks with tight spreads in order to provide the best offers possible to users. It also requires deep liquidity in the orderbook to avoid big price moves in case of sporadic big demand on a market.

Centralized exchanges usually delegate this duty to "market makers", this service can be very expensive and still the exchange might have to provide a big part of the liquidity to be injected in the orderbook.

#### Uniswap

**Short history**

Uniswap is a decentralized exchange application, launched in 2018 \[@uniswap-history] it's the first DEX to gain a significant traction on Ethereum mainnet by August 2020. Since then many clones and other decentralized applications were launched on many blockchains and used by millions of users to swap tokens, lend/borrow crypto-currencies assets, bridge funds between blockchains and many more use cases.

Uniswap paved the way of DeFi (Decentralized Finance).

**Security, Auditability**

It brought many advantages compared to centralized exchanges. The exchange software is fully implemented in smart-contracts that are deployed on the blockchain, thus anyone can read how it works, many audits are performed by independents parties, many DeFi applications were left with breaches and funds were exploited, but over time the security of those applications tends to be proven.

**Automatic market making**

Anyone can provide liquidity to Uniswap markets (AKA pools), and receive a revenue share from fees collected during trading (AKA swaps). Moreover the price of assets is managed automatically, every trade impacts the price up or down depending if it's a ask or a bid, since the version one the protocol evolved to be more resilient in v2 and to use more effectively the liquidity in v3 \[@angeris2020improved].

Uniswap protocol provided an elegant solution to the problem of market making and access to liquidity.

**Performances**

The success of those applications lead to traffic jams in Ethereum network, cost of transactions have been growing to reach unsustainable levels. DEX and DeFi application are facing the limitation of blockchains throughput. Many projects claim to solve the problem of blockchains scalibity, some drastically improve the throughput of transaction like Polygon or Solana.

**Front running bots**

Transparency of blockchain transactions and the fact that Ethereum order transactions by gas price exposes users of DEXes to front-run bots \[@daian2019flash].

Such bots are monitoring the blockchain and the in-memory transactions pool containing (transactions not yet mined in a block) for incoming swap transactions. They check slippage tolerance allowed by the user, calculate the cost of front-running transactions, when profitable they execute a transaction just before the user by setting a higher gas price and finally a transaction just after the user to take the profit.

**Comply with local regulations**

Decentralized application are accessible by everyone on the planet the same way, without any difference between users. This feature allows anyone to access crypto-currencies wallets and use those solutions, this is a very powerful feature which gives access to people who can't open a bank account to crypto-currencies and decentralized finance.

Nevertheless each country has specific regulations in place applied to financial products, thus those fully open and restrictionless solutions are violating those rules.

#### Lightning Network \[@poon2016bitcoin]

Lightning network is a _Layer-2_ solution for Bitcoin network. It was proposed as a solution to the scalability problem of Bitcoin network. It leverages state channel technology to be used as payments channels to perform any number of off-chain transactions. A payment channel is initiated by an on-chain funding transaction, then it's followed by any number of off-chain transactions, finally to commit balances a settlement transaction is performed on-chain; while lightning network introduce real-time fund transfer in blockchain, our protocol leverage the same technology for high frequency trading.

#### DyDx \[@juliano2017dydx]

DyDx is a trading platform of derivatives markets. It combines the speed of centralized orderbook and the transparency of decentralized applications. The result is a more secure and faster software architecture. The downside is that it's limited to Ethereum's users.

#### Qredo \[@mccuskerqredo]

Qredo focuses on securing digital assets using MPC \[REFERENCE NEEDED], eliminating the sensitive private key from the signature computation. It also allows settlements by updating an internal ledger entry without costly transaction on the blockchain \[TO BE CONFIRMED]. While Qredo brings state of the art security for digital assets it doesn't facilitate high frequency trading.

#### LayerZero

LayerZero provides an SDK which enable cross-chains transactions; but it requires to modify existing smart contracts and doesn't solve performance issues, it actually increase the numbers of transactions on already congested chains.

### &#x20;<a href="#_udybe2pc46jg" id="_udybe2pc46jg"></a>
