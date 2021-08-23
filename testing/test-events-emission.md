---
description: >-
  Write tests to ensure that events are emitted as expected from smart contract
  functions.
---

# Test Events Emission

## Description

As recommended in development best practices, it is useful to emit events for all contract state changes. Event emission should also be covered by tests, both to ensure contract functionality and to make sure that events are working as expected which can be critical for protocol monitoring.

Multiple test helper libraries exist that make it fairly easy to test event emission. Incorporate the use of such a library and make sure that tests cover all events for both positive and negative cases.

## Example with OpenZeppelin Test Helpers

Use [OpenZeppelin Test Helpers](https://www.npmjs.com/package/@openzeppelin/test-helpers) with Truffle or Hardhat or other environments to test event emission.

```text
const { accounts, contract } = require('@openzeppelin/test-environment');

const {
  BN,           // Big Number support 
  expectEvent,  // Assertions for emitted events
} = require('@openzeppelin/test-helpers');

const MyContract = contract.fromArtifacts('MyContract');

describe('Test events on MyContract', function () {
  let myContract;
  const [sender, receiver] = accounts;

  beforeEach(async function () {
    myContract = await MyContract.new();
  });

  it('emits Transfer event on a good transfer', async function () {
    const goodValue = new BN(1);
    const receipt = await myContract.transferRequiresValue(
      receiver, goodValue, { from: sender }
    );

    // event assertion and verify arguments
    expectEvent(receipt, 'Transfer', {
      from: sender,
      to: receiver,
      value: goodValue,
    });
  });

  it('does not emit Transfer event on a failed transfer', async function () {
    const failValue = new BN(0);
    const receipt = await myContract.transferRequiresValue(
      receiver, failValue, { from: sender }
    );

    // make sure event not emitted
    expectEvent.notEmitted(receipt, 'Transfer');
  });
});
```

## Example with Truffle-Assertions

Use [truffle-assertions](https://www.npmjs.com/package/truffle-assertions) with Truffle to test event emission.

```text
contract('MyContract', function (accounts) {
    let myContract;
    const sender = accounts[0];
    const receiver = accounts[1];

    beforeEach(async () => {
        myContract = await MyContract.new();
    });
});

it("emits Transfer event on a good transfer", async function () {
    const goodValue = 1;

    let tx = await myContract.transferRequiresValue(receiver, goodValue, { from: sender });

    // event assertion and verify arguments
    truffleAssert.eventEmitted(tx, 'Transfer', (ev) => {
        return ev.from === sender && ev.to === receiver && ev.value === goodValue;
    });
});

it("does not emit Transfer event on a failed transfer", async function () {
    const failValue = 0;

    let tx = await myContract.transferRequiresValue(receiver, failValue, { from: sender });

    // makes sure event not emitted
    truffleAssert.eventNotEmitted(tx, 'Transfer');
});
```

## Example with Waffle

Use [Waffle](https://getwaffle.io/) with [Ethers](https://https//github.com/ethers-io/ethers.js/) to test event emission.

```text
import {expect, use} from 'chai';
import {Contract} from 'ethers';
import {deployContract, MockProvider, solidity} from 'ethereum-waffle';

use(solidity);

describe('MyContract', () => {
  const [sender, receiver] = new MockProvider().getWallets();
  let myContract: Contract;

  beforeEach(async () => {
    myContract = await deployContract();
  });

  it('emits Transfer event on a good transfer', async function () {
    const goodValue = 1;
    await expect(myContract.transferRequiresValue({
       from: sender.address,
       to: receiver.address,
       value: goodValue}))
      .to.emit(myContract, 'Transfer')
      .withArgs(sender.address, receiver.address, goodValue);
  });

  it('does not emit Transfer event on a failed transfer', async function () {
    const failValue = 0;
    await expect(myContract.transferRequiresValue({
       from: sender.address,
       to: receiver.address,
       value: failValue}))
      .to.not.emit(myContract, 'Transfer');
  });
});
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/test-emission-of-events?](https://defender.openzeppelin.com/#/advisor/docs/test-emission-of-events?)

