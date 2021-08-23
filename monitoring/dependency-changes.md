# Dependency Changes

For any application or protocol that relies on or uses other protocols or ERC20 tokens, monitor and notify on updates to those so that administrators or stakeholders can review the changes and evaluate the potential impact.

## Description

Modifications and upgrades to protocols or tokens may have a variety of effects upon other systems that are dependent upon them. For instance, a dependency migrating to a new address could cause failures when calling to previously used addresses, changing token decimals could result in erroneous calculations, a new version of a DEX such as Uniswap could cause a lack of liquidity for a previously used pool, or there might be breaking changes in external interfaces that break functionality. Action might be required to properly adapt and integrate with the dependencies’ newer versions.

## Example

Determine sources of information for protocol changes in each dependency. As an example, Compound may be updated via community governance, but this is typically monitored on their Discord server. Reliable processes should be established to stay up-to-date on all changes in dependencies, raising alerts to administrators or stakeholders whenever a change to dependent smart-contract functionality is detected.

Additionally, consider analyzing all dependencies to build a list of sensitive actions within these protocols, such as changes to the administrator addresses and modifications to sensitive parameters that may change the expected behavior. Note that this also applies for all ERC20 token contracts used which might implement additional functionalities \(such as upgradeability in USDC, or forking and migration in REP\). In these cases use an off-chain client to monitor mined transactions or specific events and to notify administrators or stakeholders if sensitive actions are detected.

Changes and sensitive actions will need to be examined and handled on a case-by-case basis. Where possible, earlier detection and examination of proposed changes will afford more time for evaluation and action, so consider implementing automation or even manual processes that will favor earlier detection of dependency changes.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/dependency-changes?](https://defender.openzeppelin.com/#/advisor/docs/dependency-changes?)

