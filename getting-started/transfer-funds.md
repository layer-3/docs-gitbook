---
description: >-
  Learn how to move funds between your Account Balance and Trading Account, when
  to use each transfer direction, and what to do before withdrawing.
---

# Transfer Funds

Use the **Transfer** tab to move assets between your **Account Balance** and your **Trading Account**.

Transfer supports both directions:

* **Account Balance → Trading Account**
* **Trading Account → Account Balance**

This lets you move funds into trading when you want to place orders, and move them back when you want to withdraw.

### Before You Start

Make sure you have funds available in the balance you want to transfer from.

{% hint style="info" %}
**Important:** If you want to trade, your funds must be in **Trading Account**. If you want to withdraw to an external wallet, your funds must first be in **Account Balance**.
{% endhint %}

{% stepper %}
{% step %}
### Open the Transfer Tab

Go to the **Assets** page and open the **Transfer** tab.
{% endstep %}

{% step %}
### Choose the Transfer Direction

Select the direction you want to use:

* **Account Balance → Trading Account** to make funds available for trading
* **Trading Account → Account Balance** to move funds back before withdrawing

Review the direction carefully before continuing.
{% endstep %}

{% step %}
### Select the Asset and Enter the Amount

Choose the asset you want to transfer, then enter the amount.

Make sure the amount does not exceed your available balance in the source account.
{% endstep %}

{% step %}
### Review and Confirm

Review the transfer details and confirm the transfer.

Once completed, the funds will appear in the destination balance.
{% endstep %}
{% endstepper %}

<figure><img src="../.gitbook/assets/chrome-capture-2026-3-18.gif" alt=""><figcaption></figcaption></figure>

### When to Use Each Transfer Direction

#### Account Balance → Trading Account

Use this when:

* you signed in with Google and already deposited funds to **Account Balance**
* you want to move funds into **Trading Account** before placing trades

#### Trading Account → Account Balance

Use this when:

* you want to withdraw funds to an external wallet
* you need to move funds out of **Trading Account** after trading

{% hint style="info" %}
**Important:** Moving funds from **Trading Account** to **Account Balance** does not complete the withdrawal by itself. After that transfer is done, you still need to use the **Withdraw** tab to send funds to an external wallet.
{% endhint %}

### Fees and Timing

Transfers are processed onchain.

* transfer time depends on the selected network and current network conditions
* network confirmation may take a few moments

For transfers from **Trading Account** to **Account Balance** as part of the withdrawal flow, Yellow covers the network fee.

<details>

<summary>Why can’t I trade after depositing?</summary>

If you signed in with Google, your deposit first arrives in **Account Balance**. You must transfer it to **Trading Account** before trading.

</details>

<details>

<summary>Why can’t I withdraw from Trading Account?</summary>

Withdrawals can only be made from **Account Balance**. First transfer funds from **Trading Account** to **Account Balance**, then use the **Withdraw** tab.

</details>

<details>

<summary>Is there a fee for transfer?</summary>

Transfers are processed onchain. In the withdrawal direction, from **Trading Account** to **Account Balance**, Yellow covers the network fee.

</details>
