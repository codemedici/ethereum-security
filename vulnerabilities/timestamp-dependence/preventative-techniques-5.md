# Preventative Techniques

## Timestamp Dependence

There are three main considerations when using a timestamp to execute a critical function in a contract, especially when actions involve fund transfer.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts \(time-locking\), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use [block.number](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#block-and-transaction-properties) and an average block time to estimate times; with a `10 second` block time, `1 week` equates to approximately, `60480` blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable easily to manipulate the block number. The [BAT ICO](https://etherscan.io/address/0x0d8775f648430679a709e98d2b0cb6250d2887ef#code) contract employed this strategy.

It is possible to estimate a time delta using the `block.number` property and [average block time](https://etherscan.io/chart/blocktime), however this is not future proof as block times may change \(such as [fork reorganisations](https://blog.ethereum.org/2015/08/08/chain-reorganisation-depth-expectations/) and the [difficulty bomb](https://github.com/ethereum/EIPs/issues/649)\). In a sale spanning days, the [15-second rule](https://consensys.github.io/smart-contract-best-practices/recommendations/#the-15-second-rule) allows one to achieve a more reliable estimate of time.

**Note**  
If the scale of your time-dependent event can vary by 15 seconds and maintain integrity, it is safe to use a `block.timestamp`.

