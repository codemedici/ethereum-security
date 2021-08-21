# Use PullPayment When Sending ETH

{% hint style="success" %}
Avoid making external calls to transfer ETH. Instead, have the intended recipient of the funds call your contract to withdraw them.
{% endhint %}

## Description

External calls to unknown addresses expose your contract to reentrancy issues and reverted transactions due to gas overusage or downstream failures. To avoid exposing your contract to these issues, you can use the Pull Payment strategy.

The idea behind the Pull Payment strategy is that instead of “pushing” funds to a receiver, they have to be “pulled” out of a contract.

You can find an implementation of this strategy in the [OpenZeppelin Contracts library](https://docs.openzeppelin.com/contracts/3.x/api/payment#PullPayment). Inheriting this contract makes an internal function available, `_asyncTransfer`, that is analogous to `transfer`. The difference is that, instead of sending the funds to the receiver, it will transfer them to an escrow contract. Additionally, `PullPayment` provides a public function for receivers to withdraw \(pull\) their funds: `withdrawPayments`.

It’s worth mentioning that the withdrawal function can be called by anyone, and not just the receiver. This means that the receiver doesn’t need to be aware that it is the target of a pull payment, which is particularly important when it’s an existing smart contract that is unable to pull for itself.

Further reference:

* [Favor pull over push for external calls](https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls)
* [OpenZeppelin's PullPayment contract documentation](https://docs.openzeppelin.com/contracts/3.x/api/payment#PullPayment)
* [Reentrancy after Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)

## Example

In this example the contract attempts to prevent a reentrancy by the sole use of transfer. However, the transfer of ETH will fail if the receiver is a contract with a payable fallback function that requires more than 2300 units of gas, thus not allowing a user to withdraw their funds.

```text
// Vulnerable code - Do not use
contract Payer {

    mapping(address => uint256) private balances;

    function pay(uint256 amount) external {
        require(amount <= balances[msg.sender]);
        msg.sender.transfer(amount);
        balances[msg.sender] -= amount;
    }
    ...

}
```

In the fixed version below we make our `Payer` contract inherit from `PullPayment`, which implements the Pull Payment strategy. We simply replace calls to `transfer` with equivalent calls to `_asyncTransfer`. Payees can then call the `payments` function to check whether they have any funds available to withdraw, and the `withdrawPayments` to get those funds.

```text
import "@openzeppelin/contracts/payments/PullPayment.sol";

contract Payer is PullPayment {

    mapping(address => uint256) private balances;
        
    function pay(uint256 amount) payable external {
      require(amount <= balances[msg.sender]);      

      balances[msg.sender] -= amount;

      // Funds are transferred to an escrow account,
      // payee gets the funds after calling `withdrawPayments`.
      //
      // This operation is completely controlled by this contract,
      // so it isn't exposed to reentrancy issues or reversions caused
      // by external contracts.
      _asyncTransfer(msg.sender, amount);
    }
    ...

}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/use-pull-payment-when-sending-eth?](https://defender.openzeppelin.com/#/advisor/docs/use-pull-payment-when-sending-eth?)

