---
description: Notify and review if contract balances drop below a predefined threshold.
---

# Drop In System Funds

## Description

When funds stored in contracts drop significantly, it may be an indicator of either users exiting the system or a vulnerability being exploited to steal funds at large. Setup monitoring to detect funds dropping below specified thresholds, and notify administrators or stakeholders when such drops occur.

## Example

Every ERC20 Transfer event of relevant assets, where the first parameter is a contract of the protocol, should be followed by a verification of the contract's balances before and after the transaction. This should be done by emitting events and then using an off-chain client to monitor events and check balances, but in the absence of events may also be done by using the off-chain client to inspect mined transactions.

An alert to inform the appropriate stakeholders should be triggered when the balances drop below a predefined threshold. The threshold should be configured in the client so that it can be easily changed which may be advisable if overall balances change. The configured thresholds may given as a specific value, or a percentage drop, or both, and may also include a timeframe component. For example the client might notify if the balance drops by more than 5% in a single transaction, or by more than 10% in one hour or one day.

Separately, before integrating new assets into the protocol, ensure that any transfer of such assets will emit the expected ERC20 Transfer event when the protocol’s balances are affected, and that the token is fully supported in the threshold monitoring.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/drop-in-system-funds](https://defender.openzeppelin.com/#/advisor/docs/drop-in-system-funds?query=drop)

