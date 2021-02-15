# Protocol specific recommendations

## Protocol specific recommendations

## Protocol specific recommendations¶

[https://consensys.github.io/smart-contract-best-practices/recommendations/\#protocol-specific-recommendations](https://consensys.github.io/smart-contract-best-practices/recommendations/#protocol-specific-recommendations)

The following recommendations apply to the development of any contract system on Ethereum.

External Calls¶  
Use caution when making external calls¶  
Calls to untrusted contracts can introduce several unexpected risks or errors. External calls may execute malicious code in that contract or any other contract that it depends upon. As such, every external call should be treated as a potential security risk. When it is not possible, or undesirable to remove external calls, use the recommendations in the rest of this section to minimize the danger.

### Mark untrusted contracts

When interacting with external contracts, name your variables, methods, and contract interfaces in a way that makes it clear that interacting with them is potentially unsafe. This applies to your own functions that call external contracts.

// bad  
Bank.withdraw\(100\); // Unclear whether trusted or untrusted

function makeWithdrawal\(uint amount\) { // Isn't clear that this function is potentially unsafe  
    Bank.withdraw\(amount\);  
}

// good  
UntrustedBank.withdraw\(100\); // untrusted external call  
TrustedBank.withdraw\(100\); // external but trusted bank contract maintained by XYZ Corp

function makeUntrustedWithdrawal\(uint amount\) {  
    UntrustedBank.withdraw\(amount\);  
}

Avoid state changes after external calls¶  
Whether using raw calls \(of the form someAddress.call\(\)\) or contract calls \(of the form ExternalContract.someMethod\(\)\), assume that malicious code might execute. Even if ExternalContract is not malicious, malicious code can be executed by any contracts it calls.

One particular danger is malicious code may hijack the control flow, leading to vulnerabilities due to reentrancy. \(See Reentrancy for a fuller discussion of this problem\).

If you are making a call to an untrusted external contract, avoid state changes after the call. This pattern is also sometimes known as the checks-effects-interactions pattern.

Be aware of the tradeoffs between send\(\), transfer\(\), and call.value\(\)\(\)¶  
When sending ether be aware of the relative tradeoffs between the use of someAddress.send\(\), someAddress.transfer\(\), and someAddress.call.value\(\)\(\).

someAddress.send\(\)and someAddress.transfer\(\) are considered safe against reentrancy. While these methods still trigger code execution, the called contract is only given a stipend of 2,300 gas which is currently only enough to log an event.  
x.transfer\(y\) is equivalent to require\(x.send\(y\)\);, it will automatically revert if the send fails.  
someAddress.call.value\(y\)\(\) will send the provided ether and trigger code execution.  
The executed code is given all available gas for execution making this type of value transfer unsafe against reentrancy.  
Using send\(\) or transfer\(\) will prevent reentrancy but it does so at the cost of being incompatible with any contract whose fallback function requires more than 2,300 gas. It is also possible to use someAddress.call.value\(ethAmount\).gas\(gasAmount\)\(\) to forward a custom amount of gas.

One pattern that attempts to balance this trade-off is to implement both a push and pull mechanism, using send\(\) or transfer\(\) for the push component and call.value\(\)\(\) for the pull component.

It is worth pointing out that exclusive use of send\(\) or transfer\(\) for value transfers does not itself make a contract safe against reentrancy but only makes those specific value transfers safe against reentrancy.

Handle errors in external calls¶  
Solidity offers low-level call methods that work on raw addresses: address.call\(\), address.callcode\(\), address.delegatecall\(\), and address.send\(\). These low-level methods never throw an exception, but will return false if the call encounters an exception. On the other hand, contract calls \(e.g., ExternalContract.doSomething\(\)\) will automatically propagate a throw \(for example, ExternalContract.doSomething\(\) will also throw if doSomething\(\) throws\).

If you choose to use the low-level call methods, make sure to handle the possibility that the call will fail, by checking the return value.

// bad  
someAddress.send\(55\);  
someAddress.call.value\(55\)\(\); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result  
someAddress.call.value\(100\)\(bytes4\(sha3\("deposit\(\)"\)\)\); // if deposit throws an exception, the raw call\(\) will only return false and transaction will NOT be reverted

// good  
if\(!someAddress.send\(55\)\) {  
 // handle failure code  
}

ExternalContract\(someAddress\).deposit.value\(100\);  
Favor pull over push for external calls¶  
External calls can fail accidentally or deliberately. To minimize the damage caused by such failures, it is often better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically. \(This also reduces the chance of problems with the gas limit.\) Avoid combining multiple transfer\(\) calls in a single transaction.

// bad  
contract auction {  
 address highestBidder;  
 uint highestBid;

 function bid\(\) payable {  
 require\(msg.value &gt;= highestBid\);

 if \(highestBidder != address\(0\)\) {  
 highestBidder.transfer\(highestBid\); // if this call consistently fails, no one else can bid  
 }

 highestBidder = msg.sender;  
 highestBid = msg.value;  
 }  
}

// good  
contract auction {  
 address highestBidder;  
 uint highestBid;  
 mapping\(address =&gt; uint\) refunds;

 function bid\(\) payable external {  
 require\(msg.value &gt;= highestBid\);

 if \(highestBidder != address\(0\)\) {  
 refunds\[highestBidder\] += highestBid; // record the refund that this user can claim  
 }

 highestBidder = msg.sender;  
 highestBid = msg.value;  
 }

 function withdrawRefund\(\) external {  
 uint refund = refunds\[msg.sender\];  
 refunds\[msg.sender\] = 0;  
 msg.sender.transfer\(refund\);  
 }  
}  
Don't delegatecall to untrusted code¶  
The delegatecall function is used to call functions from other contracts as if they belong to the caller contract. Thus the callee may change the state of the calling address. This may be insecure. An example below shows how using delegatecall can lead to the destruction of the contract and loss of its balance.

contract Destructor  
{  
 function doWork\(\) external  
 {  
 selfdestruct\(0\);  
 }  
}

contract Worker  
{  
 function doWork\(address \_internalWorker\) public  
 {  
 // unsafe  
 \_internalWorker.delegatecall\(bytes4\(keccak256\("doWork\(\)"\)\)\);  
 }  
}  
If Worker.doWork\(\) is called with the address of the deployed Destructor contract as an argument, the Worker contract will self-destruct. Delegate execution only to trusted contracts, and never to a user supplied address.

Warning

Don't assume contracts are created with zero balance An attacker can send ether to the address of a contract before it is created. Contracts should not assume that its initial state contains a zero balance. See issue 61 for more details.

Remember that Ether can be forcibly sent to an account¶  
Beware of coding an invariant that strictly checks the balance of a contract.

An attacker can forcibly send ether to any account and this cannot be prevented \(not even with a fallback function that does a revert\(\)\).

The attacker can do this by creating a contract, funding it with 1 wei, and invoking selfdestruct\(victimAddress\). No code is invoked in victimAddress, so it cannot be prevented. This is also true for

Remember that on-chain data is public¶  
Many applications require submitted data to be private up until some point in time in order to work. Games \(eg. on-chain rock-paper-scissors\) and auction mechanisms \(eg. sealed-bid Vickrey auctions\) are two major categories of examples. If you are building an application where privacy is an issue, make sure you avoid requiring users to publish information too early. The best strategy is to use commitment schemes with separate phases: first commit using the hash of the values and in a later phase revealing the values.

Examples:

In rock paper scissors, require both players to submit a hash of their intended move first, then require both players to submit their move; if the submitted move does not match the hash throw it out.  
In an auction, require players to submit a hash of their bid value in an initial phase \(along with a deposit greater than their bid value\), and then submit their auction bid value in the second phase.  
When developing an application that depends on a random number generator, the order should always be \(1\) players submit moves, \(2\) random number generated, \(3\) players paid out. The method by which random numbers are generated is itself an area of active research; current best-in-class solutions include Bitcoin block headers \(verified through http://btcrelay.org\), hash-commit-reveal schemes \(ie. one party generates a number, publishes its hash to "commit" to the value, and then reveals the value later\) and RANDAO. As Ethereum is a deterministic protocol, no variable within the protocol could be used as an unpredictable random number. Also be aware that miners are in some extent in control of the block.blockhash\(\) value\*.  
Beware of the possibility that some participants may "drop offline" and not return¶  
Do not make refund or claim processes dependent on a specific party performing a particular action with no other way of getting the funds out. For example, in a rock-paper-scissors game, one common mistake is to not make a payout until both players submit their moves; however, a malicious player can "grief" the other by simply never submitting their move - in fact, if a player sees the other player's revealed move and determines that they lost, they have no reason to submit their own move at all. This issue may also arise in the context of state channel settlement. When such situations are an issue, \(1\) provide a way of circumventing non-participating participants, perhaps through a time limit, and \(2\) consider adding an additional economic incentive for participants to submit information in all of the situations in which they are supposed to do so.

Beware of negation of the most negative signed integer¶  
Solidity provides several types to work with signed integers. Like in most programming languages, in Solidity a signed integer with N bits can represent values from -2^\(N-1\) to 2^\(N-1\)-1. This means that there is no positive equivalent for the MIN\_INT. Negation is implemented as finding the two's complement of a number, so the negation of the most negative number will result in the same number.

This is true for all signed integer types in Solidity \(int8, int16, ..., int256\).

contract Negation {  
 function negate8\(int8 \_i\) public pure returns\(int8\) {  
 return -\_i;  
 }

 function negate16\(int16 \_i\) public pure returns\(int16\) {  
 return -\_i;  
 }

 int8 public a = negate8\(-128\); // -128  
 int16 public b = negate16\(-128\); // 128  
 int16 public c = negate16\(-32768\); // -32768  
}  
One way to handle this is to check the value of a variable before negation and throw if it's equal to the MIN\_INT. Another option is to make sure that the most negative number will never be achieved by using a type with a higher capacity \(e.g. int32 instead of int16\).

A similar issue with int types occurs when MIN\_INT is multiplied or divided by -1.

