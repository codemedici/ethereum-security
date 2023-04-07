# Common Design Patterns

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
