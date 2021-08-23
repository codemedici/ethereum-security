# Achieve High Smart Contract Test Coverage

Smart contract tests should be instrumented to measure their line coverage, and test cases added to reach a high coverage percentage.

## Description

Line test coverage is a widely used metric. It indicates which percentage of source code lines were 'tested' \(executed\) by a test suite run. Typically the coverage measurement is required alongside the tests themselves.

100% coverage means all lines were hit. This metric is a good goal to strive for. Important: it doesn't mean that the tests are perfect, or even that they are good. However, a coverage of less than 100% does mean that there are scenarios not being tested by the suite. In other words, the coverage analysis tells you what percentage of code you are **not** testing.

There are valid reasons why 100% coverage may not be achievable, including difficulties introduced by the instrumentation software, e.g., measuring gas spent would not be possible since instrumenting tests would increase gas usage. 100% coverage should not be seen as a goal, but rather use the coverage report as a tool to better understand which branches of code are not covered by tests, and guide further test development. Practice shows that most projects should be able to achieve over 90% testing coverage.

[solidity-coverage](https://github.com/sc-forks/solidity-coverage) is a de facto test coverage solution for smart contracts tests. Its integration into smart contracts tests is non trivial, that is why it's preferred to use it via tools like [Test Environment](https://docs.openzeppelin.com/test-environment/0.1/), [Truffle](https://www.trufflesuite.com/), and [Hardhat](https://hardhat.org/). Internally it works by [instrumenting](https://blog.colony.io/code-coverage-for-solidity-eecfa88668c2/) solidity code with special instructions which emit events.

Important considerations:

* [Tests that rely heavily on gas usage may fail](https://github.com/sc-forks/solidity-coverage/blob/master/docs/faq.md#notes-on-gas-distortion) and [should be skipped](https://github.com/sc-forks/solidity-coverage/blob/master/docs/advanced.md#skipping-tests).
* Coverage launches its own in-process ganache server. Meaning if your tests use Geth or Parity client, it is not possible to run solidity-coverage on them.
* Coverage runs tests a little more slowly. You may consider not including them in every CI run.
* Contracts are compiled without optimization.

## Example Solidity Coverage with Test Environment

### Add solidity-coverage package to your smart contracts project

```text
npm install --save-dev solidity-coverage
```

### Create `coverage.js` file

Here is a good example to start with:

```text
#!/usr/bin/env node
const { runCoverage } = require('@openzeppelin/test-environment');

async function main () {
  await runCoverage(
    ['mocks'], // an array of contracts to ignore, e.g., mock contracts
    'npm run compile', // npm command to compile smart contracts
    './node_modules/.bin/mocha --exit --timeout 10000 --recursive'.split(' '), // npm command to run tests
  );
}

main().catch(e => {
  console.error(e);
  process.exit(1);
});
```

The `coverage.js` script allows you to run smart contracts with any test runner, e.g., Jest, Mocha, and Ava.

### Add coverage command to package.json

```text
"coverage": "node ./coverage.js"
```

### Run Test Environment Coverage

```text
npm run coverage
```

You would get a similar result with: ![Solidity Coverate Test Environment report](https://github.com/OpenZeppelin/security-best-practices/blob/ef434b9aa28a46cddd21d64e58ade1a3bf22d136/images/test-env-solidity-coverage.png?raw=true)

Checkout [`coverage.js`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/scripts/coverage.js) used in the OpenZeppelin Contracts library.

## Example Solidity Coverage with Truffle

### Add solidity-coverage package to your Truffle project

```text
npm install --save-dev solidity-coverage
```

### Add solidity-coverage plugin to your plugins array in truffle-config.js

```text
module.exports = {
  ...
  plugins: ["solidity-coverage", ...]
}
```

### Run Truffle Coverage

```text
npx truffle run coverage
```

You would get a similar result: ![Solidity Coverate Truffle report](https://github.com/OpenZeppelin/security-best-practices/blob/ef434b9aa28a46cddd21d64e58ade1a3bf22d136/images/truffle-solidity-coverage.png?raw=true)

Checkout a full [MetaCoin Truffle Example](https://github.com/ylv-io/metacoin-solidity-coverage-example).

## Example Solidity Coverage with Hardhat

### Add solidity-coverage package to your Hardhat project

```text
npm install --save-dev solidity-coverage
```

### Add solidity-coverage plugin to your plugins array in hardhat.config.js

```text
usePlugin('solidity-coverage')

```

### Run Hardhat Coverage

```text
npx hardhat coverage
```

You would get a similar result with: ![Solidity Coverage Buidler report](https://github.com/OpenZeppelin/security-best-practices/blob/ef434b9aa28a46cddd21d64e58ade1a3bf22d136/images/buidler-solidity-coverage.png?raw=true)

Checkout a full [MetaCoin Buidler Example](https://github.com/sc-forks/buidler-e2e/tree/coverage).

### Coverage report visualization

Consider adding report visualization for your coverage reports. [codecov](https://codecov.io/) and [coveralls](https://coveralls.io/) are two suitable options. Setup depends on your CI and other tools used in the project, but should be a minor effort. Take a look at OpenZeppelin Contracts library test coverage [visualization](https://codecov.io/gh/OpenZeppelin/openzeppelin-contracts).

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/achieve-high-smart-contract-test-coverage?](https://defender.openzeppelin.com/#/advisor/docs/achieve-high-smart-contract-test-coverage?)

