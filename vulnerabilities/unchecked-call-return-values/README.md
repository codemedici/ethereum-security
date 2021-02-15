# Unchecked CALL Return Values

## Unchecked CALL Return Values

## Unchecked CALL Return Values

There are a number of ways of performing external calls in Solidity. Sending ether to external accounts is commonly performed via the transfer method. However, the send function can also be used, and for more versatile external calls the CALL opcode can be directly employed in Solidity. The `call` and `send` functions return a Boolean indicating whether the call succeeded or failed. Thus, these functions have a simple caveat, in that the transaction that executes these functions will not revert if the external call \(intialized by call or send\) fails; rather, the functions will simply return false. A common error is that the developer expects a revert to occur if the external call fails, and does not check the return value.

One of the deeper features of Solidity are the low level functions call\(\), callcode\(\), delegatecall\(\) and send\(\). Their behavior in accounting for errors is quite different from other Solidity functions, as they will not propagate \(or bubble up\) and will not lead to a total reversion of the current execution. Instead, they will return a boolean value set to false, and the code will continue to run. This can surprise developers and, if the return value of such low-level calls are not checked, can lead to fail-opens and other unwanted outcomes. Remember, send can fail!

For further reading, see [\#4 on the DASP Top 10 of 2018](http://www.dasp.co/#item-4) and “[Scanning Live Ethereum Contracts for the ‘Unchecked-Send’ Bug](http://bit.ly/2RnS1vA)”.

### The Vulnerability

Consider the following example:

contract Lotto {

    bool public payedOut = false;  
    address public winner;  
    uint public winAmount;

    // ... extra functionality here

    function sendToWinner\(\) public {  
        require\(!payedOut\);  
        winner.send\(winAmount\);  
        payedOut = true;  
    }

    function withdrawLeftOver\(\) public {  
        require\(payedOut\);  
        msg.sender.send\(this.balance\);  
    }  
}

This represents a Lotto-like contract, where a winner receives winAmount of ether, which typically leaves a little left over for anyone to withdraw.

The vulnerability exists on line 11, where a send is used without checking the response. In this trivial example, a winner whose transaction fails \(either by running out of gas or by being a contract that intentionally throws in the fallback function\) allows payedOut to be set to true regardless of whether ether was sent or not. In this case, anyone can withdraw the winner’s winnings via the withdrawLeftOver function.

The following code is an example of what can go wrong when one forgets to check the return value of `send()`. If the call is used to send ether to a smart contract that does not accept them \(e.g. because it does not have a payable fallback function\), the EVM will replace its return value with false. Since the return value is not checked in our example, the function's changes to the contract state will not be reverted, and the etherLeft variable will end up tracking an incorrect value:

function withdraw\(uint256 \_amount\) public {  
 require\(balances\[msg.sender\] &gt;= \_amount\);  
 balances\[msg.sender\] -= \_amount;  
 etherLeft -= \_amount;  
 msg.sender.send\(\_amount\);  
}

### Resources

[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#unchecked-call-return-values](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#unchecked-call-return-values)  
[Unchecked external call](https://github.com/trailofbits/not-so-smart-contracts/tree/master/unchecked_external_call)  
[Scanning Live Ethereum Contracts for the "Unchecked-Send" Bug](http://hackingdistributed.com/2016/06/16/scanning-live-ethereum-contracts-for-bugs/)

