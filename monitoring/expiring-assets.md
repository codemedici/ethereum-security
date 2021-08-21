# Expiring Assets

{% hint style="success" %}
For protocols that issue assets with expiration times, monitor the volume of expirations to correlate that with other monitored system events in case of unusual user behavior or attacks.
{% endhint %}

## Description

Near the expiration time of protocol assets, malicious actors may attempt different attacks to harm the protocol, such as purposely spamming the network or manipulating prices of underlying assets. These may trigger false alarms in other monitoring systems, or may signal attempts of abusing the protocol. Monitoring and keeping administators aware of expiration dates and volumes may help administrators more quickly identify the root cause or understand attack targets when other monitoring alarms are raised.

This information may also help administrators identify that other monitoring alarms can be ignored if they are due to normal increased activity for expiring assets, in which case the administrators may choose to adjust the sensitivity of other system monitoring and alarms.

## Example

Implement a monitoring client which gathers information about the value and volume of expiring assets and makes that information easily available to administrators in an easy-to-read format searchable by date and time.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/expiring-assets?](https://defender.openzeppelin.com/#/advisor/docs/expiring-assets?)

