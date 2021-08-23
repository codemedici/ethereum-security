# Declare Constants Explicitly

{% hint style="success" %}
Avoid using literal values with unexplained meaning. Instead, replace such values with constant variables that have clear, explicit and self-explanatory names, so as to increase readability and ease maintenance.
{% endhint %}

## Description

Literal values in the codebase without an explained meaning make the code harder to read, understand and maintain, thus hindering the experience of developers, auditors and external contributors alike.

Developers should define a constant variable for every magic value used \(including booleans\), giving it a clear and self-explanatory name. Additionally, for complex values, inline comments explaining how they were calculated or why they were chosen are highly recommended. Following [Solidity's style guide](https://solidity.readthedocs.io/en/latest/style-guide.html#constants), constants should be named in `UPPER_CASE_WITH_UNDERSCORES` format, and specific public getters should be defined to read each one of them.

## Example

The following contract only allows executing a function after a certain period of time has passed, yet it is not straightforward to understand what the number 950400 represents.

```text
// Inadequate code - Do not use
contract SomeContract {

    uint256 private immutable initialTime;

    constructor() public {
        initialTime = block.timestamp;
    }

    function foo() public {
        require(block.timestamp - initialTime >= 950400);
        ...
    }
}
```

By simply declaring the state variable as `constant` in the contract, the code becomes much more readable, and it is straightforward to understand the precondition to execute the function.

```text
contract SomeContract {

    uint256 private immutable initialTime;
    uint256 private constant EXECUTION_DELAY_IN_SECONDS = 11 days;

    constructor() public {
        initialTime = block.timestamp;
    }

    function getExecutionDelayInSeconds() external view returns (uint256) {
        return EXECUTION_DELAY_IN_SECONDS;
    }

    function foo() public {
        require(block.timestamp - initialTime >= EXECUTION_DELAY_IN_SECONDS);
        ...
    }
}
```

Note that `EXECUTION_DELAY_IN_SECONDS` could be declared as [`immutable`](https://solidity.readthedocs.io/en/latest/contracts.html#constant-and-immutable-state-variables) instead of `constant`. In some cases, `immutable` can perform better than `constant` in terms of gas costs.

```text
contract SomeContract {

    uint256 private immutable initialTime;
    uint256 private immutable EXECUTION_DELAY_IN_SECONDS = 11 days;

    ...
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/declare-constants-explicitly?](https://defender.openzeppelin.com/#/advisor/docs/declare-constants-explicitly?)

