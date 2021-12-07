# Unchecked CALL Return Values

If the return value of a low-level call is not checked, the execution may resume even if the function call throws an error. This can lead to unexpected behaviour and break the program logic. A failed call can even be caused by an attacker, who may be able to further exploit the application.

In solidity, you can either use low-level calls such as: `address.call()`, `address.callcode()`, `address.delegatecall()`, and `address.send()`; or you can use contract calls such as: `ExternalContract.doSomething()`. Low-level calls will never throw an exception, instead they will return `false` if they encounter an exception, whereas contract calls will automatically throw.

In the case that you use low-level calls, be sure to check the return value to handle possible failed calls.

There are a number of ways of performing external calls in Solidity. Sending ether to external accounts is commonly performed via the transfer method. However, the send function can also be used, and for more versatile external calls the CALL opcode can be directly employed in Solidity. The `call` and `send` functions return a Boolean indicating whether the call succeeded or failed. Thus, these functions have a simple caveat, in that the transaction that executes these functions will not revert if the external call (intialized by call or send) fails; rather, the functions will simply return false. A common error is that the developer expects a revert to occur if the external call fails, and does not check the return value.

One of the deeper features of Solidity are the low level functions call(), callcode(), delegatecall() and send(). Their behavior in accounting for errors is quite different from other Solidity functions, as they will not propagate (or bubble up) and will not lead to a total reversion of the current execution. Instead, they will return a boolean value set to false, and the code will continue to run. This can surprise developers and, if the return value of such low-level calls are not checked, can lead to fail-opens and other unwanted outcomes. Remember, send can fail!

For further reading, see [#4 on the DASP Top 10 of 2018](http://www.dasp.co/#item-4) and “[Scanning Live Ethereum Contracts for the ‘Unchecked-Send’ Bug](http://bit.ly/2RnS1vA)”.

### The Vulnerability

Consider the following example:

contract Lotto {

&#x20;   bool public payedOut = false;\
&#x20;   address public winner;\
&#x20;   uint public winAmount;

&#x20;   // ... extra functionality here

&#x20;   function sendToWinner() public {\
&#x20;       require(!payedOut);\
&#x20;       winner.send(winAmount);\
&#x20;       payedOut = true;\
&#x20;   }

&#x20;   function withdrawLeftOver() public {\
&#x20;       require(payedOut);\
&#x20;       msg.sender.send(this.balance);\
&#x20;   }\
}

This represents a Lotto-like contract, where a winner receives winAmount of ether, which typically leaves a little left over for anyone to withdraw.

The vulnerability exists on line 11, where a send is used without checking the response. In this trivial example, a winner whose transaction fails (either by running out of gas or by being a contract that intentionally throws in the fallback function) allows payedOut to be set to true regardless of whether ether was sent or not. In this case, anyone can withdraw the winner’s winnings via the withdrawLeftOver function.

The following code is an example of what can go wrong when one forgets to check the return value of `send()`. If the call is used to send ether to a smart contract that does not accept them (e.g. because it does not have a payable fallback function), the EVM will replace its return value with false. Since the return value is not checked in our example, the function's changes to the contract state will not be reverted, and the etherLeft variable will end up tracking an incorrect value:

function withdraw(uint256 \_amount) public {\
&#x20;require(balances\[msg.sender] >= \_amount);\
&#x20;balances\[msg.sender] -= \_amount;\
&#x20;etherLeft -= \_amount;\
&#x20;msg.sender.send(\_amount);\
}

### Resources

[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#unchecked-call-return-values](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#unchecked-call-return-values)\
[Unchecked external call](https://github.com/trailofbits/not-so-smart-contracts/tree/master/unchecked\_external\_call)\
[Scanning Live Ethereum Contracts for the "Unchecked-Send" Bug](http://hackingdistributed.com/2016/06/16/scanning-live-ethereum-contracts-for-bugs/)

## Preventative Techniques

## Preventative Techniques

Whenever possible, use the transfer function rather than send, as transfer will revert if the external transaction reverts. If send is required, always check the return value.\
The use of low level "call" should be avoided whenever possible. It can lead to unexpected behavior if return values are not handled properly.\
A more robust [recommendation](http://bit.ly/2CSdF7y) is to adopt a withdrawal pattern. In this solution, each user must call an isolated withdraw function that handles the sending of ether out of the contract and deals with the consequences of failed send transactions. The idea is to logically isolate the external send functionality from the rest of the codebase, and place the burden of a potentially failed transaction on the end user calling the withdraw function.

## Real-World Examples

## Real-World Example: Etherpot and King of the Ether

[Etherpot](http://bit.ly/2OfHalK) was a smart contract lottery, not too dissimilar to the example contract mentioned earlier. The downfall of this contract was primarily due to incorrect use of block hashes (only the last 256 block hashes are usable; see Aakil Fernandes’s [post](http://bit.ly/2Jpzf4x) about how Etherpot failed to take account of this correctly). However, this contract also suffered from an unchecked call value.

Consider the function cash in [lotto.sol: Code snippet](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#lotto\_security).

...\
&#x20; function cash(uint roundIndex, uint subpotIndex){

&#x20;       var subpotsCount = getSubpotsCount(roundIndex);

&#x20;       if(subpotIndex>=subpotsCount)\
&#x20;           return;

&#x20;       var decisionBlockNumber = getDecisionBlockNumber(roundIndex,subpotIndex);

&#x20;       if(decisionBlockNumber>block.number)\
&#x20;           return;

&#x20;       if(rounds\[roundIndex].isCashed\[subpotIndex])\
&#x20;           return;\
&#x20;       //Subpots can only be cashed once. This is to prevent double payouts

&#x20;       var winner = calculateWinner(roundIndex,subpotIndex);\
&#x20;       var subpot = getSubpot(roundIndex);

&#x20;       winner.send(subpot);

&#x20;       rounds\[roundIndex].isCashed\[subpotIndex] = true;\
&#x20;       //Mark the round as cashed\
}\
...

Notice that on line 21 the send function’s return value is not checked, and the following line then sets a Boolean indicating that the winner has been sent their funds. This bug can allow a state where the winner does not receive their ether, but the state of the contract can indicate that the winner has already been paid.

A more serious version of this bug occurred in the King of the Ether. An excellent [post-mortem](https://www.kingoftheether.com/postmortem.html) of this contract has been written that details how an unchecked failed send could be used to attack the contract.

### Resources

* [https://www.kingoftheether.com/postmortem.html](https://www.kingoftheether.com/postmortem.html)
* [https://aakilfernandes.github.io/blockhashes-are-only-good-for-256-blocks](https://aakilfernandes.github.io/blockhashes-are-only-good-for-256-blocks)
* [https://blog.sigmaprime.io/solidity-security.html#unchecked-calls](https://blog.sigmaprime.io/solidity-security.html#unchecked-calls)
* [https://swcregistry.io/docs/SWC-104](https://swcregistry.io/docs/SWC-104)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/#handle-errors-in-external-calls](https://consensys.github.io/smart-contract-best-practices/recommendations/#handle-errors-in-external-calls)
