# Program Analysis and Tools

We are going use three distinctive testing and program analysis techniques:

* **Static analysis with** [**Slither**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither)**.** All the paths of the program are approximated and analyzed at the same time, through different program presentations (e.g. control-flow-graph)
* **Fuzzing with** [**Echidna**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna)**.** The code is executed with a pseudo-random generation of transactions. The fuzzer will try to find a sequence of transactions to violate a given property.
* **Symbolic execution with** [**Manticore**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore)**.** A formal verification technique, which translates each execution path to a mathematical formula, on which on top constraints can be checked.

Each technique has advantages and pitfalls, and will be useful in [specific cases](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis#determining-security-properties):

| Technique          | Tool      | Usage                         | Speed   | Bugs missed | False Alarms |
| ------------------ | --------- | ----------------------------- | ------- | ----------- | ------------ |
| Static Analysis    | Slither   | CLI & scripts                 | seconds | moderate    | low          |
| Fuzzing            | Echidna   | Solidity properties           | minutes | low         | none         |
| Symbolic Execution | Manticore | Solidity properties & scripts | hours   | none\*      | none         |

\* if all the paths are explored without timeout

**Slither** analyzes contracts within seconds, however, static analysis might lead to false alarms and will be less suitable for complex checks (e.g. arithmetic checks). Run Slither via the API for push-button access to built-in detectors or via the API for user-defined checks.

**Echidna** needs to run for several minutes and will only produce true positives. Echidna checks user-provided security properties, written in Solidity. It might miss bugs since it is based on random exploration.

**Manticore** performs the "heaviest weight" analysis. Like Echidna, Manticore verifies user-provided properties. It will need more time to run, but it can prove the validity of a property and will not report false alarms.

## Suggested workflow

Start with Slither's built-in detectors to ensure that no simple bugs are present now or will be introduced later. Use Slither to check properties related to inheritance, variable dependencies, and structural issues. As the codebase grows, use Echidna to test more complex properties of the state machine. Revisit Slither to develop custom checks for protections unavailable from Solidity, like protecting against a function being overridden. Finally, use Manticore to perform targeted verification of critical security properties, e.g., arithmetic operations.

* Use Slither's CLI to catch common issues
* Use Echidna to test high-level security properties of your contract
* Use Slither to write custom static checks
* Use Manticore once you want in-depth assurance of critical security properties

**A note on unit tests**. Unit tests are necessary to build high-quality software. However, these techniques are not the best suited to find security flaws. They are typically used to test positive behaviors of code (i.e. the code works as expected in the normal context), while security flaws tend to reside in edge cases that the developers did not consider. In our study of dozens of smart contract security reviews, [unit test coverage had no effect on the number or severity of security flaws](https://blog.trailofbits.com/2019/08/08/246-findings-from-our-smart-contract-audits-an-executive-summary/) we found in our client's code.

## Determining Security Properties

To effectively test and verify your code, you must identify the areas that need attention. As your resources spent on security are limited, scoping the weak or high-value parts of your codebase is important to optimize your effort. Threat modeling can help. Consider reviewing:

* [Rapid Risk Assessments](https://infosec.mozilla.org/guidelines/risk/rapid\_risk\_assessment.html) (our preferred approach when time is short)
* [Guide to Data-Centric System Threat Modeling](https://csrc.nist.gov/publications/detail/sp/800-154/draft) (aka NIST 800-154)
* [Shostack thread modeling](https://www.amazon.com/Threat-Modeling-Designing-Adam-Shostack/dp/1118809998)
* [STRIDE](https://en.wikipedia.org/wiki/STRIDE\_\(security\)) / [DREAD](https://en.wikipedia.org/wiki/DREAD\_\(risk\_assessment\_model\))
* [PASTA](https://en.wikipedia.org/wiki/Threat\_model#P.A.S.T.A.)
* [Use of Assertions](https://blog.regehr.org/archives/1091)

### Components

Knowing what you want to check will also help you to select the right tool.

The broad areas that are frequently relevant for smart contracts include:

* **State machine.** Most contracts can be represented as a state machine. Consider checking that (1) No invalid state can be reached, (2) if a state is valid that it can be reached, and (3) no state traps the contract.
  * Echidna and Manticore are the tools to favor to test state-machine specifications.
* **Access controls.** If you system has privileged users (e.g. an owner, controllers, ...) you must ensure that (1) each user can only perform the authorized actions and (2) no user can block actions from a more priviledged user.
  * Slither, Echidna and Manticore can check for correct access controls. For example, Slither can check that only whitelisted functions lack the onlyOwner modifier. Echidna and Manticore are useful for more complex access control, such as a permission given only if the contract reaches a given state.
* **Arithmetic operations.** Checking the soundness of the arithmetic operations is critical. Using `SafeMath` everywhere is a good step to prevent overflow/underflow, however, you must still consider other arithmetic flaws, including rounding issues and flaws that trap the contract.
  * Manticore is the best choice here. Echidna can be used if the arithmetic is out-of-scope of the SMT solver.
* **Inheritance correctness.** Solidity contracts rely heavily on multiple inheritance. Mistakes such as a shadowing function missing a `super` call and misinterpreted c3 linearization order can easily be introduced.
  * Slither is the tool to ensure detection of these issues.
* **External interactions.** Contracts interact with each other, and some external contracts should not be trusted. For example, if your contract relies on external oracles, will it remain secure if half the available oracles are compromised?
  * Manticore and Echidna are the best choice for testing external interactions with your contracts. Manticore has an built-in mechanism to stub external contracts.
* **Standard conformance.** Ethereum standards (e.g. ERC20) have a history of flaws in their design. Be aware of the limitations of the standard you are building on.
  * Slither, Echidna, and Manticore will help you to detect deviations from a given standard.

### Tool selection cheatsheet

| Component               | Tools                       | Examples                                                                                                                                                                                                                                                        |
| ----------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| State machine           | Echidna, Manticore          |                                                                                                                                                                                                                                                                 |
| Access control          | Slither, Echidna, Manticore | [Slither exercise 2](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither/exercise2.md), [Echidna exercise 2](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-2.md)       |
| Arithmetic operations   | Manticore, Echidna          | [Echidna exercise 1](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-1.md), [Manticore exercises 1 - 3](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/manticore/exercises) |
| Inheritance correctness | Slither                     | [Slither exercise 1](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither/exercise1.md)                                                                                                                                     |
| External interactions   | Manticore, Echidna          |                                                                                                                                                                                                                                                                 |
| Standard conformance    | Slither, Echidna, Manticore | [`slither-erc`](https://github.com/crytic/slither/wiki/ERC-Conformance)                                                                                                                                                                                         |

Other areas will need to be checked depending on your goals, but these coarse-grained areas of focus are a good start for any smart contract system.

Our public audits contain examples of verified or tested properties. Consider reading the `Automated Testing and Verification` sections of the following reports to review real-world security properties:

* [0x](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
* [Balancer](https://github.com/trailofbits/publications/blob/master/reviews/BalancerCore.pdf)

## Looking For Code Smells <a href="#looking-for-code-smells" id="looking-for-code-smells"></a>

Once you have a general understanding of the contracts behavior, it's time to start really looking for flaws. This is a general list of code smells, which are implementation details that sometimes indicate a potential problem, but aren't necessarily wrong per se. Not all problems give off a smell, but following your nose often pays off.

### Arrays <a href="#arrays" id="arrays"></a>

Arrays are very commonly used in most other areas of software development, but they can be a bit tricky to implement in a smart contracts setting.

Here are a few questions to ask yourself as you analyze the usage of arrays:

1. Is it bounded?\
   If an array can grow indefinitely, it might become a problem due to gas limits when one iterates over it.
2. What happens if there are duplicates?\
   Another rather common source of errors is the presence of duplicated items in the array. Can the system handle duplicates, or does it lead to an unwanted state?
3. What happens if there are gaps?\
   Deleting an element might leave a gap in the middle of the array , which some systems don't anticipate for.
4. Arrays together with mappings.\
   A pattern that a lot of contracts use is a combination of mappings and arrays, to have the best of each data structure. When this happens take a closer look as to how these structures are tied together, especially at the zero index, since the default value for a mapping is zero.
5. Iterations\
   Does the iteration cover the entirety of the array? Quite often loops can miss an index or even go one step further and reach an out-of-bounds reference.

### Error handling <a href="#error-handling" id="error-handling"></a>

Pay attention to the way smart contracts handle errors. Some contracts revert transactions while others will use a less intrusive approach and return `false`. This isn't usually very problematic, but it can be in cases of external calls and how the foreign contract calls it. It may be impossible to know, such as cases where the callee is not known in advance. The caller must be able to handle each scenario.

Here's a [detailed article](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) with some ERC20 examples.

### Rounding Errors and Precision <a href="#rounding-errors-and-precision" id="rounding-errors-and-precision"></a>

When auditing always check the precision against which all calculations are performed, and make sure that it's enough for the given specification.

Pay close attention to error propagation from arithmetic operations. Since Solidity does not support floats, it rounds numbers down, which is not a problem in itself, but might become when dealing with a chain of operations. Strictly speaking, it is always an error to create unaccounted-for “dust” from rounding errors. Contracts should account for everything.

### Points of centralization <a href="#points-of-centralization" id="points-of-centralization"></a>

This is a particularly interesting topic. There is no consensus within the security community with regards to when too much centralization is a bug.

Let's imagine that you are auditing a particular project and realize that a single private key is allowed to move all funds to another contract. Following that realization, you read through the specification, where it clearly states that this is the intended behavior. Should you include your findings in your report?

To answer this question, let us return to the purpose and the audience of security reports. You may find yourself under pressure from the hiring company to color the report a certain way. And, they may feel that those who pay for the work should determine the content. They may, for example, believe it is not a bug, it is a feature.

However, the audience of the report is the users of the system, now, and in the future. Be sure you understand this difference. The purpose of the work and the audience should be clear in all of your communications and contracts with the client. Your purpose is to create an objective and complete report. Proceed on the understanding that your report may be referred to by any party, for any reason, at any time. Errors and omissions could even form the basis of moving against the auditors.

It is recommended that you document concerns that could be interpreted as “features” if they contain any element of potential surprise.

Recommended [read](https://medium.com/@ameensol/what-you-should-know-before-putting-half-a-million-dai-in-compound-fafdb2645f77).

### Unnecessary Complexity <a href="#unnecessary-complexity" id="unnecessary-complexity"></a>

In smart contract development, the old KISS principle (keep it simple stupid) should be followed religiously, as each extra bit of functionality increases the surface for attacks and bugs. It's extremely hard to pinpoint what makes a piece of code too complex, but you will sense it when you see it. It is vital that contracts are clear and understandable to those who wish to study them.

Unfortunately there isn't an objective measure of unnecessary complexity. In addition to specific vulnerabilities you do find, document identified opportunities to simplify the code or the processes. If complexity gives you a sense of discomfort about the possibility of undetected vulnerabilities, document your concern. For example, it may be difficult to reason about every flow in an especially convoluted process. Say so. Suggest opportunities to simplify.

Overly complex code is of course more prone to bugs, so you should spend your time (a limited resource after all) diligently, and focus particularly on these sections. It may be helpful to keep superficially-apparent complexity in mind when considering time estimates before you start.

### Collusion <a href="#collusion" id="collusion"></a>

In the realm of game-theoretical attacks, although not very common, one piece that is good to keep an eye on is the concept of collusion. It's really hard to design systems that are completely foolproof against bad actors, and this becomes even harder when we consider that multiple parties can be acting badly together. This goes even further when we consider that smart contracts allow bad participants to work together without necessarily having to trust each other.

Vitalik wrote an [extensive piece](https://vitalik.ca/general/2019/04/03/collusion.html) on the topic, which is worth a read.

### Gas Hogs <a href="#gas-hogs" id="gas-hogs"></a>

Gas usage is certainly not a vulnerability, but it might be helpful to point out instances where it could go out of hand. There's always the possibility of a specific transaction being too big to fit inside a block, which in turn could block specific processes in a project.

Similarly, a lot of times, you'll come across operations that could be simplified or made cheaper, which is good to point out in the report.





## ECDSA <a href="#ecdsa" id="ecdsa"></a>

This is a heavy cryptographic topic but a very important one. Elliptic Curve Signatures are becoming quite common in smart contracts, with, for example, meta transaction implementations, whereby a third party submits transactions and pays for gas on behalf of users. Although very powerful, there are dangers auditors should look for:

The signature consists of `r`,`s` and `v` values and when those values are given as parameters to `ecrecover`, together with a message hash, `ecrecover` returns the address that signed the message. This allows for on-chain verification of transactions signed offline.

### 1. Replay attacks <a href="#1.-replay-attacks" id="1.-replay-attacks"></a>

Replay is possible when a contract receiving a signed message doesn't confirm each transaction is unique. That is, that each transaction has never been seen and has never been executed in the past. When that step is overlooked, a third party can re-submit transactions at their leisure, which amounts to impersonating the user, which will lead to disastrous results.

### 2. Multi-chain replay attacks <a href="#2.-multi-chain-replay-attacks" id="2.-multi-chain-replay-attacks"></a>

This is similar to simple replay attacks, except it can happen across networks. Most projects have deployed contracts on testnet and occasionally it is possible to harvest a signature from one of those networks and replay it on the mainnet.

There are a few ways to prevent this. One can include the address of the contract in the hash, provided contract instances on different networks do indeed have different addresses, which is not always the case. A more direct method is to include `networkId` in the hash which renders transactions useless on other networks.

### 3. Signature Malleability <a href="#3.-signature-malleability" id="3.-signature-malleability"></a>

This happens because of a particular property of elliptic curve signatures and it's particularly complicated to get to bottom of it without diving into pretty hard math and cryptography, but we'll try to provide an overview and point to what you specifically need to worry about as an auditor.

Like we said before, a valid Ethereum signature consists of a tuple of `{ r1, s1 , v1 }` values, but interestingly we can fabricate a second tuple, say `{ r1, s2, v1 }` that is also valid, and the condition is for `s2` to be the inverse of `s1`.

Just as a curiosity, inverting `s1` to calculate `s2` isn't as simple as multiplying by `-1` like in traditional math, since both `s1` and `s2` refer to points in a curve. The actual formula is:

```
s2 = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1;
```

Where `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141` represents the order of the elliptic curve used by Ethereum, `secp256k1`.

The important takeaway is this. If a contract does not enforce the `s` value to be on the lower range, meaning that it's smaller than `0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0`, it is possible for a third party to craft another valid signature for the same hash and public key, without the access to the signing private key.

This can be a serious issue, especially if the contract is storing signatures as unique identifiers or tracking already used signatures as a replay prevention.

Refer to[Open Zeppelin\`s](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/cryptography/ECDSA.sol) implementation as a primer on how to verify signatures. The line `57` is where the contract protects against malleable signatures. It's a good practice to use this library and follow the most used advice in cryptography: "Don't implement your own crypto."

### 4. Contracts must do the hashing <a href="#4.-contracts-must-do-the-hashing" id="4.-contracts-must-do-the-hashing"></a>

A contract that relies on the sender to provide a hash of a message that it purportedly signed is insecure. A contract must compute the hash of the message itself and then use that hash to check a signature. Let us break that down:

* The message: regardless of the purpose of the message, for a contract to process it, the contract must therefore have access to the important field values, which we collective refer to as “the message” in this context.
* The hash of the message: this is a hash of “the message.” Given that a contract **must know** the content of a message to process, it must therefore be capable of computing the hash independently. It follows that it doesn’t rely on an input to work out the hash of the message.
* Check a signature: `ecrecover` uses a “hash” and signature parameters to identify the signer of the hash. After hashing the message independently, the contract determines that the hash was signed by the expected party. This proves that the message was indeed signed by the expected party.

If a contract relies on the sender to provide the hash that it should be computing itself, then a sender can craft a hash that will mislead the contract. Here's a demonstration:

```
contract ExampleVerifier {

    using ECDSA for bytes32;

    //Use this contract to verify that a given address has signed a message containing a secret number with its private key.
    function secureVerify(uint256 secretNumber, address signer, bytes memory signature) public returns(bool) {
        //Here we make sure that the value passed correctly hashs to 'messageHash'
        bytes32 messageHash = keccak256(abi.encodePacked(secretNumber));
        address realSigner = messageHash.recover(signature);
        return realSigner == signer;
    }

    function insecureVerify(uint256 secretNumber, bytes32 messageHash, address signer, bytes memory signature) public returns(bool) {
        //An attacker can create some messageHash that passes this check, that don't necessarily resolves to the hash of secretNumber
        address realSigner = messageHash.recover(signature);
        return realSigner == signer;
    }

}
```

### 5. Payload malleability <a href="#5.-payload-malleability" id="5.-payload-malleability"></a>

Keep an eye on the payload being hashed to check the signature. Payloads will often be packed using Solidity's `abi.encodePacked`, and can lead to the same result, even though different data is being passed. This happens mostly because `encodePacked` does not pack the length of dynamic types, as the less common `abi.encode`, and this could cause a valid signature to be set for a very different input than the original, for example:

`abi.encodePacked([ 1, 2, 3 ], [ 4, 5, 6 ])` will generate the same result as `abi.encodePacked([ 1, 2, 3, 4, 5 ], [ 6 ])`.

This can cause a number of unintended consequences, depending on what the smart contract is implementing, always keep an eye on these implementations, sometimes just changing the order of parameters can remove the possibility of malleability.

* [A closer look at Ethereum Signatures](https://hackernoon.com/a-closer-look-at-ethereum-signatures-5784c14abecc)
* [What is the math behind elliptic curve cryptography](https://hackernoon.com/what-is-the-math-behind-elliptic-curve-cryptography-f61b25253da3)
* [How not to use ECDSA](https://yondon.blog/2019/01/01/how-not-to-use-ecdsa/)
* [The complement of `s` (when `s &lt;` curve order / 2)](https://bitcoin.stackexchange.com/questions/38252/the-complement-of-s-when-s-curve-order-2/38253#38253)

## Tools List

These are the tools that help to comprehend a program by displaying its visual information, like a call graph that shows each of functions executed during a specific call.

* [ethereum-graph-debugger](https://github.com/fergarrui/ethereum-graph-debugger) - A graphical EVM debugger. Displays the entire program control flow graph.
* [Slither](https://github.com/trailofbits/slither) - Slither can map method visibility and modifiers, state variables that are read and written, calls, and can print the inheritance graph of a smart contract
* [Solgraph](https://github.com/raineorshine/solgraph) - Generates DOT graphs with function control flow of a solidity contract
* [Surya](https://github.com/ConsenSys/surya) - Generates various visual outputs of function call graphs
* [sol-function-profiler](https://github.com/EricR/sol-function-profiler) - Solidity contract function profiler

### Linters

* [Remix](https://remix.ethereum.org) - Browser-based Solidity IDE with linting features
* [SmarrtCheck](https://tool.smartdec.net) - A linter for Solidity and Vyper that checks code for security issues and bad practices.
* [Solhint](https://github.com/protofire/solhint) - Linter for both security and style-guide validations. It strictly adheres to the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html).
* [Solium](https://github.com/duaraghav8/Solium) - Linter for both security and style-guide validations. Does not strictly adhere to the Solidity Style Guide.

### Bug finding tools / Static Analysis

These tools will inspect the source code for known vulnerabilities, much like you do manually. Examples are **Securify** and **SmartCheck**.

* [Manticore](https://github.com/trailofbits/manticore) - Symbolic execution tool for Ethereum smart contracts that includes detectors for common security flaws
* [Mythril OSS](https://github.com/ConsenSys/mythril/) - Open-source security analysis tool for Ethereum smart contracts built around detector modules
* [Securify](https://github.com/eth-sri/securify) - Static analysis tool from ChainSecurity. [https://securify.chainsecurity.com](https://securify.chainsecurity.com) - works on bytecode level
* [Manticore](https://github.com/trailofbits/manticore) - Detects many common bug types, and can prove correctness properties with symbolic execution.
* [Mythril](https://github.com/ConsenSys/mythril) - Security analysis tool for smart contracts.
* [ethereum/sourcify](https://github.com/ethereum/sourcify) - Re-compiler that can be used to verify that bytecode corresponds to certain source code.
* [eth-sri/securify2](https://github.com/eth-sri/securify2) - Tool for analyzing smart contracts for vulnerabilities and insecure coding.
* [Slither](https://github.com/crytic/slither) - Static analyzer with support for many common bug types, including visualization tools for security-relevant information.
* [MythX](https://mythx.io) - Detection for security vulnerabilities in Ethereum smart contracts throughout the development life cycle
* Oyente
* Maian
* Zeus
* MadMax

### Verification tools

* [KEVM](https://github.com/kframework/evm-semantics) - K Semantics of the Ethereum Virtual Machine (EVM)
* [Manticore](https://github.com/trailofbits/manticore) - Symbolic execution tool for EVM

### Reversing tools

* [abi-decompiler](https://github.com/beched/abi-decompiler) - EVM reverse engineering helper utility
* [ethereum-dasm](https://github.com/tintinweb/ethereum-dasm) - EVM disassembler with static and dynamic analysis abilities, including function signature lookup
* [Ethersplay](https://github.com/trailofbits/ethersplay) - Visual disassembler for EVM bytecode built on Binary Ninja
* [evmlab](https://github.com/ethereum/evmlab) - Utilities for interacting with the Ethereum virtual machine
* [IDA-EVM](https://github.com/trailofbits/ida-evm) - IDA plugin to view EVM instructions
* [Panoramix](http://eveem.org/about)
* [pyevmasm](https://github.com/trailofbits/pyevmasm) - EVM assembler and disassembler with a CLI and a Python API
* [Rattle](https://github.com/trailofbits/rattle) - EVM binary static analysis framework. Produces SSA representations of EVM code.

### Fuzzers

Fuzzing is the technique of providing random inputs to a program and check for unwanted behavior, like unexpected reverts. It's useful in order to find unhandled edge cases.

The most well know smart contract fuzzers are **Echindia** and **Harvey**.

* [Echidna](https://github.com/trailofbits/echidna) - Fuzzer for Ethereum smart contracts. Uses property testing to generate malicious inputs that break smart contracts.

### Vizualizers <a href="#vizualizers" id="vizualizers"></a>

These are the tools that help to comprehend a program by displaying its visual information, like a call graph that shows each of functions executed during a specific call.

A well-known tool for generating good visual representations is Surya.

### Symbolic Execution (a.k.a. Dynamic Analysis)

These tools will run the code (or application, they can be used in "black box" fashion), with predefined inputs and check results. **Manticore** and **Mythril** are the most prominent ones.

A more complete list of tools can be found at [https://consensys.github.io/smart-contract-best-practices/security\_tools/.](https://consensys.github.io/smart-contract-best-practices/security\_tools/)

With that many flavors to choose from, which ones should you pick first? There are no rules about this, you should use the tools that you are comfortable handling, that will help you understand or test the code you have at hand faster. In an audit the most important constraint is time, so you don't want to spend your time only running tools, over time you will develop a sense of how to use the tools in order to get faster to the point where you are comfortable with the code, while leaving time to analyse their reports, and also manually inspect the code.

If you are just starting this journey, there are some things to keep in mind.

> Look for the most used tools, as they are usually well maintained, and you'll find better documentation and more places to get help if needed. Generally, visualizers and static analysis tools will require very little (or no) setup, fuzzers will require a little more, and symbolic execution can require extensive setup. Some tools will also use more than one method, saving you time and improving results. Keep this in mind, as tools that take long to setup will eat up your time during an audit, but can give you insights others can't.

Another important step will be to analyze the reports of the tools.

> Keep in mind that generally security tools are setup in very strict ways, and will default to alerting you at the most remote possibility of there being a problem, which leads to lots of false positives. Usually a report from any of the tools above, except maybe for a well-set-up dynamic analysis tool, will bring you more false positives than bugs. This is also caused by the fact that some of the tools will break the code down into sections and will disregard the rest of it. This can cause an alert for possible overflow in an arithmetic operation, while disregarding a require() just above that would prevent the overflow.

Some examples you've probably seen already are the uint overflow and "gas requirement high" in the static analyzer present in Remix.

If you want to get deeper into tools, [read this paper](https://publik.tuwien.ac.at/files/publik\_278277.pdf). It includes a nice list comparing methods used, and vulnerabilities detected.

* [Manticore dynamic analyzer](https://github.com/trailofbits/manticore)
* [Scribble runtime verification](https://github.com/ConsenSys/scribble)
* [Symbolic Execution with ds-test](https://fv.ethereum.org/2020/12/11/symbolic-execution-with-ds-test/)
