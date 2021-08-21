# Asset Attacks and Issues

{% hint style="success" %}
Monitor trading volume, news, social media feeds and community forums related to assets used in the protocol in order to identify serious issues that might result in sharp price variations that could impact user or system funds.
{% endhint %}

## Description

When asset protocols are successfully attacked or have other serious issues, the price of their associated asset could suffer sudden and significant drops in value, and with that user funds or balances may dramatically shift with short notice. For example if a protocol relies on collateralization, and the asset used for collateral suffers an attack resulting in a sudden price drop, then user accounts may suddenly become under-collateralized. This could lead to user or even protocl insolvency. Therefore it is important to monitor public and community news to discover ecosystem attacks or issues as soon as possible so that administrators or stakeholders may take preventative action.

## Example

Make a list of whitelisted assets and for each asset identify sources for trading volumes and community news such as Telegram or Discord channels, also identify project identifiers associated with the assets on social networks such as Twitter. Also make a list of community channels or social networks that are commonly used for security or ecosystem information such as the ETHSecurity Telegram channel. These information sources can be ranked based on the expected accuracy, timeliness and criticality of content.

Initially, monitor the information sources on a daily basis, multiple times within each day. This can be done by rotating responsibility among admnistrators or stakeholders, possibly with each taking a subset of the information sources, and notifying others if suspicious or concerning information is found. Indicators of security issues, reported protocol or asset problems, or rapid increases in trading volume for an asset are all issues that should be investigated to determine what impact they might have on user or system accounts.

Note that many hackers target attacks for weekends or other hours that they think are outside of normal working hours for project teams. Consider ways to utilize team members in different timezones to check throughout every 24 hours and reduce the latency of response to any potentially impactful issues.

Later, a monitoring client can be build to poll some or all of the information sources, scanning for keywords or trading volume spikes. The monitoring client could post hits to administrators or stakeholders, possibly through a shared communication channel. Items that are deemed to be of real concern can then be further investigated.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/asset-attacks-and-issues?](https://defender.openzeppelin.com/#/advisor/docs/asset-attacks-and-issues?)

