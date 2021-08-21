# Do Not Track Time With Block Numbers

{% hint style="success" %}
Use of block number to track time is discouraged, because it implicitly and incorrectly assumes a constant block period. It is safer to track the passing of time using block timestamps, with the caveat that timestamps can be manipulated to some extent by miners.
{% endhint %}

## Description

Tracking the passing of time is often fundamental for the business logic of smart contract systems. Common examples include the use of timelocks to allow / disallow execution of actions, tracking voting periods of a specific length, or token offerings based on vesting mechanisms.

In all cases, usage of the block number to track the passing of time is highly discouraged. Systems that rely on block numbers incorrectly assume that the block period is constant. Yet there might be minor delays in the time between blocks that, accummulated over time, can significantly affect behavior. Moreover, Ethereum’s “difficulty bomb” may make mining more difficult, thus increasing the average block time \(see [Etherscan’s average block time](https://etherscan.io/chart/blocktime) for reference\). As a consequence, time between blocks could eventually be much larger than expected, affecting the system in unexpected and undesired ways.

A more reliable way of tracking time is using the timestamp of blocks. In Solidity, this can be achieved by reading the `block.timestamp` global variable. Yet developers must be aware that timestamps can be manipulated in the order of seconds by miners. Therefore, as an abundance of caution, it is advised to **only rely on `block.timestamp` as long as time variations of five minutes or less do not significantly affect behavior nor open attack vectors** in the system.

Currently, the [OpenEthereum \(Parity\)](https://github.com/openethereum/openethereum/blob/v3.0.0/ethcore/verification/src/verification.rs#L345-L347), [Geth](https://github.com/ethereum/go-ethereum/blob/release/1.8/consensus/ethash/consensus.go#L45) and [Hyperledger Besu](https://github.com/hyperledger/besu/blob/1.4.5/ethereum/core/src/main/java/org/hyperledger/besu/ethereum/mainnet/MainnetBlockHeaderValidator.java#L39) nodes allow timestamps up to 15 seconds in the future. But this is a simple convention not strictly enforced by standards, and could therefore change in the future. In earlier versions, nodes had a tolerance of up to 900 seconds \(15 minutes\).

Having considered this, if developers are still compelled to use the number of blocks to account for the passing of time, mechanisms to update the time between blocks must be put in place for a more robust implementation. Moreover, any assumptions made on the time between blocks must be made explicit with clear documentation, highlighting how block numbers relate to timestamps.

## Example

The following contract implements a function that should allow users to withdraw funds from the contract on a weekly basis. It implicitly and incorrectly relies on a block period of 15 seconds. Therefore, it may lock users' funds for an unexpected amount of time if the block period changes. Note also that the number of blocks to be waited \(40320\) is not self-explanatory, as it cannot be easily related to a 7-days delay.

```text
// Inadequate code - do not use

contract SomeContract {

    uint256 private constant WITHDRAWAL_DELAY_BLOCKS = 40320;

    mapping(address => uint256) private lastWithdrawalBlocks;

    modifier onlyIfWithdrawalAllowed() {
        require(block.number > lastWithdrawalBlocks[msg.sender] + WITHDRAWAL_DELAY_BLOCKS);
        _;
    }

    // A user can execute withdrawals on a weekly basis
    function withdrawFunds() external onlyIfWithdrawalAllowed { ... }
}
```

To fix this, the contract should instead rely on block timestamps:

```text
contract SomeContract {

    uint256 private constant WITHDRAWAL_DELAY_SECS = 7 days;

    mapping(address => uint256) private lastWithdrawalTimestamps;

    modifier onlyIfWithdrawalAllowed() {
        require(block.timestamp > lastWithdrawalTimestamps[msg.sender] + WITHDRAWAL_DELAY_SECS);
        _;
    }

    // A user can execute withdrawals on a weekly basis
    function withdrawFunds() external onlyIfWithdrawalAllowed { ... }
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/do-not-track-time-with-block-numbers?](https://defender.openzeppelin.com/#/advisor/docs/do-not-track-time-with-block-numbers?)

