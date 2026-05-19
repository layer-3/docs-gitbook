---
description: >-
  How Yellow uses smart contracts for deposits and withdrawals, what stays under
  your control, and what happens during withdrawal.
---

# Self-Custody & Smart Contracts

Yellow uses smart contracts to lock assets for trading while keeping withdrawals under user control.

### What self-custody means on Yellow

* You deposit from your **own on-chain wallet**.
* Deposited funds are held in a **smart contract** associated with the state channel.
* Your trading balance is represented in Yellow’s **off-chain account ledger** while funds remain locked in the contract.
* You withdraw by initiating a withdrawal to an address you control.

Self-custody does not mean every trade happens on-chain. It means:

* the assets are held by code (smart contracts), not a custodian account in Yellow’s name
* the only on-chain movements are deposits and withdrawals

### Smart contracts involved

Users interact with smart contracts during:

* **Deposits**: wallet → state channel contract
* **Withdrawals**: state channel contract → wallet

Trading fills update the off-chain ledger and do not create a transaction per trade.

### What Yellow can and cannot do

Typical constraints in this design:

* Yellow can update your **off-chain ledger balances** when your orders fill.
* Yellow cannot move funds from the smart contract to an arbitrary address without a withdrawal process defined by the protocol.
* Withdrawals are the point where assets move back to an on-chain wallet.

Exact withdrawal rules (signing, delays, limits) are shown in the withdrawal flow and/or contract docs.

### Practical checklist

Before depositing:

* confirm the correct network
* confirm the contract address shown in the UI (if displayed)
* confirm token decimals and minimum deposit requirements

Before withdrawing:

* confirm destination address and network
* confirm withdrawal fee (network gas and any platform fee)
* verify the withdrawal status until the on-chain transaction is final

_Diagram: Wallet ↔ Smart Contract (state channel) boundary vs Off-chain Ledger_\
_Screenshot: Deposit flow showing network + contract address (if shown)_\
_Screenshot: Withdrawal flow showing destination address + network confirmation_
