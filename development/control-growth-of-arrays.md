# Control Growth of Arrays

Operations over large arrays can make a transaction exceed the block gas limit. If no controls are in place, attackers may conduct Denial of Service \(DoS\) attacks by manipulating user-controlled arrays, making them grow until iteration becomes impossible.

## Description

A transaction's gas is bound by the block gas limit \(that is, how many units of gas can be spent per block\). This threshold is [constantly being adjusted](https://etherscan.io/chart/gaslimit) by miners. The number of transactions per block mainly depends on how much gas each transaction spends, but a transaction must be contained in a single block. As a consequence, gas-intensive transactions that do not fit in a block will be rejected. While transactions involving simple operations are likely to stay below the block gas limit, repeating operations numerous times in a single transaction might easily increase gas consumption above it.

Performing gas-heavy operations \(such as external calls or writing to storage\) while iterating over every element of large arrays is a common case likely to fail due to the block gas limit. This issue becomes particularly risky when the length of sensitive arrays can be manipulated by malicious users. If the length of the array is not programmatically bounded, an attacker may be able to conduct a Denial of Service attack by arbitrarily increasing the array's length until it becomes impossible to iterate.

Recommended practices vary depending on the specific business logic of each system, yet general action points to consider include:

* Programmatically enforcing upper bounds in the length of arrays.
* Restricting access to functions that increase the length of arrays, using the [`AccessControl` contract](https://docs.openzeppelin.com/contracts/3.x/access-control#role-based-access-control) from the OpenZeppelin Contracts library.
* Implementing functions that allow to act on a limited number of elements of the array.

## Example

The following contract allows anyone to register for a token airdrop. A privileged account is allowed to trigger the payment of the tokens to all registered beneficiaries. See that the array of beneficiaries is unbounded and can be manipulated by any malicious user. A Denial of Service attack can arbitrarily increase the length of the array of beneficiaries, permanently breaking the system.

```text
// Vulnerable code - Do not use

import "@openzeppelin/contracts/access/Ownable.sol";

contract SomeContract is Ownable {
    address[] public beneficiares;

    // Allows anyone to increase the length of the `beneficiaries` array
    function register() public {
        require(_notRegistered(account), "Account already registered");
        beneficiares.push(msg.sender);
    }

    function payUsers(ERC20 token, uint256 amount) public onlyOwner {
        // Iterating over unbounded array
        // If block gas limit is reached, no beneficiary will be paid
        for (uint256 i = 0; i < beneficiaries.length; i++) {
            require(token.transfer(beneficiaries[i], amount), "Transfer of tokens failed");
        }
    }

    function _notRegistered(address account) private { ... }

    ...
}
```

To fix the issue, the function in charge of paying beneficiaries can be tweaked to only work on a slice of the beneficiaries array in each transaction. Regardless of the number of beneficiaries, each call to `payUsers` can be made to stay below the block gas limit:

```text
import "@openzeppelin/contracts/access/Ownable.sol";

contract SomeContract is Ownable {
    address[] public beneficiares;

    function register() public {
        require(_notRegistered(account), "Account already registered");
        beneficiares.push(msg.sender);
    }

    function payUsers(ERC20 token, uint256 amount, uint256 from, uint256 to) public onlyOwner {
        require(to < beneficiaries.length, "to must be lower than number of beneficiaries");
        require(from <= to, "from must be lower than to");

        for (uint256 i = from; i <= to; i++) {
            require(token.transfer(beneficiaries[i], amount), "Transfer of tokens failed");
            ...
        }

        // Delete payees from array and resize accordingly
        _deRegister(from, to);
    }

    function _deRegister(uint256 from, uint256 to) private { ... }

    function _notRegistered(address account) private { ... }

    ...
}
```

Our example can be further improved by putting a hard cap on the number of users that can be registered. This value is set during construction, and can be later updated by the contract owner:

```text
import "@openzeppelin/contracts/access/Ownable.sol";

contract SomeContract is Ownable {
    address[] public beneficiares;
    uint256 private _maxBeneficiaries;

    constructor(uint256 maxBeneficiaries) public {
        _maxBeneficiaries = maxBeneficiaries;
    }

    function register(address account) public {
        require(beneficiaries.length <= _maxBeneficiaries, "Registration is closed");
        require(_notRegistered(account), "Account already registered");

        beneficiares.push(account);
    }

    function payUsers(ERC20 token, uint256 amount, uint256 from, uint256 to) public {
        require(hasRole(PAYER_ROLE, msg.sender), "Caller cannot execute payments");
        require(to < beneficiaries.length, "to must be lower than number of beneficiaries");

        for (uint256 i = from; i <= to; i++) {
            require(token.transfer(beneficiaries[i], amount), "Transfer of tokens failed");
            ...
        }

        // Delete payees from array and resize accordingly
        _deRegister(from, to);
    }

    function setMaxBeneficiaries(uint256 maxBeneficiaries) external onlyOwner {
        _maxBeneficiaries = maxBeneficiaries;
    }

    function _notRegistered(address account) private { ... }

    function _deRegister(uint256 from, uint256 to) private { ... }

    ...
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/control-growth-of-arrays?](https://defender.openzeppelin.com/#/advisor/docs/control-growth-of-arrays?)

