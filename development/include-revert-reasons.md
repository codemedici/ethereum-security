# Include Revert Reasons

{% hint style="success" %}
Informative revert reasons should be included in all Solidity's error-handling statements \(such as `require` and `revert`\) to favor readability and ease debugging, maintenance and testing. Keeping revert reasons in production environments is recommended.
{% endhint %}

## Description

Solidity uses [state-reverting exceptions](https://solidity.readthedocs.io/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions) to handle errors, which roll back all changes made to the state in the current call \(and all its sub-calls\), thus flagging an error to the caller. In particular, the `require` and `revert` statements allow developers to include specific revert reasons using strings.

Revert reasons are intended to notify users about failing conditions, and should provide enough information so that the appropriate corrections needed to interact with the system can be applied. Otherwise the overall developer experience is hindered, lowering the system's quality.

All reasons must be informative and clear, reflecting the exact failing conditions. This eases debugging, and favors readability and maintenance of the project, helping reduce the number of bugs. Furthermore, revert reasons are particularly beneficial for testing, where asserting for specific revert reasons becomes crucial to avoid false positives \(as explained in the **Assert Revert Reasons** testing practice\). No specific format is advised for the strings, though aiming for uniqueness \(by adding the name of the contract and/or function\) can benefit traceability.

Revert reasons increase the size of the deployed bytecode, so keep the strings succint to reduce deployment costs in mainnet. But completely removing revert reasons before production deployment using the compiler's `--revert-strings` flag is discouraged. Currently, removing these strings can break verification in platforms that still do not support the compiler's flag.

Having revert reasons in production might also facilitate inspection of failed transactions. Note however that this is still not trivial to implement \(as can be seen in [this Geth issue](https://github.com/ethereum/go-ethereum/issues/19027), [its related PR](https://github.com/ethereum/go-ethereum/pull/21083) and this [article by Authereum](https://medium.com/authereum/getting-ethereum-transaction-revert-reasons-the-easy-way-24203a4d1844)\), specially without access to an [archive node](https://docs.ethhub.io/using-ethereum/running-an-ethereum-node/). However, some platforms like Tenderly can [show failing conditions](https://dashboard.tenderly.co/tx/kovan/0x85e51491d239f71159a49e700a6043d16b3ff3283170d496dbda0f36b37d4a3b) by displaying the offending Solidity line.

## Example

In the following example, if the caller is not the owner or the amount is zero, the call to the `deposit` function will fail. Yet the lack of revert reasons will not allow the caller to understand the exact failing condition.

```text
// Inadequate code - Do not use
contract SomeContract {
    address private owner;

    function deposit(uint256 amount) external {
        require(msg.sender == owner);
        require(amount > 0);
        // ...
    }
}
```

Adding specific and informative revert reasons solves the problem.

```text
contract SomeContract {
    address private owner;

    function deposit(uint256 amount) external {
        require(msg.sender == owner, "Caller must be owner");
        require(amount > 0, "Deposited amount must be greater than zero");
        // ...
    }
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/include-revert-reasons?](https://defender.openzeppelin.com/#/advisor/docs/include-revert-reasons?)

