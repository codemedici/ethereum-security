# King of Ether

## King of Ether

## King of Ether Throne - Denial of Service

[https://github.com/PacktPublishing/Securing-Ethereum-Smart-Contracts/blob/master/chapter%205/KingOfTheEtherThrone.sol](https://github.com/PacktPublishing/Securing-Ethereum-Smart-Contracts/blob/master/chapter%205/KingOfTheEtherThrone.sol)  
[https://www.kingoftheether.com/postmortem.html](https://www.kingoftheether.com/postmortem.html)

The King of Ether Throne smart contract was a basic game, in which one wishing to be the king would have to pay an amount greater than the one before. The old king is dethroned, but makes a profit. The original source of the contract is available here.

Below is a simplified version of the contract with updated syntax.

contract KoET {  
 // Highest bidder becomes the Leader.

 address payable public currentLeader;  
 uint public highestBid;

 function \(\) external payable {  
 require\(msg.value &gt; highestBid\);  
 currentLeader.transfer\(highestBid\); // Refund the old leader  
 currentLeader = msg.sender;  
 highestBid = msg.value;  
 }  
}  
If you are not familiar with the case, take a minute and try to imagine what could go wrong.

In February 2016, “Sir Wobblle” became the king, and was never dethroned, because the wallet he used consumed more than the 2,300 gas available. In case it isn’t clear, if Sir Wobblle cannot be dethroned because he cannot accept the payment, then a successor cannot be crowned. The contract is effectively disabled.

The developer didn't account for the fact that the .transfer\(\) function can revert if sent to a contract address:

either because the gas runs out, .transfer\(\) and .send\(\) will forward a very limited amount of gas  
or by a smart contract that reverts its fallback function, this could be due to lack of a payable fallback function, or an explicit revert such as a conditional.  
Lessons learned:  
check all external call return values  
Avoid interacting with more than one "untrusted" party during a transaction, including msg.sender.

In this case, currentLeader would be a second "untrusted" party. The bug wouldn't have happened if the best practice of implementing "pull payments" was followed. If you come across a contract that implements this, you know you have to spend some time scrutinizing it, looking at all possible failure states and their consequences. Always ensure that a malicious contract cannot interfere, manipulate information or simply cause denial of service. If you came from the B9 Solidified dev course, you will remember the recommendation to never interact with more than one untrusted party at a time. Since msg.sender is untrusted, all other interactions would be better off left for other interactions. A pull payment would've worked in this case. If you bump into a contract that interacts with two or more untrusted parties within the same transaction, you will have to assess what happens if any of the parties simply revert the transaction, what are the failure states: does it keep going, how is the failure accounted for, can someone bring the contract to a complete halt?

If needed, head back to module three, Solidity Best Practices, and review how .send\(\) and .transfer\(\) behave. Keep in mind that although the .transfer\(\) reverts, if .send\(\) was used, the contract would still malfunction, changing the king without paying the old one.

Also, remember this is one of the reasons for using pull payments.

Any external call should have it's result duly checked by the caller. Take extra care if the contract being called is untrusted \(a token address provided by the user, for example\), the caller must be prepared and have mechanisms that prevent damage from rogue contracts.

Now, go ahead, deploy this contract and make yourself the eternal king.

For more information about King of Ether Throne, take a look at the postmortem of the event.

