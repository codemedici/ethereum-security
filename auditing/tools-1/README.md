# Program Analysis

We are going use three distinctive testing and program analysis techniques:

* **Static analysis with** [**Slither**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither)**.** All the paths of the program are approximated and analyzed at the same time, through different program presentations \(e.g. control-flow-graph\)
* **Fuzzing with** [**Echidna**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna)**.** The code is executed with a pseudo-random generation of transactions. The fuzzer will try to find a sequence of transactions to violate a given property.
* **Symbolic execution with** [**Manticore**](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore)**.** A formal verification technique, which translates each execution path to a mathematical formula, on which on top constraints can be checked.

Each technique has advantages and pitfalls, and will be useful in [specific cases](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis#determining-security-properties):

| Technique | Tool | Usage | Speed | Bugs missed | False Alarms |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Static Analysis | Slither | CLI & scripts | seconds | moderate | low |
| Fuzzing | Echidna | Solidity properties | minutes | low | none |
| Symbolic Execution | Manticore | Solidity properties & scripts | hours | none\* | none |

\* if all the paths are explored without timeout

**Slither** analyzes contracts within seconds, however, static analysis might lead to false alarms and will be less suitable for complex checks \(e.g. arithmetic checks\). Run Slither via the API for push-button access to built-in detectors or via the API for user-defined checks.

**Echidna** needs to run for several minutes and will only produce true positives. Echidna checks user-provided security properties, written in Solidity. It might miss bugs since it is based on random exploration.

**Manticore** performs the "heaviest weight" analysis. Like Echidna, Manticore verifies user-provided properties. It will need more time to run, but it can prove the validity of a property and will not report false alarms.

## Suggested workflow

Start with Slither's built-in detectors to ensure that no simple bugs are present now or will be introduced later. Use Slither to check properties related to inheritance, variable dependencies, and structural issues. As the codebase grows, use Echidna to test more complex properties of the state machine. Revisit Slither to develop custom checks for protections unavailable from Solidity, like protecting against a function being overridden. Finally, use Manticore to perform targeted verification of critical security properties, e.g., arithmetic operations.

* Use Slither's CLI to catch common issues
* Use Echidna to test high-level security properties of your contract
* Use Slither to write custom static checks
* Use Manticore once you want in-depth assurance of critical security properties

**A note on unit tests**. Unit tests are necessary to build high-quality software. However, these techniques are not the best suited to find security flaws. They are typically used to test positive behaviors of code \(i.e. the code works as expected in the normal context\), while security flaws tend to reside in edge cases that the developers did not consider. In our study of dozens of smart contract security reviews, [unit test coverage had no effect on the number or severity of security flaws](https://blog.trailofbits.com/2019/08/08/246-findings-from-our-smart-contract-audits-an-executive-summary/) we found in our client's code.

## Determining Security Properties

To effectively test and verify your code, you must identify the areas that need attention. As your resources spent on security are limited, scoping the weak or high-value parts of your codebase is important to optimize your effort. Threat modeling can help. Consider reviewing:

* [Rapid Risk Assessments](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html) \(our preferred approach when time is short\)
* [Guide to Data-Centric System Threat Modeling](https://csrc.nist.gov/publications/detail/sp/800-154/draft) \(aka NIST 800-154\)
* [Shostack thread modeling](https://www.amazon.com/Threat-Modeling-Designing-Adam-Shostack/dp/1118809998)
* [STRIDE](https://en.wikipedia.org/wiki/STRIDE_%28security%29) / [DREAD](https://en.wikipedia.org/wiki/DREAD_%28risk_assessment_model%29)
* [PASTA](https://en.wikipedia.org/wiki/Threat_model#P.A.S.T.A.)
* [Use of Assertions](https://blog.regehr.org/archives/1091)

### Components

Knowing what you want to check will also help you to select the right tool.

The broad areas that are frequently relevant for smart contracts include:

* **State machine.** Most contracts can be represented as a state machine. Consider checking that \(1\) No invalid state can be reached, \(2\) if a state is valid that it can be reached, and \(3\) no state traps the contract.
  * Echidna and Manticore are the tools to favor to test state-machine specifications.
* **Access controls.** If you system has privileged users \(e.g. an owner, controllers, ...\) you must ensure that \(1\) each user can only perform the authorized actions and \(2\) no user can block actions from a more priviledged user.
  * Slither, Echidna and Manticore can check for correct access controls. For example, Slither can check that only whitelisted functions lack the onlyOwner modifier. Echidna and Manticore are useful for more complex access control, such as a permission given only if the contract reaches a given state.
* **Arithmetic operations.** Checking the soundness of the arithmetic operations is critical. Using `SafeMath` everywhere is a good step to prevent overflow/underflow, however, you must still consider other arithmetic flaws, including rounding issues and flaws that trap the contract.
  * Manticore is the best choice here. Echidna can be used if the arithmetic is out-of-scope of the SMT solver.
* **Inheritance correctness.** Solidity contracts rely heavily on multiple inheritance. Mistakes such as a shadowing function missing a `super` call and misinterpreted c3 linearization order can easily be introduced.
  * Slither is the tool to ensure detection of these issues.
* **External interactions.** Contracts interact with each other, and some external contracts should not be trusted. For example, if your contract relies on external oracles, will it remain secure if half the available oracles are compromised?
  * Manticore and Echidna are the best choice for testing external interactions with your contracts. Manticore has an built-in mechanism to stub external contracts.
* **Standard conformance.** Ethereum standards \(e.g. ERC20\) have a history of flaws in their design. Be aware of the limitations of the standard you are building on.
  * Slither, Echidna, and Manticore will help you to detect deviations from a given standard.

### Tool selection cheatsheet

| Component | Tools | Examples |
| :--- | :--- | :--- |
| State machine | Echidna, Manticore |  |
| Access control | Slither, Echidna, Manticore | [Slither exercise 2](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither/exercise2.md), [Echidna exercise 2](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-2.md) |
| Arithmetic operations | Manticore, Echidna | [Echidna exercise 1](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-1.md), [Manticore exercises 1 - 3](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/manticore/exercises) |
| Inheritance correctness | Slither | [Slither exercise 1](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/slither/exercise1.md) |
| External interactions | Manticore, Echidna |  |
| Standard conformance | Slither, Echidna, Manticore | [`slither-erc`](https://github.com/crytic/slither/wiki/ERC-Conformance) |

Other areas will need to be checked depending on your goals, but these coarse-grained areas of focus are a good start for any smart contract system.

Our public audits contain examples of verified or tested properties. Consider reading the `Automated Testing and Verification` sections of the following reports to review real-world security properties:

* [0x](https://github.com/trailofbits/publications/blob/master/reviews/0x-protocol.pdf)
* [Balancer](https://github.com/trailofbits/publications/blob/master/reviews/BalancerCore.pdf)

## Tools List

These are the tools that help to comprehend a program by displaying its visual information, like a call graph that shows each of functions executed during a specific call.

* [ethereum-graph-debugger](https://github.com/fergarrui/ethereum-graph-debugger) - A graphical EVM debugger. Displays the entire program control flow graph.
* [Slither](https://github.com/trailofbits/slither) - Slither can map method visibility and modifiers, state variables that are read and written, calls, and can print the inheritance graph of a smart contract
* [Solgraph](https://github.com/raineorshine/solgraph) - Generates DOT graphs with function control flow of a solidity contract
* [Surya](https://github.com/ConsenSys/surya) - Generates various visual outputs of function call graphs
* [sol-function-profiler](https://github.com/EricR/sol-function-profiler) - Solidity contract function profiler

### Linters

* [Remix](https://remix.ethereum.org/) - Browser-based Solidity IDE with linting features
* [SmarrtCheck](https://tool.smartdec.net/) - A linter for Solidity and Vyper that checks code for security issues and bad practices.
* [Solhint](https://github.com/protofire/solhint) - Linter for both security and style-guide validations. It strictly adheres to the [Solidity Style Guide](https://solidity.readthedocs.io/en/latest/style-guide.html).
* [Solium](https://github.com/duaraghav8/Solium) - Linter for both security and style-guide validations. Does not strictly adhere to the Solidity Style Guide.

### Bug finding tools

These tools will inspect the source code for known vulnerabilities, much like you do manually. Examples are **Securify** and **SmartCheck**.

* [Manticore](https://github.com/trailofbits/manticore) - Symbolic execution tool for Ethereum smart contracts that includes detectors for common security flaws
* [Mythril OSS](https://github.com/ConsenSys/mythril/) - Open-source security analysis tool for Ethereum smart contracts built around detector modules
* [Securify](https://github.com/eth-sri/securify) - Static analysis tool from ChainSecurity. [https://securify.chainsecurity.com](https://securify.chainsecurity.com) - works on bytecode level
* [Manticore](https://github.com/trailofbits/manticore) - Detects many common bug types, and can prove correctness properties with symbolic execution.
* [Mythril](https://github.com/ConsenSys/mythril) - Security analysis tool for smart contracts.
* [ethereum/sourcify](https://github.com/ethereum/sourcify) - Re-compiler that can be used to verify that bytecode corresponds to certain source code.
* [eth-sri/securify2](https://github.com/eth-sri/securify2) - Tool for analyzing smart contracts for vulnerabilities and insecure coding.
* [Slither](https://github.com/crytic/slither) - Static analyzer with support for many common bug types, including visualization tools for security-relevant information.
* [MythX](https://mythx.io/) - Detection for security vulnerabilities in Ethereum smart contracts throughout the development life cycle
* Oyente
* Maian
* Zeus
* MadMax

### Verification tools

* [KEVM](https://github.com/kframework/evm-semantics) - K Semantics of the Ethereum Virtual Machine \(EVM\)
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

### Symbolic Execution \(a.k.a. Dynamic Analysis\)

These tools will run the code \(or application, they can be used in "black box" fashion\), with predefined inputs and check results. **Manticore** and **Mythril** are the most prominent ones.

A more complete list of tools can be found at [https://consensys.github.io/smart-contract-best-practices/security\_tools/.](https://consensys.github.io/smart-contract-best-practices/security_tools/)

With that many flavors to choose from, which ones should you pick first? There are no rules about this, you should use the tools that you are comfortable handling, that will help you understand or test the code you have at hand faster. In an audit the most important constraint is time, so you don't want to spend your time only running tools, over time you will develop a sense of how to use the tools in order to get faster to the point where you are comfortable with the code, while leaving time to analyse their reports, and also manually inspect the code.

If you are just starting this journey, there are some things to keep in mind.

> Look for the most used tools, as they are usually well maintained, and you'll find better documentation and more places to get help if needed. Generally, visualizers and static analysis tools will require very little \(or no\) setup, fuzzers will require a little more, and symbolic execution can require extensive setup. Some tools will also use more than one method, saving you time and improving results. Keep this in mind, as tools that take long to setup will eat up your time during an audit, but can give you insights others can't.

Another important step will be to analyze the reports of the tools.

> Keep in mind that generally security tools are setup in very strict ways, and will default to alerting you at the most remote possibility of there being a problem, which leads to lots of false positives. Usually a report from any of the tools above, except maybe for a well-set-up dynamic analysis tool, will bring you more false positives than bugs. This is also caused by the fact that some of the tools will break the code down into sections and will disregard the rest of it. This can cause an alert for possible overflow in an arithmetic operation, while disregarding a require\(\) just above that would prevent the overflow.

Some examples you've probably seen already are the uint overflow and "gas requirement high" in the static analyzer present in Remix.

If you want to get deeper into tools, [read this paper](https://publik.tuwien.ac.at/files/publik_278277.pdf). It includes a nice list comparing methods used, and vulnerabilities detected.

* [Manticore dynamic analyzer](https://github.com/trailofbits/manticore)
* [Scribble runtime verification](https://github.com/ConsenSys/scribble)
* [Symbolic Execution with ds-test](https://fv.ethereum.org/2020/12/11/symbolic-execution-with-ds-test/)

