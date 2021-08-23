# Unused Tokens or Funds

{% hint style="success" %}
Monitor protocol functions used to issue tokens or deposit funds where accounts are expected to make immediate use of those tokens or funds, and alert if users do not carry out expected actions.
{% endhint %}

## Description

In many protocols, users are expected to carry out multiple actions closely one after another to avoid putting funds or tokens issued to them at risk. In some cases, however, users may not completely understand the protocol requirements, or developers may implement clients that fail to provide appropriate protections. In order to avoid users putting themselves at risk, monitor transactions or events and if steps are not fulfilled for an account within a "safe" timeframe then raise an alert for administrator action.

## Example

Define actions which, if not carried out in sequence, might put user funds or assets at risk, and define a timeframe in which they should be carried out \(from first to last\) to be considered "safe." Once defined, implement a client to monitor mined transactions or logged events for the associated contract functions, and to track the sequence from start to completion for individual accounts.

When the sequence is not fulfilled for a specific account within the "safe" timeframe, the monitoring system should alert administrators along with the account number \(and any other identifying information in the system\). Administrators may choose to notify individual users if they are putting themselves at risk, or if many alerts are raised then administrators may need to identify and address the root cause.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/unused-tokens-or-funds?](https://defender.openzeppelin.com/#/advisor/docs/unused-tokens-or-funds?)

