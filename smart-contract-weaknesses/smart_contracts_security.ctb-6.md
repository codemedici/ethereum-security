# Front-Running

## Front-Running \(AKA Transaction-Ordering Dependence\)

[https://consensys.github.io/smart-contract-best-practices/known\_attacks/\#front-running-aka-transaction-ordering-dependence](https://consensys.github.io/smart-contract-best-practices/known_attacks/#front-running-aka-transaction-ordering-dependence)

Above were examples of reentrancy involving the attacker executing malicious code within a single transaction. The following are a different type of attack inherent to Blockchains: the fact that the order of transactions themselves \(e.g. within a block\) is easily subject to manipulation.

Since a transaction is in the mempool for a short while, one can know what actions will occur before it is included in a block. This can be troublesome for things like decentralized markets, where a transaction to buy some tokens can be seen, and a market order implemented before the other transaction gets included. Protecting against this is difficult, as it would come down to the specific contract itself. For example, in markets, it would be better to implement batch auctions \(this also protects against high frequency trading concerns\). Another way to use a pre-commit scheme \(“I’m going to submit the details later”\).

