---
description: >-
  What Yellow audits cover, how to read audit reports, and what audits can and
  cannot guarantee.
---

# Security Audits

Yellow security reviews focus on the components that can affect custody, settlement, and trading integrity.

### What is audited

Audits typically cover:

* smart contracts used for deposits/withdrawals (state channel contracts)
* contract upgrade mechanisms (if any)
* permissioned roles and admin controls (if any)
* critical off-chain components that interact with contracts (if in scope)
* key cryptographic flows (signing, verification, proofs, if applicable)

### What an audit does and does not do

An audit is:

* a review of specific code versions at a point in time
* a list of findings with severity and remediation notes

An audit is not:

* a guarantee against all bugs
* continuous monitoring unless explicitly stated

### How to read an audit report (what to look for)

When Yellow publishes audit reports, useful details to check:

* scope: which repositories, contracts, and commit hashes were reviewed
* main risks: custody, withdrawal logic, upgrade rights, access control, accounting
* findings: critical/high severity items and whether they were fixed
* re-audit / verification: whether fixes were re-checked

### What users should do with audit information

* confirm you are interacting with the contract addresses referenced in the docs/UI
* verify that the deployed contracts match the audited versions (if Yellow provides proof/links)
* use the status page and incident posts if something changes

_Link placeholder: Audit reports index (PDF links or GitHub releases)_\
_Screenshot: “Security” page section listing audits + dates + scope summaries_
