---
description: >-
  What $YELLOW is used for, how supply and allocations are structured, and how
  rewards are accounted for.
---

# Tokenomics

$YELLOW is the platform token used for incentives and (where enabled) governance.

This page explains what $YELLOW is used for and where it appears in the product.

### Where $YELLOW is used

Common system uses:

* Rewards (referrals, trading incentives, campaigns)
* Staking (if enabled)
* Fee offsets or rebates (if enabled)
* Governance (if enabled)

Yellow shows the active uses inside the app. If a use is not visible in the app, treat it as not enabled.

### Supply and distribution

Tokenomics is defined by:

* Total supply
* Allocation buckets (team, treasury, community, ecosystem, investors, etc.)
* Vesting schedules for each bucket
* Emissions schedule (if rewards are emitted over time)

Publish these as a table with:

* bucket name
* allocation %
* cliff (if any)
* vesting duration
* unlock frequency (daily/weekly/monthly)
* notes

**Screenshot:** Tokenomics table (allocations + vesting summary)

### Utility availability

References to $YELLOW utilities in announcements or campaign materials may include planned features.

Only utilities marked as active in the Yellow app and this documentation should be treated as currently available.

### Emissions and rewards accounting

If Yellow distributes rewards in $YELLOW:

* rewards are tracked as ledger entries (pending → credited)
* distribution cadence and rules are defined per reward program (referral, trading, staking)

Users should be able to see:

* earned amount
* status (pending/credited/expired if applicable)
* timestamps and program name

**Screenshot:** Rewards history list showing status and timestamps

### Circulating vs locked tokens

* Circulating: tokens available to transfer on-chain and/or trade
* Locked: tokens subject to vesting, lockups, or staking lock rules

If Yellow publishes circulating supply, it should include:

* definition (what is counted)
* data source and update frequency

### Risks users should understand

* Token supply changes over time due to unlocks and emissions.
* Rewards programs can be updated or paused; rules should be versioned.
* Tokens are subject to market risk and network risk.

**Link placeholder:** On-chain explorer link(s) for $YELLOW contract(s)
