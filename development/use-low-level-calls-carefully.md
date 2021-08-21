# Use Low-Level Calls Carefully

Solidity's [low-level calls](https://solidity.readthedocs.io/en/latest/types.html#members-of-addresses) have edge cases developers must consider. Success values must be checked, returned data be handled carefully, and different behaviors for contracts and externally-owned accounts must be taken into account and be thoroughly tested.

This practice covers use of `call` and `staticcall` and more secure alternatives to them.

### Description

In Solidity, all `address` types include the [`call` and `staticcall` functions](https://solidity.readthedocs.io/en/latest/types.html#members-of-addresses) that can be used to execute arbitrary calls, optionally passing a payload of bytes. By default both functions forward all available gas, and return the success condition and any returned data if available. When the call is not expected to modify state, `staticcall` should be used instead of `call`.

While these low-level functions do have their use cases, they should be avoided when more straightforward and safer options are at hand. There are a number of scenarios in which they are typically used, and there are usually more secure alternatives for them.

| Scenario | Recommendation |
| :--- | :--- |
| Send Ether forwarding all available gas | Use [`sendValue`](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-sendValue-address-payable-uint256-) from OpenZeppelin Contracts \(detailed in **Do Not Use Solidity's `transfer` Function**\) |
| Execute contract calls when no typed interface is available, reverting on failure | Use [`functionCall`](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-functionCall-address-bytes-) from OpenZeppelin Contracts |
| Use conditional logic based on call success \(when a typed interface is available\) | Use Solidity's `try-catch` \(detailed in **Handle Call Failures Using Try/Catch**\) |
| Use conditional logic based on call success \(when no typed interface is available\) | Use `call` and `staticcall` following the recommendations outlined in this article |

Both low-level functions bypass important checks the Solidity compiler applies for regular function calls:

1. The target address is checked to verify it contains contract code.
2. The call itself is made and success asserted, bubbling up revert reasons upon failure.
3. The return value is decoded, which in turn checks the size of the returned data.

All these checks and operations are skipped when using `call` and `staticcall`, so they should be done manually by developers.

Following we include a step-by-step guide on how to use conditional logic based on call success when no typed interface is available - _the only scenario under which these low-level functions should be used_.

#### 1. Verify that target contains contract code

Developers should check that the target address is indeed a deployed contract with code. Low-level calls to accounts with no code will always suceed \(return `true`\) despite not having performed any operations. This is a peculiarity of the EVM: execution on these accounts does not result in an error but instead immediately comes to a valid STOP operation \(see [section 9.4.1 of the yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf#subsection.9.4)\).

Therefore, developers must ensure the target address contains contract code. This can be done with the [`isContract` function](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-isContract-address-) of the `Address` library from OpenZeppelin Contracts.

```text
import "@openzeppelin/contracts/utils/Address.sol";

contract SomeContract {
    using Address for address;

    function _foo(address targetAddress) internal {
        require(targetAddress.isContract(), "Call to non-contract");
        ...
    }
}
```

#### 2. Handle success condition

The first value returned by low-level calls is a boolean flag that signals whether the call has succeeded. A `false` value \(representing the "failed" status\) means that the call reverted and had no side effect, but does not cause the entire transaction to be reverted as a regular function call would. Therefore, the success flag returned by the call must always be handled appropriately.

```text
import "@openzeppelin/contracts/utils/Address.sol";

contract SomeContract {
    using Address for address;

    function _foo(address targetAddress, bytes memory data) internal {
        require(targetAddress.isContract(), "Call to non-contract");

        (bool success, ) = targetAddress).call(data);

        if (success) {
            ...
        } else {
            ...
        }
    }
}
```

#### 3. Handle returned data if available

The second value returned by a low-level call is the returned data, if any is available. When appropriate, such data should be checked and decoded accordingly.

On success, returned data can be decoded using `abi.decode` \(assuming the type of the returned data is known\). Note that a revert will be triggered if the returned data cannot be decoded to the target type.

```text
import "@openzeppelin/contracts/utils/Address.sol";

contract SomeContract {
    using Address for address;

    function _foo(address targetAddress, bytes memory data) internal {
        require(targetAddress.isContract(), "Call to non-contract");

        (bool success, bytes memory returndata) = targetAddress).call(data);

        if (success) {
            // Assuming the function call returned a single `uint256` value
            // This will revert if `returndata` cannot be decoded to a `uint256`
            uint256 value = abi.decode(returndata, (uint256));
            ...
        } else {
            ...
        }
    }
}
```

On failure, the returned data can contain a reason string \(coming from a `require` or `revert` statement\). If needed, low-level assembly must be used to recover it, taking into account [how error strings returned from `revert` and `require` are abi-encoded](https://solidity.readthedocs.io/en/latest/control-structures.html#revert).

You can see [this example from OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.0.2/contracts/token/ERC721/ERC721.sol#L508-L517) on how the revert reason can be recovered using assembly. When you execute calls with the [`functionCall` function](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-functionCall-address-bytes-) from OpenZeppelin contracts, this is done automatically for you.

### Examples

#### Example on propagating revert reasons

In the following contract, the function `execute` uses the bytes returned from the external call as the revert reason when propagating the error. However, this contract does not take into account [how error strings returned from `revert` and `require` are abi-encoded](https://solidity.readthedocs.io/en/latest/control-structures.html#revert). The resulting revert reason will therefore be malformed and potentially cause problems in off-chain clients.

```text
// Inadequate code - Do not use
contract Test {

    function execute() public {
        Target target = new Target();
        (bool success, bytes memory returndata) = address(target).call(abi.encodeWithSignature("foo()"));

        require(success, string(returndata));
    }
}

contract Target {
    function foo() public pure {
        revert("Target::foo reverted");
    }
}
```

The malformed revert reason, as shown by Truffle, is:

```text
Error: Returned error: VM Exception while processing transaction: revertï¿½yï¿½ Target::foo reverted -- Reason given:y Target::foo reverted.
```

To fix this, the [`functionCall` function](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-functionCall-address-bytes-) available in OpenZeppelin Contracts should be used to avoid manually handling revert reasons:

```text

import "@openzeppelin/contracts/utils/Address.sol";

contract Test {

    using Address for address;

    function execute() public {

        Target target = new Target();

        bytes memory returndata = address(target).functionCall(
            abi.encodeWithSignature("foo()")
        );

        ...
    }
}

contract Target {
    function foo() public pure {
        revert("Target::foo reverted");
    }
}
```

Now the revert reason is accurately propagated:

```text
Error: Returned error: VM Exception while processing transaction: revert Target::foo reverted -- Reason given: Target::foo reverted.
```

#### Example on passing data in calls to externally-owned accounts

During [OpenZeppelin's first audit for UMA](https://blog.openzeppelin.com/uma-audit-phase-1/), we found that their governance mechanism would allow executing calls over other accounts with arbitrary data. No exploitable attack vector was identified, yet the system would behave in an unexpected way when the call's target was an externally-owned account and data would be sent along with it. In particular, the external call would succeed even if no code was actually executed.

Their fix, as can be seen in [PR\#1242](https://github.com/UMAprotocol/protocol/pull/1242), entails ensuring that if a proposal in the governance mechanism includes data, then the target address must be a deployed contract. For more details, head to issue "\[L15\] Actions in proposals might fail silently when target is an externally-owned account" in the audit report.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/use-low-level-calls-carefully?](https://defender.openzeppelin.com/#/advisor/docs/use-low-level-calls-carefully?)

