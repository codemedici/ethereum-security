# Emit Events on All State Changes

{% hint style="success" %}
Actions that trigger state-changes in a smart contract should be accompanied by a [Solidity event](https://solidity.readthedocs.io/en/latest/contracts.html#events) that includes all information required to understand the effect they had on the system.
{% endhint %}

## Description

Calls to functions not marked as [`view`](https://solidity.readthedocs.io/en/latest/contracts.html#view-functions) or [`pure`](https://solidity.readthedocs.io/en/latest/contracts.html#pure-functions) can potentially modify blockchain state, by either writing to the contract's storage, transferring Ether, or calling into other contracts. Detecting that these actions happened is critical in order to monitor and analyze on-chain activity.

However, it is not possible to identify these function calls by inspecting transaction receipts only. By themselves they don't provide enough information to observe calls to [`internal` or `private`](https://solidity.readthedocs.io/en/latest/contracts.html#visibility-and-getters) functions, nor contracts calling one another \(known as 'internal transactions'\). The EVM provides a logging mechanism to workaround this issue, which can be used in Solidity in the form of [events](https://solidity.readthedocs.io/en/latest/contracts.html#events).

**All state-changing actions should emit events that describe the outcome of their execution**. A single event can comprise multiple storage writes and external calls, but it should include information about all of them. Additionally, any parameters used to filter events should be `indexed` \(for more see the best practice on **Indexed Event Parameters**\).

Events can be used to monitor new activity and to inspect executed transactions, but they **should not be relied on to determine the system's current state**. It is likely that some form of [event pruning will happen in the near future](https://gist.github.com/karalabe/60be7bef184c8ec286fc7ee2b35b0b5b#broken-invariants). Therefore, avoid using events as long-term storage, and see the best practice to **Make State Observable** instead.

## Example

The following contract is a simple token that hinders monitoring by not emitting any events on balance changes:

```text
// Inadequate code - do not use

pragma solidity ^0.6.0;

import "@openzeppelin/contracts/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    mapping (address => uint256) private _balances;

    function transfer(address to, uint256 amount) public {
        _balances[msg.sender] = _balances[msg.sender].sub(amount);
        _balances[to] = _balances[to].add(amount);
    }

    function transferBatch(address[] recipients, uint256[] amounts) public {
        require(recipients.length == amounts.length);

        for (uint256 i = 0; i < recipients.length; ++i) {
            transfer(recipients[i], amounts[i]);
        }
    }
}
```

Calls from externally-owned accounts to the `transfer` and `transferBatch` functions can be detected by inspecting a transaction's `data` field, but it is impossible to learn about other contracts calling them without performing runtime analysis on a synced node. This is specially problematic in the case of `transferBatch`, which allows performing multiple transfers between the same accounts in a single call.

To fix this, we define and emit a `Transfer` event.

```text
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    mapping (address => uint256) private _balances;

    event Transfer(address indexed from, address indexed to, uint256 amount);

    function transfer(address to, uint256 amount) public {
        _balances[msg.sender] = _balances[msg.sender].sub(amount);
        _balances[to] = _balances[to].add(amount);

        // All token transfers are accompanied by an event
        emit Transfer(msg.sender, to, amount);
    }

    function transferBatch(address[] recipients, uint256[] amounts) public {
        require(recipients.length == amounts.length);

        for (uint256 i = 0; i < recipients.length; ++i) {
            transfer(recipients[i], amounts[i]);
        }
    }
}
```

Note that in our example, calls `transferBatch` are indistinguishable from multiple calls to `transfer`. If needed, they can be differentiated by adding a `BatchTransfer` event.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/emit-events-on-all-state-changes?](https://defender.openzeppelin.com/#/advisor/docs/emit-events-on-all-state-changes?)

