---
description: >-
  Learn which order types are available on Yellow, how they work, and what to
  review before submitting an order.
---

# Order Types

Every order on Yellow includes:

* **side** — buy or sell
* **size** — how much you want to trade
* **order type** — market, limit, or stop
* **optional controls** — such as time-in-force, where available

### Market Order

A market order is submitted for immediate execution at the best available prices in the order book.

Possible outcomes:

* full fill
* partial fill, if available liquidity is limited

Market orders do not let you set a fixed execution price. If the market moves quickly, your final fill price may differ from the price shown before submission.

### Limit Order

A limit order lets you define the price at which you want to buy or sell.

* a **buy limit** sets the maximum price you are willing to pay
* a **sell limit** sets the minimum price you are willing to accept

Possible outcomes:

* if your limit price matches available liquidity, the order may fill immediately
* if it does not, the order remains open until it is filled or canceled

### Time-in-Force

Time-in-force defines how long an order can remain active.

Common options include:

* **GTC (Good-Til-Canceled)** — stays active until filled or canceled
* **IOC (Immediate-Or-Cancel)** — fills what it can immediately and cancels the rest
* **FOK (Fill-Or-Kill)** — must fill completely immediately or it is canceled
* **GTD (Good-Til-Date)** — stays active until a specified time, if supported

Availability may vary depending on the market and order type.

### Stop Orders

A stop order is triggered only when a specified price condition is met.

Common stop order types:

* **Stop Market** — once triggered, a market order is submitted
* **Stop Limit** — once triggered, a limit order is submitted

Key fields usually include:

* **trigger price**
* **trigger source**
* **order type after trigger**

Always review stop order settings carefully before submission.

### Take Profit and Stop Loss

Take Profit and Stop Loss instructions must currently be created manually through the order form.

They are not automatically linked to an existing order or position, so users should review them carefully after submission.

### Common Order Errors

Typical reasons an order may be rejected include:

* price is not aligned with tick size
* size is not aligned with step size
* order is below the minimum size or value
* insufficient available balance
* invalid trigger settings
* market restrictions or temporary unavailability

### What to Check Before Submitting an Order

Before placing an order, confirm:

* selected market
* order side
* order type
* price, if required
* size
* any trigger conditions
* any time-in-force setting, if used
