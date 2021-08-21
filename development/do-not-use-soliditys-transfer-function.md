# Do Not Use Solidity's transfer Function

Solidity's `transfer` and `send` functions forward a limited amount of gas as a simple way to prevent contract reentrancy. However, hard dependencies on specific gas costs might render contracts unusable after hard forks. Use OpenZeppelin's [`sendValue` function](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-sendValue-address-payable-uint256-) instead, avoiding reentrancies with the Checks-Effects-Interactions pattern and reentrancy guards.

### Description

Transfers of Ether used to be executed with Solidity's `transfer` or `send` functions, which forward a limited amount of gas to the receiver. However, with the inclusion of [EIP 1884](https://eips.ethereum.org/EIPS/eip-1884) in the Istanbul hard fork, it is now a best practice to not use `transfer` nor `send` to avoid hard dependencies on specific gas costs. Use of these functions becomes problematic when the receiver is a contract with a fallback function: due to EVM gas repricings, it may in the future require more than 2300 units of gas to execute. This can lead to entire components becoming unusable and loss of funds. Examples of systems affected by the Istanbul gas repricings include [OpenZeppelin upgradeable contracts](https://forum.openzeppelin.com/t/openzeppelin-upgradeable-contracts-affected-by-istanbul-hardfork/1616) and [Aragon organizations](https://aragon.org/blog/istanbul-hard-fork-impact).

Usage of `transfer` and `send` functions is discouraged. They should be replaced with the [`sendValue` function](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-sendValue-address-payable-uint256-) from the OpenZeppelin Contracts library, which forwards all available gas and reverts on errors.

Risks of reentrancy attacks stemming from forwarding all available gas in external calls must be mitigated by tightly following the “Check-effects-interactions” pattern and using reentrancy guards where appropriate. Find more details about how to protect against reentrancy in the development best practice **Implement Reentrancy Protections**.

Further reference:

* [Stop using Solidity's transfer now](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/)
* [Detailed list of contracts affected by Istanbul hardfork](https://gist.github.com/ritzdorf/1c6bd72955391e831f8a397d3152b4e0)
* [Reentrancy after Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul)

### Example

In this example the contract attempts to prevent a reentrancy by the sole use of `transfer`. However, the transfer of ETH will fail if the receiver is a contract with a payable fallback function that requires more than 2300 units of gas, thus not allowing a user to withdraw their funds.

```text
// Vulnerable code - Do not use
contract SomeContract {

    mapping(address => uint256) private balances;

    function withdraw(uint256 amount) external {
        require(amount <= balances[msg.sender]);
        msg.sender.transfer(amount);
        balances[msg.sender] -= amount;
    }
    ...
}
```

In the fixed version we use the `sendValue` function from OpenZeppelin Contracts, tightly following the Checks-Effects-Interactions pattern to prevent a reentrancy bug.

```text
import "@openzeppelin/contracts/utils/Address.sol";

contract SomeContract {

    using Address for address payable;

    mapping(address => uint256) private balances;

    function withdraw(uint256 amount) external {
        // Check
        require(amount <= balances[msg.sender], "Not enough balance");

        // Effect
        balances[msg.sender] -= amount;

        // Interaction
        msg.sender.sendValue(amount);
    }
    ...
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/do-not-use-soliditys-transfer-function?](https://defender.openzeppelin.com/#/advisor/docs/do-not-use-soliditys-transfer-function?)

