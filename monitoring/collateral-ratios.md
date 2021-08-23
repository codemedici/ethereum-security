---
description: >-
  In cases where collateral is required, notify and review if the collateral
  ratio of any open account drops to the minimum collateralization threshold.
---

# Collateral Ratios

## Description

Collateral ratios are intended as a buffer to protect protocol solvency. When collateral ratios are trending lower, the risk of the protocol becoming insolvent is greater. Immediate action might be required if an increasing number of account collateral ratios drop suddenly or in unison. Make sure to monitor collateral ratios and alert when they drop precipitously or below defined thresholds.

## Example

Assuming that your contracts include a function to check the collateral ratio of an account, implement an off-chain monitoring client which checks and records the collateral ratio by calling the function for every open account. Use the monitoring client to maintain an off-chain list of accounts ordered by risk \(lowest collateral ratio combined with highest balance equals highest risk\). Whenever balances or obligations change for an account, or when associated price oracles are updated, the collateral ratios for each account should be checked and recorded. Using predetermined thresholds for minimum collateral ratio and maximum risk, the monitoring client should notify system administrators, and possibly take other automated actions, if the thresholds are crossed.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/collateral-ratios?](https://defender.openzeppelin.com/#/advisor/docs/collateral-ratios?)

