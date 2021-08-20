# Test Contract Upgrades

Before upgrading a live mainnet smart contract to a new implementation, it's critical to test the upgrade process itself, even if both the old and new implementations are correct, since the upgrade itself can potentially introduce issues.

## Description

Smart contract upgrades allow changing the code executed in a contract, while preserving the existing contract state, balance, and address. Regardless of the upgrade pattern being used, it's important to test the upgrade before actually executing it on mainnet.

Tests should verify not only the behavior of the ugpraded implementation, but also that state was correctly preserved during the upgrade, and that it's possible to rollback if needed. Tests should be run on local development nodes, testnets, and ideally on mainnet forks as well. It's also a good idea to include static analyzers if available for the upgrade pattern being used.

## What to test

A good setup for testing starts by deploying and seeding the original version of the contract, and then executes the upgrade using the same pattern as the live contract uses. Tests should assert that the new implementation behaves correctly, and that all state and balance has been properly preserved. Note that testing the new implementation in isolation is not enough, and any test suite developed for it should be re-run on an upgraded instance as well. For example, in a delegate-call proxy upgrade pattern, a new implementation that [redefines the order of storge variables](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts) may corrupt the contract state when upgrading; this cannot be detected by testing the old or new implementations alone, and can only be found by performing an upgrade.

Additionally, if there are any migration methods to be executed during the upgrade, such as calculating the value for a new field, these should be tested as well. Furthermore, it's important to test that these methods, after being called for the migration, cannot be called again.

Another good practice is to test that the new implementation can be further upgraded, or even rolled back to the previous implementation if an error is detected. Some upgrade patterns, such as [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822#proxiable-contract), depend on the implementation defining the rules for executing an upgrade, so if a contract is upgraded to a faulty implementation, there is a risk to be locked in that version without having the possibility to roll back or switch to a different one.

## Tooling

When using delegate-call proxy upgrades such as [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967), it's highly recommended to use static analysis tools to verify that the new implementation is [upgrade-safe](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable) and [compatible with the previous one](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts).

We recommend using the [OpenZeppelin upgrades plugins](https://docs.openzeppelin.com/upgrades-plugins/1.x/) for Truffle or Hardhat, that will automatically verify that your contracts are upgrade-safe, and will check that the storage layout between both implementations is compatible. An alternative is [Slither](https://github.com/crytic/slither)'s [upgradeability checks plugin](https://github.com/crytic/slither/wiki/Upgradeability-Checks).

## How to test

The most immediate way to test an upgrade is to set up a suite of unit tests on a local development node, using the testing framework of your choice \(see below for an example using [Hardhat](https://hardhat.org/)\). Note that this requires keeping both the old and new implementations of your contract in your repository. Tests should deploy the original contract instance, set its state, run the upgrade, and assert that all state was correctly preserved and that the new implementation behaves as expected.

However, since live mainnet state is usually more complex than what can be set up in a unit test, it's recommended to test the upgrade by _forking_ mainnet. If using [ganache](https://github.com/trufflesuite/ganache-cli), you can spin up a new instance out of a mainnet block by starting it with the [`--fork` option](https://github.com/trufflesuite/ganache-cli#options). You can also _unlock_ the account with admin rights to execute the upgrade via another `--unlock ADDRESS` option, so you don't include the keys for upgrading the live contract in your tests. When testing the upgrade this way, your tests should upgrade the mainnet instance in the ganache fork, and run assertions to verify the state and behavior of the upgraded instance.

Another good practice is to test the upgrade in a testnet. This requires reproducing your mainnet setup in a testnet, and following the same steps you would take in mainnet but in your testnet setup. This helps you test not only the code upgrade, but also the process you have designed for it. For instance, if you are using a Gnosis Safe multisig for controlling the upgrade and executing it via Defender Admin, you should replicate the same setup in a testnet, and run it using exactly the same tools.

## Example: unit tests on Hardhat

As an example, we will test the upgrade for a simple `Box` contract using [Hardhat](https://hardhat.org/) together with the [OpenZeppelin upgrades plugin](https://docs.openzeppelin.com/upgrades-plugins/1.x/). The contract holds a numerical value, which we will test that is properly persisted inbetween upgrades.

```javascript
pragma solidity ^0.6.8;

contract BoxV1 {
  uint256 public value;

  function setValue(uint256 _value) public {
    value = _value;
  }
}

contract BoxV2 {
  uint256 public value;

  function setValue(uint256 _value) public {
    value = _value;
  }

  function increase() public {
    value = value + 1;
  }
}
```

Note that we are keeping both implementations in our code, so we can test the upgrade from one to the other. Old implementations can optionally be kept in a separate folder to avoid polluting the main contracts folder in the project.

The test will deploy a new `BoxV1` instance, set an initial value, upgrade it to `BoxV2`, and assert that the value was migrated successfully from one version to the next. We also check that the new functionality added by V2 works as expected.

```javascript
const { expect } = require("chai");

describe("Upgrade test", function() {
  it("upgrades successfully", async function() {
    // Deploy V1 and set initial value
    const BoxV1 = await ethers.getContractFactory("BoxV1");
    const instance = await upgrades.deployProxy(BoxV1);
    await instance.setValue('42');
    expect((await instance.value()).toString()).to.equal('42');

    // Upgrade to V2 and check value
    const BoxV2 = await ethers.getContractFactory("BoxV2");
    const upgraded = await upgrades.upgradeProxy(instance.address, BoxV2);
    expect((await upgraded.value()).toString()).to.equal('42');

    // Verify that new implementation works as expected
    await upgraded.increase();
    expect((await upgraded.value()).toString()).to.equal('43');
  });
});
```



## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/test-contract-upgrades](https://defender.openzeppelin.com/#/advisor/docs/test-contract-upgrades)
* [https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/)
* [Proxy Delegate](https://github.com/fravoll/solidity-patterns/blob/master/docs/proxy_delegate.md)\*\*\*\*
* [Eternal Storage](https://github.com/fravoll/solidity-patterns/blob/master/docs/eternal_storage.md) - separate storage and logic between two contracts.

