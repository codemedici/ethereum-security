# Implement Reentrancy Protections

Reentrancy is a consequence of a contract calling into or transferring Ether to other contracts. Care must be taken to ensure the caller only does this while in a consistent state, or to disallow reentrancy entirely.

## Description

External calls to untrusted contracts must be treated with utter care. Calling an external contract (including simple transfers of Ether) gives execution control to the receiver. When external calls are made before relevant state changes are applied, malicious contracts can call back before the first function execution is finished, leveraging inconsistent states in the victim contract to trigger undesired behaviors. This is called a "reentrancy attack", and has been succesfully exploited in the wild several times (including [the DAO hack](https://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/), [SpankChain](https://medium.com/spankchain/we-got-spanked-what-we-know-so-far-d5ed3a0f38fe) and [Uniswap v1 with ERC777](https://medium.com/@peckshield/uniswap-lendf-me-hacks-root-cause-and-loss-analysis-50f3263dcc09)).

Scenarios where multiple contracts interact with each other must be taken into account. Any call to a trusted contract may in turn execute calls to untrusted, malicious ones, that can execute reetrancy attacks.

There is a wide range of attack vectors depending on the specific business logic of the vulnerable contract, but the "state change after external call" anti-pattern is the common denominator among all reentrancy vulnerabilities.

To prevent this type of attacks, it used to be a common practice to limit the amount of gas forwarded to external calls. Today such practice is highly discouraged (see the best practice article **Do Not Use Solidity's `transfer` Function** for more details), but still there are at least two effective ways of protecting against reentrancy attacks.

## The Checks-Effects-Interactions Pattern

This [coding pattern](https://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) consists of structuring functions in three stages:

1. Check for all preconditions
2. Modify state variables
3. Make external calls

These steps ensure that all state transitions are settled before executing any external call. However, strictly following this pattern is not always possible, since state transitions may depend on the result of external calls. In these cases, reentrancy guards are recommended.

```
function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    userBalances[msg.sender] = 0;
    (bool success, ) = msg.sender.call.value(amountToWithdraw)(""); 
    // The user's balance is already 0,
    // so future invocations won't withdraw anything
    require(success);
}
```

## Reentrancy Guards

These are runtime checks that prohibit reentrancy by reverting execution when it is detected. They are useful when dealing with more convoluted cases where Checks-Effects-Interactions cannot be easily followed due to the logic's complexity. For a secure and efficient reentrancy guard, use the [`ReentrancyGuard` contract](https://docs.openzeppelin.com/contracts/3.x/api/utils#ReentrancyGuard) from the OpenZeppelin Contracts library.

Further reference:

* [Reentrancy after Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul)

## Examples

### Basic Reentrancy Bug

The `withdrawAllFunds` function of the following contract allows a user to withdraw all corresponding Ether in balance. However, a malicious contract can re-enter the function after receiveing its funds and steal all ETH in balance.

```
import "@openzeppelin/contracts/utils/Address.sol"

contract SomeContract {

    using Address for address payable;

    mapping(address => uint256) public balances;

    // Vulnerable code - Do not use
    function withdrawAllFunds() external {
        uint256 amount = balances[msg.sender];
        msg.sender.sendValue(amount);
        balances[msg.sender] = 0;
    }
}

contract Attacker {
    SomeContract private victim;

    receive() external payable {
        if (address(victim).balance >= victim.balances(address(this))) {
            // reenter victim if there are still funds to claim
            victim.withdrawAllFunds();
        }
   }
}
```

To fix this, follow the Checks-Effects-Interactions pattern.

```

import "@openzeppelin/contracts/utils/Address.sol"

contract SomeContract {

    using Address for address payable;

    function withdrawAllFunds() external {
        uint256 amount = balances[msg.sender];

        // Effect
        balances[msg.sender] = 0;

        // Interaction
        msg.sender.sendValue(amount);
        // An attacker can still reenter at this point, but the balances
        // have already been updated
    }
}
```

### Reentrancy in Flash Loans

Lending pools offering flash loans always call an untrusted contract after the flash-loaned funds are transferred, but before the pool's balance is verified to ensure the loan's been paid back. If the pool accounts all fund deposits as repayments, a reentrancy attack during a flash loan can allow anyone to drain all funds from a lending pool.

In the following example, any receiver of a flash loan can use the borrowed ETH to re-enter the victim contract via its `deposit` function, thus increasing the contract's balance. The flash loan will be considered repaid, and later the attacker can withdraw the funds via the `withdraw` function, effectively stealing funds from the vulnerable pool.

```
import "@openzeppelin/contracts/utils/Address.sol";

interface IFlashLoanEtherReceiver {
    function execute() external payable;
}

// Vulnerable code - Do not use
contract LendingPool {
    using Address for address payable;

    mapping (address => uint256) private balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amountToWithdraw = balances[msg.sender];
        balances[msg.sender] = 0;
        msg.sender.sendValue(amountToWithdraw);
    }

    function flashLoan(uint256 amount) external {
        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= amount, "Not enough ETH in balance");

        IFlashLoanEtherReceiver(msg.sender).execute{value: amount}();

        require(address(this).balance >= balanceBefore, "Flash loan hasn't been paid back");
    }
}
```

To fix this, we can mark all functions with the `nonReentrant` modifer to avoid any kind of unexpected behaviors.

```
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

interface IFlashLoanEtherReceiver {
    function execute() external payable;
}

contract LendingPool is ReentrancyGuard {
    using Address for address payable;

    mapping (address => uint256) private balances;

    function deposit() external payable nonReentrant {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external nonReentrant {
        uint256 amountToWithdraw = balances[msg.sender];
        balances[msg.sender] = 0;
        msg.sender.sendValue(amountToWithdraw);
    }

    function flashLoan(uint256 amount) external nonReentrant {
        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= amount, "Not enough ETH in balance");

        IFlashLoanEtherReceiver(msg.sender).execute{value: amount}();

        require(address(this).balance >= balanceBefore, "Flash loan hasn't been paid back");
    }
}
```

### Reentrancy in Governance

Contracts that execute governance actions most likely execute external calls to other, potentially malicious, contracts. These actions are usually approved by the governance mechanism, so the likelihood of a reentrancy vulnerability being exploited is usually lower than in other scenarios, but can still be considered a valid attack vector.

A real example was found in [UMA](https://umaproject.org)'s governance during OpenZeppelin's [phase 1 audit](https://blog.openzeppelin.com/uma-audit-phase-1/) of their system, reported in "\[H03] Any governance action could be executed multiple times". The fix for the vulnerability consists of following the Checks-Effects-Interactions pattern so as to delete a governance proposal _before_ its execution. This prevents malicious contracts from re-entering the governance contract and unexpectedly executing an action several times.

Based on findings, UMA replaced this code:

```
require(_executeCall(transaction.to, transaction.value, transaction.data), "Transaction execution failed");

// Delete the transaction.
```

With this code:

```
// Delete the transaction before execution to avoid any potential re-entrancy issues.
delete proposal.transactions[transactionIndex];

require(_executeCall(transaction.to, transaction.value, transaction.data), "Transaction execution failed");
```

### Reentrancy in Uniswap v1

The [Uniswap v1 contracts](https://github.com/Uniswap/uniswap-v1/) have proven to be vulnerable to a reentrancy vulnerability when the exchange is trading [ERC777](https://eips.ethereum.org/EIPS/eip-777) tokens. The exploit relies on the fact that ERC777 tokens execute hooks on the sender and receiver contracts before and after a transfer of tokens is made.

The vulnerability in Uniswap was [first reported by ConsenSys Dilligence](https://github.com/ConsenSys/Uniswap-audit-report-2018-12#31-liquidity-pool-can-be-stolen-in-some-tokens-eg-erc-777-29), later [explained with a proof-of-concept exploit by OpenZeppelin](https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit/), and finally [exploited in the wild](https://medium.com/@peckshield/uniswap-lendf-me-hacks-root-cause-and-loss-analysis-50f3263dcc09).

## Resources

* [https://defender.openzeppelin.com/#/advisor/docs/implement-reentrancy-protections](https://defender.openzeppelin.com/#/advisor/docs/implement-reentrancy-protections)
