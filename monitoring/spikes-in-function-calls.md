# Spikes in Function Calls

{% hint style="success" %}
Notify and review whenever a particular set of exposed functions experience unusually high volume of subsequent calls.
{% endhint %}

## Description

Unusually high call volume may indicate variations in user behavior. Sudden spikes in calls to critical user-facing functions that operate on funds may be a signal of users leveraging unexpected lucrative opportunities, which could be detrimental for the solvency of the protocol.

## Example

All interactions with critical user-facing functions should be monitored to understand expected patterns of user behavior. This can be detected via inspection of mined transactions or by monitoring the emission of related events.

For each function or set of functions of interest, a threshold measured in the number of calls per unit of time should be defined. Whenever the monitoring system detects that the number of calls in a given time period exceeds the threshold, an alert should be raised to administrators to investigate the type of operations being executed and their actual impact on the system. For example, an alert could be triggered when the number of calls in a single day exceeds 125% of a 30-day-long moving average, or when the number of calls in an hour on a given day exceeds 125% of the average for that same day and hour over the prior 10 weeks.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/spikes-in-function-calls?](https://defender.openzeppelin.com/#/advisor/docs/spikes-in-function-calls?)

