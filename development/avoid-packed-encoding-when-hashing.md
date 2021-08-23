# Avoid Packed Encoding When Hashing

When hashing data of dynamic type, the use of `keccak256(abi.encodePacked(...))` can lead to hash collisions. Thus using `abi.encode` is usually preferred over `abi.encodePacked`. If compelled to use `abi.encodePacked` for operations related to signatures, authentication or data integrity, ensure that _at most_ one of the data types being encoded is dynamic.

### Description

Numerous systems use some form of hashed data for signatures, authentication and data integrity. In Solidity, generating the hash of a piece of data is as simple as using the `keccak256(bytes)` function, which computes the Keccak-256 hash of the bytes passed as input.

When structured data is to be hashed, `keccak256` is used together with `abi.encode` or `abi.encodePacked`. The latter is known as the [non-standard packed mode](https://solidity.readthedocs.io/en/latest/abi-spec.html#non-standard-packed-mode), which encodes dynamic types in-place and without their length. When two dynamically-sized elements, such as strings, are packed using `abi.encodePacked`, the encoding becomes ambiguous because of the missing length field. As an example, `abi.encodePacked("a","bc")` will return the same bytes as `abi.encodePacked("ab","c")`.

The ambiguity of `abi.encodePacked` can become problematic when it is used with `keccak256` to produce a hash that is expected to be unique. Dangerous hash collisions can silently occur which may render data structures inconsistent or corrupt.

When hashing data, `abi.encode` should be preferred over `abi.encodePacked`. Yet, `abi.encodePacked` can be simpler to reproduce in off-chain scripts than `abi.encode`. Therefore, should `abi.encodePacked` be used, the system must ensure that at most one of the data types passed to the function is dynamic to avoid potential hash collisions.

Further reference:

* [Use of `abi.encode` when hashing hashing and signing typed structured data in EIP 712](https://eips.ethereum.org/EIPS/eip-712#rationale-for-encodedata)
* [What are ABI encoding functions?](https://medium.com/@libertylocked/what-are-abi-encoding-functions-in-solidity-0-4-24-c1a90b5ddce8)
* [The trap of using packed encoding in Solidity](https://forum.openzeppelin.com/t/the-trap-of-using-encodepacked-in-solidity/1052)

### Example

The following contract implements a function to post asset prices on chain. However, as the final price identifier is built hashing two strings, there might be clashes in the identifier of different assets. For example, the pair "USD"-"TUSD" will have the same identifier as "USDT"-"USD", even though in reality they represent prices of different assets.

```text
// Vulnerable code - Do not use

contract SomeContract {

    mapping(bytes32 => uint256) public prices;

    function postPrice(string memory symbol1, string memory symbol2, uint256 price) external {
        bytes32 priceIdentifier = keccak256(abi.encodePacked(symbol1, symbol2));
        prices[priceIdentifier] = price;
    }

    ...
}
```

To fix this, one can replace `abi.encodePacked` with `abi.encode`.

```text

contract SomeContract {

    mapping(bytes32 => uint256) public prices;

    function postPrice(string memory symbol1, string memory symbol2, uint256 price) external {
        bytes32 priceIdentifier = keccak256(abi.encode(symbol1, symbol2));
        prices[priceIdentifier] = price;
    }

    ...
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/avoid-packed-encoding-when-hashing?](https://defender.openzeppelin.com/#/advisor/docs/avoid-packed-encoding-when-hashing?) 

