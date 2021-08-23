---
description: >-
  Notify and review system health if a single account exceeds a predefined
  number of transactions in a certain time frame.
---

# Spikes in Account Activity

## Description

Extremely frequent use of the protocol by a single account may be the reflection of malicious operations on the protocol, either attempting to exploit a zero-day vulnerability or spam transactions in the network. This takes particular relevance for accounts that never interacted with the protocol before.

## Example

Create a client to monitor pending and mined transactions targeting protocol contracts, and maintain an off-chain history of accounts used along with average and maximum transactions per account per hour. The historical data record will make it possible to identify common averages and peaks and to establish thresholds that can be used to determine spikes. Thresholds may be tied to times of day or days of the week. Thresholds should be within a specific timeframe, for example NN transactions within 1 hour.

In the early stage of monitoring, before the historical record is captured and analyzed, administrators can choose thresholds that can be used to flag potential spikes. After historical data is captured, more granular thresholds can be established, possibly also different thresholds for new accounts.

The monitoring client should notify administrators or stakeholders when the number of transactions submitted by an individual account exceeds the established thresholds. Activity for flagged accounts should be reviewed.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/spikes-in-account-activity](https://defender.openzeppelin.com/#/advisor/docs/spikes-in-account-activity)

