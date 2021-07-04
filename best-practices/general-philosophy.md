# General Philosophy

Ethereum and complex blockchain programs are new and highly experimental. Therefore, you should expect constant changes in the security landscape, as new bugs and security risks are discovered, and new best practices are developed. Following the security practices in this document is therefore only the beginning of the security work you will need to do as a smart contract developer.

Smart contract programming requires a different engineering mindset than you may be used to. The cost of failure can be high, and change can be difficult, making it in some ways more similar to hardware programming or financial services programming than web or mobile development. It is therefore not enough to defend against known vulnerabilities. 

Defensive programming is a style of programming that is particularly well suited to smart contracts. It emphasizes the following, all of which are best practices:

## Minimalism/simplicity

Complexity is the enemy of security. The simpler the code, and the less it does, the lower the chances are of a bug or unforeseen effect occurring. When first engaging in smart contract programming, developers are often tempted to try to write a lot of code. Instead, you should look through your smart contract code and try to find ways to do less, with fewer lines of code, less complexity, and fewer "features." If someone tells you that their project has produced "thousands of lines of code" for their smart contracts, you should question the security of that project. Simpler is more secure.

Complexity increases the likelihood of errors.

* Ensure the contract logic is simple
* Modularize code to keep contracts and functions small
* Use already-written tools or code where possible \(eg. don't roll your own random number generator\)
* Prefer clarity to performance whenever possible
* Only use the blockchain for the parts of your system that require decentralization

## Code quality

Smart contract code is unforgiving. Every bug can lead to monetary loss. You should not treat smart contract programming the same way as general-purpose programming. Writing DApps in Solidity is not like creating a web widget in JavaScript. Rather, you should apply rigorous engineering and software development methodologies, as you would in aerospace engineering or any similarly unforgiving discipline. Once you "launch" your code, there’s little you can do to fix any problems.

## Readability/auditability

Your code should be clear and easy to comprehend. The easier it is to read, the easier it is to audit. Smart contracts are public, as everyone can read the bytecode and anyone can reverse-engineer it. Therefore, it is beneficial to develop your work in public, using collaborative and open source methodologies, to draw upon the collective wisdom of the developer community and benefit from the highest common denominator of open source development. You should write code that is well documented and easy to read, following the style and naming conventions that are part of the Ethereum community.

## Test coverage

Test everything that you can. Smart contracts run in a public execution environment, where anyone can execute them with whatever input they want. You should never assume that input, such as function arguments, is well formed, properly bounded, or has a benign purpose. Test all arguments to make sure they are within expected ranges and properly formatted before allowing execution of your code to continue.

## Prepare for failure

Any non-trivial contract will have errors in it. Your code must, therefore, be able to respond to bugs and vulnerabilities gracefully.

* Pause the contract when things are going wrong \('circuit breaker'\)
* Manage the amount of money at risk \(rate limiting, maximum usage\)
* Have an effective upgrade path for bugfixes and improvements

## Rollout carefully

It is always better to catch bugs before a full production release.

* Test contracts thoroughly, and add tests whenever new attack vectors are discovered
* Provide [bug bounties](https://consensys.github.io/smart-contract-best-practices/software_engineering/#bug-bounty-programs) starting from alpha testnet releases
* Rollout in phases, with increasing usage and testing in each phase

## Stay up to date

Keep track of new security developments.

* Check your contracts for any new bug as soon as it is discovered
* Upgrade to the latest version of any tool or library as soon as possible
* Adopt new security techniques that appear useful

### Be aware of blockchain properties <a id="be-aware-of-blockchain-properties"></a>

While much of your programming experience will be relevant to Ethereum programming, there are some pitfalls to be aware of.

* Be extremely careful about external contract calls, which may execute malicious code and change control flow.
* Understand that your public functions are public, and may be called maliciously and in any order. The private data in smart contracts is also viewable by anyone.
* Keep gas costs and the block gas limit in mind.
* Be aware that timestamps are imprecise on a blockchain, miners can influence the time of execution of a transaction within a margin of several seconds.
* Randomness is non-trivial on blockchain, most approaches to random number generation are gameable on a blockchain.

## Fundamental Tradeoffs: Simplicity versus Complexity cases

There are multiple fundamental tradeoffs to consider when assessing the structure and security of a smart contract system. The general recommendation for any smart contract system is to identify the proper balance for these fundamental tradeoffs.

An ideal smart contract system from a software engineering bias is modular, reuses code instead of duplicating it, and supports upgradeable components. An ideal smart contract system from a secure architecture bias may share this mindset, especially in the case of more complex smart contract systems.

However, there are important exceptions where security and software engineering best practices may not be aligned. In each case, the proper balance is obtained by identifying the optimal mix of properties along contract system dimensions such as:

* Rigid versus Upgradeable
* Monolithic versus Modular
* Duplication versus Reuse

### Rigid versus Upgradeable

While multiple resources, including this one, emphasize malleability characteristics such as Killable, Upgradeable or Modifiable patterns there is a _fundamental tradeoff_ between malleability and security.

Malleability patterns by definition add complexity and potential attack surfaces. Simplicity is particularly effective over complexity in cases where the smart contract system performs a very limited set of functionality for a pre-defined limited period of time, for example, a governance-free finite-time-frame token-sale contract system.

### Monolithic versus Modular

A monolithic self-contained contract keeps all knowledge locally identifiable and readable. While there are few smart contract systems held in high regard that exist as monoliths, there is an argument to be made for extreme locality of data and flow - for example, in the case of optimizing code review efficiency.

As with the other tradeoffs considered here, security best practices trend away from software engineering best practices in simple short-lived contracts and trend toward software engineering best practices in the case of more complex perpetual contract systems.

### Duplication versus Reuse

A smart contract system from a software engineering perspective wishes to maximize reuse where reasonable. There are many ways to reuse contract code in Solidity. Using proven previously-deployed contracts _which you own_ is generally the safest manner to achieve code reuse.

Duplication is frequently relied upon in cases where self-owned previously-deployed contracts are not available. Efforts such as [OpenZeppelin's Solidity Library](https://github.com/OpenZeppelin/openzeppelin-contracts) seek to provide patterns such that secure code can be re-used without duplication. Any contract security analyses must include any re-used code that has not previously established a level of trust commensurate with the funds at risk in the target smart contract system.

Try not to reinvent the wheel. If a library or contract already exists that does most of what you need, reuse it. Within your own code, follow the DRY principle: Don’t Repeat Yourself. If you see any snippet of code repeated more than once, ask yourself whether it could be written as a function or library and reused. Code that has been extensively used and tested is likely more secure than any new code you write. Beware of “Not Invented Here” syndrome, where you are tempted to "improve" a feature or component by building it from scratch. The security risk is often greater than the improvement value.

#### Contract Libraries

There is a lot of existing code available for reuse, both deployed on-chain as callable libraries and off-chain as code template libraries. On-platform libraries, having been deployed, exist as bytecode smart contracts, so great care should be taken before using them in production. However, using well-established existing on-platform libraries comes with many advantages, such as being able to benefit from the latest upgrades, and saves you money and benefits the Ethereum ecosystem by reducing the total number of live contracts in Ethereum.

In Ethereum, the most widely used resource is the [OpenZeppelin suite](https://openzeppelin.org/), an ample library of contracts ranging from implementations of ERC20 and ERC721 tokens, to many flavors of crowdsale models, to simple behaviors commonly found in contracts, such as Ownable, Pausable, or LimitBalance. The contracts in this repository have been extensively tested and in some cases even function as de facto standard implementations. They are free to use, and are built and maintained by Zeppelin together with an ever-growing list of external contributors.



## Resources

* [https://consensys.github.io/smart-contract-best-practices/](https://consensys.github.io/smart-contract-best-practices/)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/](https://consensys.github.io/smart-contract-best-practices/recommendations/)
* [https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#security-best-practices](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#security-best-practices)

## 

