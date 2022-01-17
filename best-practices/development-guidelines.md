# Development Guidelines

## General Philosophy

Ethereum and complex blockchain programs are new and highly experimental. Therefore, you should expect constant changes in the security landscape, as new bugs and security risks are discovered, and new best practices are developed. Following the security practices in this document is therefore only the beginning of the security work you will need to do as a smart contract developer.

Smart contract programming requires a different engineering mindset than you may be used to. The cost of failure can be high, and change can be difficult, making it in some ways more similar to hardware programming or financial services programming than web or mobile development. It is therefore not enough to defend against known vulnerabilities.&#x20;

Defensive programming is a style of programming that is particularly well suited to smart contracts. It emphasizes the following, all of which are best practices:

### Minimalism/simplicity

Complexity is the enemy of security. The simpler the code, and the less it does, the lower the chances are of a bug or unforeseen effect occurring. When first engaging in smart contract programming, developers are often tempted to try to write a lot of code. Instead, you should look through your smart contract code and try to find ways to do less, with fewer lines of code, less complexity, and fewer "features." If someone tells you that their project has produced "thousands of lines of code" for their smart contracts, you should question the security of that project. Simpler is more secure.

Complexity increases the likelihood of errors.

* Ensure the contract logic is simple
* Modularize code to keep contracts and functions small
* Use already-written tools or code where possible (eg. don't roll your own random number generator)
* Prefer clarity to performance whenever possible
* Only use the blockchain for the parts of your system that require decentralization

### Code quality

Smart contract code is unforgiving. Every bug can lead to monetary loss. You should not treat smart contract programming the same way as general-purpose programming. Writing DApps in Solidity is not like creating a web widget in JavaScript. Rather, you should apply rigorous engineering and software development methodologies, as you would in aerospace engineering or any similarly unforgiving discipline. Once you "launch" your code, there’s little you can do to fix any problems.

### Readability/auditability

Your code should be clear and easy to comprehend. The easier it is to read, the easier it is to audit. Smart contracts are public, as everyone can read the bytecode and anyone can reverse-engineer it. Therefore, it is beneficial to develop your work in public, using collaborative and open source methodologies, to draw upon the collective wisdom of the developer community and benefit from the highest common denominator of open source development. You should write code that is well documented and easy to read, following the style and naming conventions that are part of the Ethereum community.

### Test coverage

Test everything that you can. Smart contracts run in a public execution environment, where anyone can execute them with whatever input they want. You should never assume that input, such as function arguments, is well formed, properly bounded, or has a benign purpose. Test all arguments to make sure they are within expected ranges and properly formatted before allowing execution of your code to continue.

### Prepare for failure

Any non-trivial contract will have errors in it. Your code must, therefore, be able to respond to bugs and vulnerabilities gracefully.

* Pause the contract when things are going wrong ('circuit breaker')
* Manage the amount of money at risk (rate limiting, maximum usage)
* Have an effective upgrade path for bugfixes and improvements

### Rollout carefully

It is always better to catch bugs before a full production release.

* Test contracts thoroughly, and add tests whenever new attack vectors are discovered
* Provide [bug bounties](https://consensys.github.io/smart-contract-best-practices/software\_engineering/#bug-bounty-programs) starting from alpha testnet releases
* Rollout in phases, with increasing usage and testing in each phase

### Stay up to date

Keep track of new security developments.

* Check your contracts for any new bug as soon as it is discovered
* Upgrade to the latest version of any tool or library as soon as possible
* Adopt new security techniques that appear useful

### Be aware of blockchain properties

While much of your programming experience will be relevant to Ethereum programming, there are some pitfalls to be aware of.

* Be extremely careful about external contract calls, which may execute malicious code and change control flow.
* Understand that your public functions are public, and may be called maliciously and in any order. The private data in smart contracts is also viewable by anyone.
* Keep gas costs and the block gas limit in mind.
* Be aware that timestamps are imprecise on a blockchain, miners can influence the time of execution of a transaction within a margin of several seconds.
* Randomness is non-trivial on blockchain, most approaches to random number generation are gameable on a blockchain.

### Fundamental Tradeoffs: Simplicity versus Complexity cases

There are multiple fundamental tradeoffs to consider when assessing the structure and security of a smart contract system. The general recommendation for any smart contract system is to identify the proper balance for these fundamental tradeoffs.

An ideal smart contract system from a software engineering bias is modular, reuses code instead of duplicating it, and supports upgradeable components. An ideal smart contract system from a secure architecture bias may share this mindset, especially in the case of more complex smart contract systems.

However, there are important exceptions where security and software engineering best practices may not be aligned. In each case, the proper balance is obtained by identifying the optimal mix of properties along contract system dimensions such as:

* Rigid versus Upgradeable
* Monolithic versus Modular
* Duplication versus Reuse

#### Rigid versus Upgradeable

While multiple resources, including this one, emphasize malleability characteristics such as Killable, Upgradeable or Modifiable patterns there is a _fundamental tradeoff_ between malleability and security.

Malleability patterns by definition add complexity and potential attack surfaces. Simplicity is particularly effective over complexity in cases where the smart contract system performs a very limited set of functionality for a pre-defined limited period of time, for example, a governance-free finite-time-frame token-sale contract system.

#### Monolithic versus Modular

A monolithic self-contained contract keeps all knowledge locally identifiable and readable. While there are few smart contract systems held in high regard that exist as monoliths, there is an argument to be made for extreme locality of data and flow - for example, in the case of optimizing code review efficiency.

As with the other tradeoffs considered here, security best practices trend away from software engineering best practices in simple short-lived contracts and trend toward software engineering best practices in the case of more complex perpetual contract systems.

#### Duplication versus Reuse

A smart contract system from a software engineering perspective wishes to maximize reuse where reasonable. There are many ways to reuse contract code in Solidity. Using proven previously-deployed contracts _which you own_ is generally the safest manner to achieve code reuse.

Duplication is frequently relied upon in cases where self-owned previously-deployed contracts are not available. Efforts such as [OpenZeppelin's Solidity Library](https://github.com/OpenZeppelin/openzeppelin-contracts) seek to provide patterns such that secure code can be re-used without duplication. Any contract security analyses must include any re-used code that has not previously established a level of trust commensurate with the funds at risk in the target smart contract system.

Try not to reinvent the wheel. If a library or contract already exists that does most of what you need, reuse it. Within your own code, follow the DRY principle: Don’t Repeat Yourself. If you see any snippet of code repeated more than once, ask yourself whether it could be written as a function or library and reused. Code that has been extensively used and tested is likely more secure than any new code you write. Beware of “Not Invented Here” syndrome, where you are tempted to "improve" a feature or component by building it from scratch. The security risk is often greater than the improvement value.

#### Contract Libraries

There is a lot of existing code available for reuse, both deployed on-chain as callable libraries and off-chain as code template libraries. On-platform libraries, having been deployed, exist as bytecode smart contracts, so great care should be taken before using them in production. However, using well-established existing on-platform libraries comes with many advantages, such as being able to benefit from the latest upgrades, and saves you money and benefits the Ethereum ecosystem by reducing the total number of live contracts in Ethereum.

In Ethereum, the most widely used resource is the [OpenZeppelin suite](https://openzeppelin.org), an ample library of contracts ranging from implementations of ERC20 and ERC721 tokens, to many flavors of crowdsale models, to simple behaviors commonly found in contracts, such as Ownable, Pausable, or LimitBalance. The contracts in this repository have been extensively tested and in some cases even function as de facto standard implementations. They are free to use, and are built and maintained by Zeppelin together with an ever-growing list of external contributors.

## Secure Development Workflow

Here's a high-level process we recommend following while you write your smart contracts.

Check for known security issues:

* &#x20;Review your contracts with [Slither](https://github.com/crytic/slither). It has more than 70 built-in detectors for common vulnerabilities. Run it on every check-in with new code and ensure it gets a clean report (or use triage mode to silence certain issues).

Consider special features of your contract:

* &#x20;Are your contracts upgradeable? Review your upgradeability code for flaws with [`slither-check-upgradeability`](https://github.com/crytic/slither/wiki/Upgradeability-Checks) or [Crytic](https://blog.trailofbits.com/2020/06/12/upgradeable-contracts-made-safer-with-crytic/). We've documented 17 ways upgrades can go sideways.
* &#x20;Do your contracts purport to conform to ERCs? Check them with [`slither-check-erc`](https://github.com/crytic/slither/wiki/ERC-Conformance). This tool instantly identifies deviations from six common specs.
* &#x20;Do you have unit tests in Truffle? Enrich them with [`slither-prop`](https://github.com/crytic/slither/wiki/Property-generation). It automatically generates a robust suite of security properties for features of ERC20 based on your specific code.
* &#x20;Do you integrate with 3rd party tokens? Review our [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md) before relying on external contracts.

Visually inspect critical security features of your code:

* &#x20;Review Slither's [inheritance-graph](https://github.com/trailofbits/slither/wiki/Printer-documentation#inheritance-graph) printer. Avoid inadvertent shadowing and C3 linearization issues.
* &#x20;Review Slither's [function-summary](https://github.com/trailofbits/slither/wiki/Printer-documentation#function-summary) printer. It reports function visibility and access controls.
* &#x20;Review Slither's [vars-and-auth](https://github.com/trailofbits/slither/wiki/Printer-documentation#variables-written-and-authorization) printer. It reports access controls on state variables.

Document critical security properties and use automated test generators to evaluate them:

* &#x20;Learn to [document security properties for your code](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis). It's tough as first, but it's the single most important activity for achieving a good outcome. It's also a prerequisite for using any of the advanced techniques in this tutorial.
* &#x20;Define security properties in Solidity, for use with [Echidna](https://github.com/crytic/echidna) and [Manticore](https://manticore.readthedocs.io/en/latest/verifier.html). Focus on your state machine, access controls, arithmetic operations, external interactions, and standards conformance.
* &#x20;Define security properties with [Slither's Python API](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither). Focus on inheritance, variable dependencies, access controls, and other structural issues.
* &#x20;Run your property tests on every commit with [Crytic](https://crytic.io). Crytic can consume and evaluate security property tests so everyone on your team can easily see that they pass on GitHub. Failing tests can block commits.

Finally, be mindful of issues that automated tools cannot easily find:

* Lack of privacy: everyone else can see your transactions while they're queued in the pool
* Front running transactions
* Cryptographic operations
* Risky interactions with external DeFi components

## Design guidelines

The design of the contract should be discussed ahead of time, prior writing any line of code.

### Documentation and specifications

Documentation can be written at different levels, and should be updated while implementing the contracts:

* **A plain English description of the system**, describing what the contracts do and any assumptions on the codebase.
* **Schema and architectural diagrams**, including the contract interactions and the state machine of the system. [Slither printers](https://github.com/crytic/slither/wiki/Printer-documentation) can help to generate these schemas.
* **Thorough code documentation**, the [Natspec format](https://solidity.readthedocs.io/en/develop/natspec-format.html) can be used for Solidity.

### On-chain vs off-chain computation

* **Keep as much code as you can off-chain.** Keep the on-chain layer small. Pre-process data with code off-chain in such a way that verification on-chain is simple. Do you need an ordered list? Sort the list offchain, then only check its order onchain.

### Upgradeability

We discussed the different upgradeability solutions in [our blogpost](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/). Make a deliberate choice to support upgradeability or not prior to writing any code. The decision will influence how you structure our code. In general, we recommend:

* **Favoring** [**contract migration**](https://blog.trailofbits.com/2018/10/29/how-contract-migration-works/) **over upgradeability.** Migration system have many of the same advantages than upgradeable, without their drawbacks.
* **Using the data separation pattern over the delegatecallproxy one.** If your project has a clear abstraction separation, upgradeability using data separation will necessitate only a few adjustments. The delegatecallproxy requires EVM expertise and is highly error-prone.
* **Document the migration/upgrade procedure before the deployment.** If you have to react under stress without any guidelines, you will make mistakes. Write the procedure to follow ahead of time. It should include:
  * The calls that initiate the new contracts
  * Where are stored the keys and how to access them
  * How to check the deployment! Develop and test a post-deployment script.

## Common Design Patterns <a href="#common-design-patterns" id="common-design-patterns"></a>

The patterns outlined below are widely used to solve common problems or provide useful functionality. Ultimately the design is a decision of the developer / owner of the smart contracts, but for common problems these patterns are advisable. Developers should favor widely-used patterns that have received the most scrutiny from the community and have been tested in the harshest environment.

### Checks, Effects, Interaction <a href="#checks-effects-interaction" id="checks-effects-interaction"></a>

This pattern arose from a painful episode: The DAO hack, in July 2016. As the name suggests, three concerns should be addressed in a specific order.

1. Acceptability checks should come first.
2. Second, a function must perform state changes to put its storage state in order **optimistically**. This includes event emitters.
3. Always lastly, external calls to other contracts.

Implicitly, if step 3 fails for any reason, then the state changes from step 2 must be unwound - usually by reverting the transaction.

This simple pattern prevents reentrance by making it pointless.

This approach is not always possible, as state changes could be dependent on external calls. In this case, the creation of a reentrancy guard such as below (from Open Zeppelin) is mandatory.

```
pragma solidity ^0.4.24;

/**
 * @title Helps contracts guard against reentrancy attacks.
 * @author Remco Bloemen <remco@2π.com>, Eenae <alexey@mixbytes.io>
 * @dev If you mark a function `nonReentrant`, you should also
 * mark it `external`.
 */
contract ReentrancyGuard {

    /// @dev counter to allow mutex lock with only one SSTORE operation
    uint256 private guardCounter = 1;

    /**
     * @dev Prevents a contract from calling itself, directly or indirectly.
     * If you mark a function `nonReentrant`, you should also
     * mark it `external`. Calling one `nonReentrant` function from
     * another is not supported. Instead, you can implement a
     * `private` function doing the actual work, and an `external`
     * wrapper marked as `nonReentrant`.
     */
    modifier nonReentrant() {
        guardCounter += 1;
        uint256 localCounter = guardCounter;
        _;
        require(localCounter == guardCounter);
  }

}
```

### Pull payments <a href="#pull-payments" id="pull-payments"></a>

Quite often there'll be a need to send ETH to third parties during the flow of a contract and, for this, there are basically two approaches: push or pull payments. In the first, the contract actively sends the transaction and in the latter the user must take an action to withdraw it.

Push payments can be a source of a few issues, like DoS attacks, whenever the push function is restricted to a specific user, say an owner, and they can simply refuse to call it. Another aspect of DoS is on the receiving end, where receivers purposely fail incoming transactions to keep the caller contract in a locked state.

This issue is amplified, when dealing with multiple payments in the same transaction, because a single fail will revert the whole call. There's also the possibility of the payment chain growing too large, or even opcode costs increasing so much, that a payment chain isn't able to fit into a single block anymore, rendering the mechanism useless.

In a similar note, push payments have the side effect of delegating all gas costs to the contract, whereas in pull payments, the gas cost are diluted among several participants.

For the reasons above the recommended design is to instead have the users pull the Ether / tokens individually through by calling a function.

### Access control <a href="#access-control" id="access-control"></a>

The owner pattern is still the most used, with variations for different clearances and numbers of administrators within a dapp. Role-based access control is gaining popularity in more complex situations.

Access controls are vital to many designs, but they should be avoided if not strictly necessary. Always review the design used and confirm appropriate access rights for every function. Protected functions, unlike almost any other smart contract pattern, brings up a few issues regarding the handling of private keys off chain, since a traditional hack os misbehave can abuse the privileges. That issue lies on the edge of what is considered an auditor's responsibility or not, but if you encounter a contract that has a user with high power, it's always good practice to also recommend the project to use secure solutions, like an audited and widely used multisigs.

It's also good to report what are the consequences if a privileged account doesn't take the actions that it's required of it, for example, setting up correct values or triggering a specific switch.

Consider also the _effects_ of restricted functions because they usually imply privileges for certain users. And privilege is often a form of centralization. For example, consider a token that allows its owner to transfer user balances. That would be a re-introduction of centralization, by a centralized authority, that should be reported - that such a possibility exists could be a surprise to the users, so it should not go unreported in the audit report. Remember the audience relies on audit reports to avoid surprise.

Questionable functions may be implemented by design and with good intentions. Our task is to consider the worst-case malicious misuse and explain the risk. This is so users can make informed judgements about the risk, whom or what they are required to trust, and the benefits (intended properties) of the design.

### Upgradeability <a href="#upgradeability" id="upgradeability"></a>

Upgradeability, as in access control, is sometimes desirable for complex applications, but it should be avoided when possible. Generally, contracts meant for one time use (such as a crowdsale), or for absolute trustlessness (a well designed token) should not be upgradeable. Upgradeability undermines immutability. When implemented as a trustful process where one party has the privilege of upgrading, upgradeability casts doubt on all assurances about the contract in question.

Upgradeability can take other forms. For example, in a trustless upgrade pattern, no user can force another user to accept a new implementation. Still another pattern, contract factories that produce nodes or instance contracts can be subject to periodic upgrades. By extension, output contracts themselves can follow amended templates. Such patterns are popular and may provide a rolling upgrade without compromising immutability.

Trustful upgrade patterns are vulnerable to severe consequences if a privileged key is compromised. The smart contracts could be upgraded to a rogue version.

Upgradeable contracts usually make use of low-level calls that can lead to unexpected effects. Pay particular attention to hidden assumptions about data storage location whenever you see proxy contracts and novel types of message forwarding.

Also note that functions declared as internal in libraries are copied to the smart contract at compile time. Such functions will not be upgradeable. This can cause confusion - a contract successfully pointing to an updated library but behaving like the old library.

When auditing, check the design of upgradeability for vulnerabilities. Focus on low-level calls, who and what situations allow for changes, and restrictions - for example, not allowed at certain stages of the contract lifecycle such as when voting is open. Also note there are several designs in use as the community has not yet settled on best practices. If the design uses a single contract to store data / state, check the possibility of direct access / tampering of data and the impact of a state shared between two or more contracts.

### Privacy - Commit and Reveal <a href="#privacy---commit-and-reveal" id="privacy---commit-and-reveal"></a>

If privacy over a piece of information is required, it should be encrypted off chain because everything stored and all processes on the chain are visible to determined adversaries.

Premature exposure of secret information can overturn assumptions about the fairness of process in the same way that peeking at an opponent’s cards alters the fairness of a card game. Indeed, it may alter the balance of incentives to the extent that the entire system becomes unviable, meaning unable to function as designed.

The Commit and Reveal pattern is the usual approach to these types of problems. Users post a hash of their entry and financially (or materially) commit. In a second stage, results are determined by submitting solutions to the committed hashes.

_Note that if the options are small (a small value range or rock, paper, scissors for example) this scheme is easy to brute force, and therefore requires the addition of a large secret value to provide entropy and further obfuscate the committed choice)._

### Pause (Circuit Breakers) <a href="#pause-circuit-breakers" id="pause-circuit-breakers"></a>

Circuit-breaker functionality (or pause) has the goal of halting an ongoing attack as soon as it is noticed, or pausing operations if the smart contract shows signs of unintended behavior.

The vast majority rely on manual interaction from the owner of the contract. In these cases the auditor has to pay close attention to how it could be used, and document that in the report. As with all controls, the risk and impact of it being used with malicious intentions should be evaluated and weighed against the benefits of the control itself.

Consider a token contract. Pause functionality could deter an attack on a token, but it could also facilitate a full denial of service with centralized control. Not a very effective solution, and definitely not worth the risk of the creator of such a powerful attack vector.

Automated circuit breakers can also be helpful in more complex contracts but great care is required to prevent intentionally freezing the contract with unexpected parameters.

### Fallback functions <a href="#fallback-functions" id="fallback-functions"></a>

Keep the logic inside the fallback function simple. Contracts that transfer Ether using `.send` or `.transfer` will provide a very low gas amount (2300) and will cause the transaction to revert if anything more than firing an event happens.

For complex contracts, especially those that are upgradeable, it is a good practice to check the data of the transaction inside the fallback function. The reason for this is that if a user calls a function that does not exist in the current version of the contract, the fallback will be executed, accepting Ether if it is `payable` while completely ignoring the remainder of the transaction data.

```
function() payable {
    require(msg.data.length == 0);
}
```

### Speed bumps <a href="#speed-bumps" id="speed-bumps"></a>

A speed bump is logic that prevents some actions within the smart contract with the goal of allowing time for the owners and other users of a contract to notice if anything wrong is happening.

In the now-famous The DAO hack a feature of this nature prevented even greater losses. It had a speed bump that required a month between the ordering and the actual execution of a withdrawal (check).

Speed bumps can prevent greater losses and should be considered when dealing with large sums of currency.

### Restricting Ether amount <a href="#restricting-ether-amount" id="restricting-ether-amount"></a>

Limiting the amount of Ether (and tokens) a contract can handle is a great way to limit the impact of a possible vulnerability, while also discouraging an attacker due to smaller rewards.

A common, and recommended, pattern is to use some sort of vault contract, usually a simpler contract which is kept separate from the main logic with a smaller attack surface, preferably with an already tested solution.

### Assert vs Require <a href="#assert-vs-require" id="assert-vs-require"></a>

Assert and require will behave differently in the future and should be used accordingly. Assert should be used to check invariants for conditions that should hold true in all cases and exists to enable static analysis tools to check for unintended behavior.

Require, on the other hand, should be used for other checks such as input verification. It returns the remaining gas to users, and since Solidity (0.4.22) allows for the inclusion of an error message that can help users understand what went wrong.

## Implementation guidelines

**Strive for simplicity.** Always use the simplest solution that fits your purpose. Any member of your team should be able to understand your solution.

### Function composition

The architecture of your codebase should make your code easy to review. Avoid architectural choices that decrease the ability to reason about its correctness.

* **Split the logic of your system**, either through multiple contracts or by grouping similar functions together (for example, authentification, arithmetic, ...).
* **Write small functions, with a clear purpose.** This will facilitate easier review and allow the testing of individual components.

### Inheritance

* **Keep the inheritance manageable.** Inheritance should be used to divide the logic, however, your project should aim to minimize the depth and width of the inheritance tree.
* **Use Slither’s** [**inheritance printer**](https://github.com/crytic/slither/wiki/Printer-documentation#inheritance-graph) **to check the contracts’ hierarchy.** The inheritance printer will help you review the size of the hierarchy.

### Events

* **Log all crucial operations.** Events will help to debug the contract during the development, and monitor it after deployment.

### Avoid known pitfalls

* **Be aware of the most common security issues.** There are many online resources to learn about common issues, such as [Ethernaut CTF](https://ethernaut.openzeppelin.com), [Capture the Ether](https://capturetheether.com), or [Not so smart contracts](https://github.com/crytic/not-so-smart-contracts/).
* **Be aware of the warnings sections in the** [**Solidity documentation**](https://solidity.readthedocs.io/en/latest/)**.** The warnings sections will inform you about non-obvious behavior of the language.

### Dependencies

* **Use well-tested libraries.** Importing code from well-tested libraries will reduce the likelihood that you write buggy code. If you want to write an ERC20 contract, use [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/contracts/token/ERC20).
* **Use a dependency manager; avoid copy-pasting code.** If you rely on an external source, then you must keep it up-to-date with the original source.

### Testing and verification

* **Write thorough unit-tests.** An extensive test suite is crucial to build high-quality software.
* **Write** [**Slither**](https://github.com/crytic/slither)**,** [**Echidna**](https://github.com/crytic/echidna) **and** [**Manticore**](https://github.com/trailofbits/manticore) **custom checks and properties.** Automated tools will help ensure your contract is secure. Review the rest of this guide to learn how to write efficient checks and properties.
* **Use** [**crytic.io**](https://crytic.io)**.** Crytic integrates with Github, provides access to private Slither detectors, and runs custom property checks from Echidna.

### Solidity

* **Favor Solidity 0.5 or 0.6.** In our opinion, Solidity 0.5 and 0.6 are more secure and have better built-in practices than 0.4. Solidity 0.7 is too young to be used in production and needs time to mature.
* **Use a stable release to compile; use the latest release to check for warnings.** Check that your code has no reported issues with the latest compiler version. However, Solidity has a fast release cycle and has a history of compiler bugs, so we do not recommend the latest version for deployment (see Slither’s [solc version recommendation](https://github.com/crytic/slither/wiki/Detector-Documentation#recommendation-39)).
* **Do not use inline assembly.** Assembly requires EVM expertise. Do not write EVM code if you have not _mastered_ the yellow paper.

## Deployment guidelines

Once the contract has been developed and deployed:

* **Monitor your contracts.** Watch the logs, and be ready to react in case of contract or wallet compromise.
* **Add your contact info to** [**blockchain-security-contacts**](https://github.com/crytic/blockchain-security-contacts)**.** This list helps third-parties contact you if a security flaw is discovered.
* **Secure the wallets of privileged users.** Follow our [best practices](https://blog.trailofbits.com/2018/11/27/10-rules-for-the-secure-use-of-cryptocurrency-hardware-wallets/) if you store keys in hardware wallets.
* **Have a response to incident plan.** Consider that your smart contracts can be compromised. Even if your contracts are free of bugs, an attacker may take control of the contract owner's keys.

## Resources

* [https://consensys.github.io/smart-contract-best-practices/](https://consensys.github.io/smart-contract-best-practices/)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/](https://consensys.github.io/smart-contract-best-practices/recommendations/)
* [https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#security-best-practices](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#security-best-practices)
* [Secure Development Workflow](https://ethereum.org/bg/developers/tutorials/secure-development-workflow/)

###

