# Spikes in Low Value Transactions

{% hint style="success" %}
Notify and review operations that occur with high frequency and only affect an unusually low amount of funds.
{% endhint %}

## Description

High frequency of low-value operations may indicate attempts of spamming or griefing attacks on a protocol, or may indicate a hacker trying to exploit an unreported vulnerability. For safety monitor such low-value transactions and alert and review when a batch is detected.

## Example

Implement a client to monitor mined transactions or logged events to determine transaction value, which may also require integration to obtain asset pricing data from trusted data sources. A lower threshold for total transaction value must be defined, and a timeframe and a threshold for the number of transactions which would be deemed a "spike" must also be defined, so that when the monitoring process detects a set of transactions below the value threshold but above the spike threshold within the specified timeframe then an alert is sent to administrators or stakeholders \(who then should review the transactions and system health\).

Over time, historical data can be used to determine the low-value and the spike thresholds. For example the monitoring system could keep statistics for the prior 30 days and alert when detecting a set of transactions within a timeframe \(such as 15 minutes\) where the value of each transactions is below the 5th percentile for transaction values for the prior 30 days and the count of such transactions is above the 75th percentile for transactions within a 15 minute window.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/spikes-in-low-value-transactions?](https://defender.openzeppelin.com/#/advisor/docs/spikes-in-low-value-transactions?)

