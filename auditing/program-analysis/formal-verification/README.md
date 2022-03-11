# Formal Verification

## Introduction

[Formal Verification](../../auditing-process.md#formal-verification) is the process by which one proves properties of a system mathematically. In order to do that one writes a formal specification of the application behavior. The formal specification is analogous to our Statement of Intended Behavior, but it is written in a machine-readable language. The formal specification is later proved (or not) using one of the available tools.

The process is widely used by the hardware industry, and also by industries that produce systems in fields such as flight controls, space flight and life-support where they routinely deal with high risk (non-trivial Likelihood of a defect in a complex system \* catastrophic Impact). Think about the operating system for a nuclear power station.

Formal verification is out of the scope of this course (there are large books on the subject), but we should understand what it is, how it works, and its limitations, at a high level.

Formal verification is a trendy topic in the blockchain world. It is natural that the community would relate smart contracts to the aforementioned industries. Formal verification is not, however, a silver bullet, or a substitute for a good audit. Also in the aforementioned industries, an audit is frequently conducted, including not only on the code base, but also on the formal verification specification itself. Flaws in this specification will mean that some of the properties of the application are not actually proven in the end, with bugs possibly going unnoticed.

Formal verification requires segregation of duties, in order to be effective. If the writer of the specifications is the coder of the application, the effectiveness of formal verification is greatly reduced.

Lastly, in our industry, due to the rough state of the tooling, formal verification processes are costly and lengthy. That is likely to change as more people include it in their development processes.

## Code Audits

While no smart contract can be guaranteed as safe and free of bugs, a thorough code audit and formal verification process from a reputable security firm helps uncover critical, high severity bugs that otherwise could result in financial harm to users.

A recent example is MakerDAO's Multi Collateral DAI set of smart contracts. Most of the smart contracts were formally verified and an audit was conducted. This was the start of an excellent process. Even so, a USD $50,000 critical bug was awarded by their Bug Bounty program, demonstrating the value of a Bug Bounty even after audits and formal verification.

## Tools

### Manticore

****[**Manticore**](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/manticore) performs the "heaviest weight" analysis. Like Echidna, Manticore verifies user-provided properties. It will need more time to run, but it can prove the validity of a property and will not report false alarms.

### solc-verify

Solc-verify is a source-level formal verification tool for Soldity smart contracts, developed in collaboration with SRI International. Solc-verify takes smart contracts written in Solidity and discharges verification conditions using modular program analysis and SMT solvers. Built on top of the Solidity compiler, solc-verify reasons at the level of the contract source code. This enables solc-verify to effectively reason about high-level functional properties while modeling low-level language semantics (e.g., the memory model) precisely. The contract properties, such as contract invariants, loop invariants, function pre- and post-conditions and fine grained access control can be provided as in-code annotations by the developer. This enables automated, yet user-friendly formal verification for smart contracts.
