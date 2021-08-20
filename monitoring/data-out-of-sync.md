---
description: >-
  Notify and review if on-chain bookkeeping data gets out of sync, of if
  differences between protocol and external data exceeds an acceptable level of
  deviation.
---

# Data Out of Sync

## Description

Many protocols include bookkeeping balances that are expected to remain in sync, for example between individual user vaults and collateral funds. Mismatches between on-chain results and the expected behavior could indicate a security issue being exploited or an undiscovered bug that could lead to unexpected results or the loss of users’ funds.

On-chain bookkeeping is best enforced in the smart contract code, ensuring that system invariants are never broken. Maintaining consistency in all on-chain data is first a responsibility of on-chain code. This monitoring best practice, however, is an important safety net and also can be implemented in cases where the smart contracts do not fully enforce invariants.

Differences between protocol data and external data to which it is expected to sync can also introduce critical issues. For example in a protocol that involves use of on-chain prices, arbitrage exposures may be created for users of the protocol if the on-chain prices deviate from prices in other systems. Some systems require other synchronization of on- and off-chain data. Ultimately, in such situations, if on-chain data is not promptly corrected to be in sync with external data, then the system and users' assets may be at risk.

In cases where the protocol relies on internal bookkeeping or synchronization of data with external systems it is recommended to monitor the data values and notify and review if they are not in sync.

## Example

The most straightforward monitoring implementation is to setup an off-chain client which checks the bookkeeping and data synchronization after every mined transaction that involves any function that can change the bookkeeping data, and at fixed time intervals \(such as every 10 seconds\) in the absence of new transactions to check for changes in off-chain data. The monitoring client should notify administrators or stakeholders of deviations beyond acceptable thresholds.

Ideally the monitoring client will identify and indicate any specific transaction which immediately proceeds the deviation discovery. In the case of discovered deviations, administrators will need to review the transaction\(s\) that occurred just prior to determine if there are logic errors which must be fixed. In the case of deviation between on-chain and external data, the deviation may be due to a factor outside the contracts \(for example a failing oracle\), which is why it is critical that the monitoring client checks on fixed time intervals, and not just after mined transactions or events, when checking consistency with external data.

Another option to monitor on-chain bookkeeping is to implement an off-chain system to calculate the expected outcome of all transactions using parameters from mined transactions and then comparing the results with the on-chain data values. Transactions that show different results should then be flagged and sent to developers for further investigation. While this approach requires much more work, it could also be used later to run checks on transactions in the mempool before they are mined which might enable faster alerting or even pre-emptive action to avoid user losses.

