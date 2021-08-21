# Network Congestion

{% hint style="success" %}
Monitor, notify and review transaction details when the Ethereum network is highly congested.
{% endhint %}

## Description

A malicious actor may attempt to congest the Ethereum network via a [block stuffing attack](https://solmaz.io/2018/10/18/anatomy-block-stuffing/), for example to prevent timely price oracle updates, leveraging arbitrage opportunities to steal funds from protocols. Furthermore, this type of attack can ultimately prevent protocols from completing user or administrator transactions which in specific cases might expose user or system funds to attack. Existing network congestion, even if not maliciously provoked, may have the same effects.

## Example

Create a client to monitor the Ethereum network \(either directly or via a trusted, reliable, source\), aiming at detecting anomalies in the percentage of gas limit used per block. If block gas used is unusually high \(for example, greater than 95%\), an alert should be triggered to notify all appropriate stakeholders and take the necessary precautions to protect users and funds.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/network-congestion?](https://defender.openzeppelin.com/#/advisor/docs/network-congestion?)

