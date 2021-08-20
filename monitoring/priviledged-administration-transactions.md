---
description: >-
  Notify and review all transactions executed by privileged accounts in the
  system to ensure they were expected and have caused the intended consequences.
---

# Priviledged Administration Transactions

## Description

If approved administrators did not trigger the related transaction / set of transactions, private keys controlling the privileged account may have been compromised. The entire system, including users’ funds, might be at risk. Also, mistakes in executing administrative transactions could lead to unintended and potentially widespread effects. Instituting a post-transaction audit and review process for administrative changes is considered a security best practice.

Therefore it is recommended to monitor and confirm all administrative transactions. This can be done by monitoring emitted events \(if implemented\), or specific contract functions, or by monitoring any use at all of privileged accounts especially if the use of those accounts should be limited to performing specific administrative transactions.

## Example

Set up an off-chain client that notifies the appropriate stakeholders whenever an administrator action is performed. A notification would not necessarily mean the system is experiencing an issue, but must be promptly inspected and escalated into a security alert if required.

Specifically, setup monitoring for:

* Changes in sensitive parameters of core contracts
* Calls to functions which add, renounce, or transfer ownership
* Withdrawals of central funds
* Calls to function which allow or block access to assets or capabilities for specific accounts
* Calls using privileged accounts not related to expected administrator operations, detected via inspection of mined transactions

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/privileged-administrator-transactions](https://defender.openzeppelin.com/#/advisor/docs/privileged-administrator-transactions?)

