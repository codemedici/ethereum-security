# Spikes in Failed Transactions

{% hint style="success" %}
Any unusual high volume of failed transactions targeting a protocol’s contracts should be detected and reviewed.
{% endhint %}

## Description

While some amount of failures during execution may be normal or expected, a sudden spike in the number of failed transactions might reveal the beginning of an attack or an underlying problem in the system or its dependencies. For example, this can occur if the parameters set for a new option were not properly selected. This could prevent users from performing normal actions on the platform, such as collecting their assets or exercising their rights.

A sudden rise in failed transactions might also indicate attackers attempting to execute complex attacks on the system. This also gains more relevance right after sensitive parameters of the protocol are updated or new features added.

During execution, a transaction might fail for multiple reasons, such as business logic errors, insufficient privileges to execute an action, insufficient gas included in the transaction, etc. While not all transaction failures represent a security risk, any spike in failed transactions targeting the protocol’s contracts should result in administrator notification and review.

## Example

Setup an off-chain client to collect metrics on the number of user transactions interacting with each contract function, and the ratio of failed transactions. A threshold can be defined for this ratio, so that any time the number of failed transactions grows above such threshold then administrators or stakeholders are promptly notified \(via alert sent to a communications channel\).

Close inspection and reproduction of failed transactions using debuggers may be required, to understand whether the behavior is expected and intended, or if there might be underlying problems in the system that demand response from the development or operations team.

Key focus should be placed on critical user-facing functions, especially those that operate on financial assets. However it is also desirable to have all exposed functions of contracts included in the failed transaction monitoring because any spike in failures may indicate potentially serious issues.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/spikes-in-failed-transactions?](https://defender.openzeppelin.com/#/advisor/docs/spikes-in-failed-transactions?)

