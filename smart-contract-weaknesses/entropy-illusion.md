# Entropy Illusion

## Entropy Illusion

[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#entropy-illusion](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#entropy-illusion)\
All transactions on the Ethereum blockchain are deterministic state transition operations. This means that every transaction modifies the global state of the Ethereum ecosystem in a calculable way, with no uncertainty. This has the fundamental implication that there is no source of entropy or randomness in Ethereum. Achieving decentralized entropy (randomness) is a well-known problem for which many solutions have been proposed, including RANDAO, or using a chain of hashes, as described by Vitalik Buterin in the blog post “[Validator Ordering and Randomness in PoS](https://vitalik.ca/files/randomness.html)”.

### The Vulnerability

Some of the first contracts built on the Ethereum platform were based around gambling. Fundamentally, gambling requires uncertainty (something to bet on), which makes building a gambling system on the blockchain (a deterministic system) rather difficult. It is clear that the uncertainty must come from a source external to the blockchain. This is possible for bets between players (see for example the [commit–reveal technique](http://bit.ly/2CUh2KS)); however, it is significantly more difficult if you want to implement a contract to act as “the house” (like in blackjack or roulette). A common pitfall is to use future block variables—that is, variables containing information about the transaction block whose values are not yet known, such as hashes, timestamps, block numbers, or gas limits. The issue with these are that they are controlled by the miner who mines the block, and as such are not truly random. Consider, for example, a roulette smart contract with logic that returns a black number if the next block hash ends in an even number. A miner (or miner pool) could bet $1M on black. If they solve the next block and find the hash ends in an odd number, they could happily not publish their block and mine another, until they find a solution with the block hash being an even number (assuming the block reward and fees are less than $1M). Using past or present variables can be even more devastating, as Martin Swende demonstrates in his excellent [blog post](https://swende.se/blog/Breaking\_the\_house.html). Furthermore, using solely block variables means that the pseudorandom number will be the same for all transactions in a block, so an attacker can multiply their wins by doing many transactions within a block (should there be a maximum bet).

### Preventative Techniques

The source of entropy (randomness) must be external to the blockchain. This can be done among peers with systems such as commit–reveal, or via changing the trust model to a group of participants (as in RandDAO). This can also be done via a centralized entity that acts as a randomness oracle. Block variables (in general, there are some exceptions) should not be used to source entropy, as they can be manipulated by miners.\


## Examples

### Real-World Example: PRNG Contracts

In February 2018 Arseny Reutov [blogged](http://bit.ly/2Q589lx) about his analysis of 3,649 live smart contracts that were using some sort of pseudorandom number generator (PRNG); he found 43 contracts that could be exploited.

## Resources

* [https://blog.sigmaprime.io/solidity-security.html#visibility](https://blog.sigmaprime.io/solidity-security.html#visibility)
