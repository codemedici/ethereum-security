---
description: >-
  Avoid explicit integer downcasts and typecasts from signed to unsigned without
  first checking error conditions.
---

# Recast Variables Safely

### Description

Downcasts and typecasts are sometimes necessary when dealing with mixed integer types, for example in arithmetic operations that might overflow the base type in intermediate steps.

Unfortunately, Solidity does not raise any errors on explicit downcasts where the destination type is not large enough to hold the value. For example, `uint8(300)` will silently evaluate to `44` with no errors or warnings. The same is true for sign typecasts: `uint8(-1)` evaluates to `255`. These types of unexpected results can lead to security issues, miscalculated balances or the loss of user funds.

It is critical therefore to provide defensive measures when dealing with explicit casts. The OpenZeppelin Contracts [`SafeCast` library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) provides helper functions to safely downcast integers and change their sign, reverting on errors. Avoid any explicit casts, and use these functions instead.

### Example 1

In this example we attempt to downcast from `uint256` to `uint64`. However, if `input > uint64(-1)` then this `downcast` function will not return a value equal to the input.

```text
// VULNERABLE CODE - DO NOT USE
contract SomeContract {

    function downcast(uint256 input) public pure returns (uint64) {
        return uint64(input);
    }

}
```

In the fixed version we use the `SafeCast` to downcast safely. If `input` is too large, then `SafeCast` reverts.

```text
import "@openzeppelin/contracts/utils/SafeCast.sol";

contract SomeContract {

    function downcast(uint256 input) public pure returns (uint64) {
        return SafeCast.toUint64(input);
    }

}
```

### Example 2

In this example we attempt to cast from `int256` to `uint256`. However, if `input < 0` then this `recast` function will not return a value equal to the input.

```text
// VULNERABLE CODE - DO NOT USE
contract SomeContract {

    function recast(int256 input) public pure returns (uint256) {
        return uint256(input);
    }

}
```

In the fixed version we write a `safeIntToUint` function and use it recast safely. If `input` is negative, then `safeIntToUint` reverts.

```text

contract SomeContract {

    function safeIntToUint(int256 input) public pure returns (uint256) {
        require(input >= 0, "input is negative");
        return uint256(input);
    }

    function recast(int256 input) public pure returns (uint256) {
        return safeIntToUint(input);
    }

}
```

