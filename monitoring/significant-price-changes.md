# Significant Price Changes

{% hint style="success" %}
Notify and review system health if prices of assets relied on by the system rise or drop a specified % within a specified time interval.
{% endhint %}

## Description

Steep variations in prices of assets that a protocol relies on might have undesired or unexpected consequences, with the greatest risk being that the protocol becomes insolvent. Monitor the prices of assets the protocol relies on, and notify administrators and stakeholders of significant price changes so that system health can be reviewed.

## Example

Create a monitoring client to frequently check prices of critical assets using multiple trusted price data sources, and record the price history in a local database. For example the monitoring client might record price data every 5 seconds. If the protocol relies on an on-chain oracle, include data from that oracle in the monitoring but also include other price data sources for cross-reference. The client might also monitor pending transactions to the oracle \(to identify pending price changes before they are recorded on the blockchain\).

On top of the monitoring price database implement an alert system that checks for rapid price increases or decreases and notifies adminitrators or stakeholders when they occur. For example the alert system might check for price variation of 20% up or down within any 30 minute window. These parameters \(% change and timeframe\) should be configurable. When notified of a significant price change, administrators will need to review the details to determine impact on the protocol and to determine what if any are the appropriate actions to take.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/significant-price-changes?](https://defender.openzeppelin.com/#/advisor/docs/significant-price-changes?)

