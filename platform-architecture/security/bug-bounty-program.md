---
description: >-
  How to report security issues, what is in scope, and how severity and rewards
  are typically handled.
---

# Bug Bounty Program

A bug bounty program is a channel for reporting security issues to Yellow with defined rules and rewards.

### What the program is for

The bounty program is intended for:

* vulnerabilities in smart contracts used for deposits/withdrawals
* issues that could cause loss of funds, incorrect balances, or withdrawal bypass
* critical issues in trading, margin, liquidation, or settlement logic
* API and auth issues that can impact account safety (if included in scope)

### What is usually out of scope

Common exclusions (varies by program):

* social engineering and phishing
* issues requiring physical access to a device
* DoS issues without impact on custody or balances
* low-severity UI/UX issues without security impact

The exact scope and rules should be listed in the bounty page.

### How to report

A valid report should include:

* a clear description of the issue
* affected component (contract/API/UI)
* impact: what an attacker can do
* steps to reproduce
* proof of concept (if possible)
* suggested fix or mitigation (optional)

### Disclosure rules

Bug bounty programs usually require:

* private reporting first
* no public disclosure until Yellow confirms a fix or grants permission

### Severity and payout logic

Severity is typically based on:

* ability to steal funds or bypass withdrawals
* ability to manipulate balances, PnL, or liquidation outcomes
* ability to compromise accounts or API keys

Rewards depend on severity and scope.

_Link placeholder: Bug bounty policy (platform link)_\
_Screenshot: Bug bounty page showing scope + reporting method + disclosure rules_
