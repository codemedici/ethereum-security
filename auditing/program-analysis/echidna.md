# Echidna

## Echidna Tutorial

The aim of this tutorial is to show how to use Echidna to automatically test smart contracts.

The first part introduces how to write a property for Echidna. The second part is a set of exercises to solve.

**Table of contents:**

* Introduction
  * [Installation](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/echidna#installation)
  * [Introduction to fuzzing](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/fuzzing-introduction.md): Brief introduction to fuzzing
  * [How to test a property](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/how-to-test-a-property.md): How to test a property with Echidna
* Basic
  * [How to filter functions](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/filtering-functions.md): How to filters the functions to be fuzzed
  * [How to test assertions](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/assertion-checking.md): How to test assertions with Echidna
* Advanced
  * [How to collect a corpus](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/collecting-a-corpus.md): How to use Echidna to collect a corpus of transactions
  * [How to detect high gas consumption](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/finding-transactions-with-high-gas-consumption.md): How to find functions with high gas consumption.
  * [How to perform smart contract fuzzing at a large scale](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/smart-contract-fuzzing-at-scale.md): How to use Echidna to run long fuzzing campaign in complex smart contracts.
  * [How to test a library](https://blog.trailofbits.com/2020/08/17/using-echidna-to-test-a-smart-contract-library/): How Echidna was used to test the a library in Set Protocol \(blogpost\)
  * [How to test bytecode-only contracts](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/testing-bytecode.md): How to fuzz a contracts without bytecode, or to perform differential fuzzing between Solidity and Vyper
  * [How to seed Echidna with unit tests](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/end-to-end-testing.md): How to use existing unit tests to seed Echidna
* Exercises
  * [Exercise 1](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-1.md): Testing token's balance
  * [Exercise 2](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-2.md): Testing access control
  * [Exercise 3](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-3.md): Testing with custom initialization
  * [Exercise 4](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/Exercise-4.md): Testing with `assert`

Join the team on Slack at: [https://empireslacking.herokuapp.com/](https://empireslacking.herokuapp.com/) \#ethereum

### Installation

Echidna can be installed through docker or using the pre-compiled binary.

#### Echidna through docker

```text
docker pull trailofbits/eth-security-toolbox
docker run -it -v "$PWD":/home/training trailofbits/eth-security-toolbox
```

_The last command runs eth-security-toolbox in a docker that has access to your current directory. You can change the files from your host, and run the tools on the files from the docker_

Inside docker, run :

```text
solc-select 0.5.11
cd /home/training
```

#### Binary

[https://github.com/crytic/echidna/releases/tag/v1.6.0](https://github.com/crytic/echidna/releases/tag/v1.6.0)

solc 0.5.11 is recommended for the exercises.

