# Access Arrays Using Enumeration and Pagination

Blockchain state queries in the form of calls to `view` functions are bounded in complexity, and gas-intensive operations such as returning large arrays can make them unusable. Use enumeration and pagination techniques to prevent this.

## Description

A block's gas limit places an upper bound on how much gas transactions can use, limiting their complexity. Calls to `view` functions do not create transactions and rely instead on the [JSON-RPC `eth_call`](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_call). However, they are also constrained by gas.

Gas on an `eth_call` is not related to block size or fees paid by transaction senders, but instead to node performance. By preserving gas deduction semantics, nodes can easily implement Denial of Service protections and limit resources spent processing each query. This is specially relevant when they provide an API to third parties. As an example, Infura notes on its documentation [the existence of a cap on the `gas` value](https://infura.io/docs/ethereum/json-rpc/eth-call) for `eth_call`.

This makes queries \(`view` functions\) that involve unbounded iteration \(such as accessing an arbitrarily large array\) dangerous: if these grow large enough interacting with the contract will become impossible since execution will not have enough gas.

Enumeration avoids this issue by breaking up an arbitrarily large operation into a number of smaller steps, each executable in a single call. A good example of this is the [`ERC721Enumerable` contract](https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#IERC721Enumerable), an extension to the ERC721 standard that allows accessing its internal arrays in a scalable manner. ERC721's first drafts did include a `tokensOfOwner` function that returned the entire array, but it was [removed](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1064#issuecomment-462837092) after it [proved to be flawed](https://github.com/ethereum/eips/issues/721#issuecomment-351871604) due to unbounded gas consumption.

Unfortunately, enumeration also comes with drawbacks: by performing a new JSON-RPC call for each element in a collection, iterating over a large array becomes very inefficient in terms of number of requests and network usage. This is easily overcome by means of pagination. Instead of reading one entry at a time, with pagination each query returns multiple elements defined by a start index and page size, thus allowing for fine-grained control over the number and size of requests to be made.

Pagination can also be built on top of existing enumerable contracts _without modifying them_, by deploying a separate contract capable of performing paginated queries.

## Example

In the following example, the contract is dangerously returning an unbounded array, which will error out once the array becomes large enough due to insufficient gas.

```text
// Inadequate code - do not use

pragma solidity ^0.6.0;

contract Store {
    uint256[] private _entries;

    function entries() public view returns (uint256[] memory) {
        return _entries;
    }

    ...
}
```

This can be improved by using enumeration, returning entries queried by index. The new `entryAt` function has a constant gas cost, independent of the length of the `_entries` array.

```text
pragma solidity ^0.6.0;

contract EnumerableStore {
    uint256[] private _entries;

    function entryAt(uint256 index) public view returns (uint256) {
        return _entries[index];
    }

    function totalEntries() public view returns (uint256) {
        return _entries.length;
    }

    ...
}
```

For increased network efficiency, multiple calls to `entryAt` can be performed on the same `eth_call` by using pagination. This can be built either directly on the same contract, or on a separate contract that queries `EnumerableStore`, as shown below.

```text
pragma solidity ^0.6.0;

contract PaginatedStoreQuerier {
    function entryAtPaginated(EnumerableStore store, uint256 startIndex, uint256 pageSize) public view returns(uint256[] memory) {
        uint256[] memory result;
        require(startIndex + pageSize <= store.totalEntries(), "Read index out of bounds");

        for (uint256 i = 0; i < pageSize; ++i) {
            result[i] = store.entryAt(i + startIndex);
        }

        return result;
    }
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/access-arrays-using-enumeration-and-pagination?](https://defender.openzeppelin.com/#/advisor/docs/access-arrays-using-enumeration-and-pagination?)

