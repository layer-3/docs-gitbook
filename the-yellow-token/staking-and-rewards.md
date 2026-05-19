---
description: >-
  How staking positions work, how rewards are tracked, and what to expect from
  claim or auto-credit flows.
---

# Staking & Rewards

This page explains how staking and rewards work in the app.

### Staking (if enabled)

Staking means locking $YELLOW under defined rules to receive benefits.

A staking position is defined by:

* amount staked
* lock type (flexible or fixed term)
* lock duration (if fixed)
* start time and unlock time
* reward calculation rules

Users should be able to see:

* staked amount
* claimable rewards
* next reward timestamp (if scheduled)
* unlock date (for fixed staking)

**Screenshot:** Staking dashboard (staked, claimable, lock status)

#### Locking rules

Typical locking mechanics:

* Flexible staking: withdraw anytime (may have cooldown)
* Fixed staking: withdraw only after unlock (or with penalty if allowed)

If cooldowns exist, show:

* cooldown length
* what happens during cooldown (rewards continue or stop)

**Screenshot:** Stake/unstake flow with lock and cooldown info

### Rewards (general model)

Rewards in Yellow can come from multiple programs:

* referrals
* trading incentives
* staking rewards
* other campaigns

Each reward entry should specify:

* source (program name)
* amount
* asset ($YELLOW or other)
* status (pending/credited)
* timestamp

**Screenshot:** Rewards history with program labels

### Claiming vs auto-credit

Two common models:

* Auto-credit: rewards are credited automatically on schedule
* Claim-based: you must claim; unclaimed rewards may expire after a period

Yellow should explicitly state which model is used.

### Referral rewards (common case)

If referral rewards are paid in $YELLOW:

* trading fees are still charged normally
* rewards are tracked separately in the rewards ledger
* rewards appear as entries tied to referral activity events

**Screenshot:** Referral rewards entry in rewards ledger

### Important constraints

* Rewards can be reduced/withheld if activity is flagged as invalid (if enforcement exists).
* Rewards schedules and formulas can change; rule versions should be documented.
