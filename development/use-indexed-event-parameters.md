# Use Indexed Event Parameters

{% hint style="success" %}
Event parameters that will be used as a filter when searching through past events should have the `indexed` attribute.
{% endhint %}

## Description

Events are useful to monitor contract activity and look at past transactions. Often, a search will be performed for a subset of all events of a given type, such as token `Transfer` events with a specific recipient. For this operation to be efficient, the event parameters used for filtering must have the `indexed` attribute.

Solidity events are included in the transaction logs, which means that a naive event search requires going through all transaction receipts. This can be improved by marking some parameters as `indexed`: these values will be added to the block's [Bloom filter](https://en.wikipedia.org/wiki/Bloom_filter), which makes searching for them very efficient. Because indexed parameters are sligthly more expensive, and Solidity events are limited to three `indexed` parameters each, it is important to only apply this attribute to values that will be used when narrowing down searches.

While the `indexed` attribute is most commonly applied to integer and address types, dynamic types such as strings or arrays can also be indexed, albeit with a big limitation. The actual value will not be stored in the logs, but rather it's _hash_. This means that `indexed` dynamic types are only suitable for instances where the underlying values \(the preimages of the hashes\) are known.

## Example

The following contract is a simple token where the `Transfer` event has the `indexed` attribute for both sender and recipient. This makes it possible to efficiently search for token transfers from or to specific addresses.

Note that the transferred amount is not indexed: searches for token transfers of specific amounts are very rare, so making them inefficient is a non-issue.

```text
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    mapping (address => uint256) _balances;

    event Transfer(address indexed from, address indexed to, uint256 amount);

    function transfer(address to, uint256 amount) public {
        _balances[msg.sender] = _balances[msg.sender].sub(amount);
        _balances[to] = _balances[to].add(amount);

        emit Transfer(msg.sender, to, amount);
    }
}
```

The next token contract adds an indexed `data` field to the event, with arbitrary data passed by the caller. This can be used to tag token transfers with the same data so that they their corresponding events can be efficiently retrieved. Because `bytes` is a dynamic type, its actual value will not be included in the event, only its hash.

```text
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    mapping (address => uint256) _balances;

    event Transfer(address indexed from, address indexed to, uint256 amount, bytes indexed data);

    // Token transfer with arbitrary associated data
    function transfer(address to, uint256 amount, bytes memory data) public {
        _balances[msg.sender] = _balances[msg.sender].sub(amount);
        _balances[to] = _balances[to].add(amount);

        emit Transfer(msg.sender, to, amount, data);
    }
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/use-indexed-event-parameters?](https://defender.openzeppelin.com/#/advisor/docs/use-indexed-event-parameters?)

