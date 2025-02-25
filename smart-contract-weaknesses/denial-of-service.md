---
description: >-
  including gas limit reached, unexpected throw, unexpected kill, access control
  breached
---

# Denial of Service

Denial of service is deadly in the world of Ethereum: while other types of applications can eventually recover, smart contracts can be taken offline forever by just one of these attacks. Many ways lead to denials of service, including maliciously behaving when being the recipient of a transaction, artificially increasing the gas necessary to compute a function, abusing access controls to access private components of smart contracts, taking advantage of mixups and negligence, etc. This class of attack includes many different variants and will probably see a lot of development in the years to come.

## Example

1\. An auction contract allows its users to bid on different assets.\
2\. To bid, a user must call a bid(uint object) function with the desired amount of ether. The auction contract will store the ether in escrow until the object's owner accepts the bid or the initial bidder cancels it. This means that the auction contract must hold the full value of any unresolved bid in its balance.\
3\. The auction contract also contains a withdraw(uint amount) function which allows admins to retrieve funds from the contract. As the function sends the amount to a hardcoded address, the developers have decided to make the function public.\
4\. An attacker sees a potential attack and calls the function, directing all the contract's funds to its admins. This destroys the promise of escrow and blocks all the pending bids.\
5\. While the admins might return the escrowed money to the contract, the attacker can continue the attack by simply withdrawing the funds again.

### Code Example

In the following example (inspired by King of the Ether) a function of a game contract allows you to become the president if you publicly bribe the previous one. Unfortunately, if the previous president is a smart contract and causes reversion on payment, the transfer of power will fail and the malicious smart contract will remain president forever. Sounds like a dictatorship to me:

function becomePresident() payable {\
&#x20;   require(msg.value >= price); // must pay the price to become president\
&#x20;   president.transfer(price);   // we pay the previous president\
&#x20;   president = msg.sender;      // we crown the new president\
&#x20;   price = price \* 2;           // we double the price to become president\
}

In this second example, a caller can decide who the next function call will reward. Because of the expensive instructions in the for loop, an attacker can introduce a number too large to iterate on (due to gas block limitations in Ethereum) which will effectively block the function from functioning.

function selectNextWinners(uint256 \_largestWinner) {\
&#x20;for(uint256 i = 0; i < largestWinner, i++) {\
&#x20;// heavy code\
&#x20;}\
&#x20;largestWinner = \_largestWinner;\
}\
Additional Resources:\
[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#denial-of-service-dos](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#denial-of-service-dos)\
[https://dasp.co/#item-5](https://dasp.co/#item-5)\
Parity Multisig Hacked. Again\
Statement on the Parity multi-sig wallet vulnerability and the Cappasity token crowdsale

## DoS with Block Gas Limit

Many contracts rely on calls happening within certain period of time, but Ethereum can be spammed with very high Gwei transactions for a decent amount of time relatively cheaply.

For example, FOMO3D (a countdown game where the last investor wins the jackpot, but each investment adds time to the countdown) was won by a user who completely clogged the blockchain for a small period of time, disallowing others from investing until the timer ran out and he won (see [DoS with Block Gas Limit](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/dos-gas-limit.md)).

There are many "croupier" gambling contracts nowadays that rely on past blockhashes to provide RNG. This is not a terrible source of RNG for the most part, and they even account for the pruning of hashes that happens after 256 blocks, but at that point many of them simply null the bet. This would allow someone to make bets on many of these similarly-functioning contracts with a certain result as the winner for them all, check the croupier's submission while it's still pending, and, if it's unfavorable, simply clog the blockchain until pruning occurs and you can get your bets returned.

### Example: Governmental - Gas limit

The Governmental was a game, deployed in April 2016, in which players could call a payout function that paid taxes to previous players Yes, it was a kind of a Ponzi scheme, with government-inspired dynamics.

The original contract can be inspected here. Below are the lines that caused the problem:\
[https://etherscan.io/address/0xf45717552f12ef7cb65e95476f217ea008167ae3#code](https://etherscan.io/address/0xf45717552f12ef7cb65e95476f217ea008167ae3#code)

`creditorAddresses = new address[](0);`\
`creditorAmounts = new uint[](0);`

The code above clears two arrays of previous players. These operations' costs are unbounded, directly proportional to the number of users. In case of Governmental, the payout function ceased working when a large enough number of users signed up.

The best practice in this case would be to avoid unbounded loops as much as possible. If they are needed, account for the block gas limit with a method that ensures processing will be successful in any case.

Later on, someone was able to drain the contract. It’s unclear if that was due to an increased gas limit or fewer players after the incident.

This same contract was later found to be vulnerable to the deep callstack attack whereby one could manipulate the callstack by performing useless operations before calling the target contract, with the goal of halting the function at a determined point, allowing for manipulation of state. This attack is no longer applicable since implementation of EIP 150.\
[https://ethereum.stackexchange.com/questions/9398/how-does-eip-150-change-the-call-depth-attack](https://ethereum.stackexchange.com/questions/9398/how-does-eip-150-change-the-call-depth-attack)

Reddit thread about the vulnerability.

## DoS with (Unexpected) revert

DoS (Denial of Service) attacks can occur in functions when you try to send funds to a user and the functionality relies on that fund transfer being successful.

This can be problematic in the case that the funds are sent to a smart contract created by a bad actor, since they can simply create a fallback function that reverts all payments.

For example:

```
// INSECURE
contract Auction {
    address currentLeader;
    uint highestBid;

    function bid() payable {
        require(msg.value > highestBid);

        require(currentLeader.send(highestBid)); // Refund the old leader, if it fails then revert

        currentLeader = msg.sender;
        highestBid = msg.value;
    }
}
```

As you can see in this example, if an attacker bids from a smart contract with a fallback function reverting all payments, they can never be refunded, and thus no one can ever make a higher bid.

This can also be problematic without an attacker present. For example, you may want to pay an array of users by iterating through the array, and of course you would want to make sure each user is properly paid. The problem here is that if one payment fails, the funtion is reverted and no one is paid.

```
address[] private refundAddresses;
mapping (address => uint) public refunds;

// bad
function refundAll() public {
    for(uint x; x < refundAddresses.length; x++) { // arbitrary length iteration based on how many addresses participated
        require(refundAddresses[x].send(refunds[refundAddresses[x]])) // doubly bad, now a single failure on send will hold up all funds
    }
}
```

An effective solution to this problem would be to use a pull payment system over the current push payment system. To do this, separate each payment into it's own transaction, and have the recipient call the function.

```
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() payable external {
        require(msg.value >= highestBid);

        if (highestBidder != address(0)) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        (bool success, ) = msg.sender.call.value(refund)("");
        require(success);
    }
}
```

Examples from: [https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert) [https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls](https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls)

## King of Ether

## King of Ether Throne - Denial of Service

[https://github.com/PacktPublishing/Securing-Ethereum-Smart-Contracts/blob/master/chapter%205/KingOfTheEtherThrone.sol](https://github.com/PacktPublishing/Securing-Ethereum-Smart-Contracts/blob/master/chapter%205/KingOfTheEtherThrone.sol)\
[https://www.kingoftheether.com/postmortem.html](https://www.kingoftheether.com/postmortem.html)

The King of Ether Throne smart contract was a basic game, in which one wishing to be the king would have to pay an amount greater than the one before. The old king is dethroned, but makes a profit. The original source of the contract is available here.

Below is a simplified version of the contract with updated syntax.

contract KoET {\
&#x20;// Highest bidder becomes the Leader.

&#x20;address payable public currentLeader;\
&#x20;uint public highestBid;

&#x20;function () external payable {\
&#x20;require(msg.value > highestBid);\
&#x20;currentLeader.transfer(highestBid); // Refund the old leader\
&#x20;currentLeader = msg.sender;\
&#x20;highestBid = msg.value;\
&#x20;}\
}\
If you are not familiar with the case, take a minute and try to imagine what could go wrong.

In February 2016, “Sir Wobblle” became the king, and was never dethroned, because the wallet he used consumed more than the 2,300 gas available. In case it isn’t clear, if Sir Wobblle cannot be dethroned because he cannot accept the payment, then a successor cannot be crowned. The contract is effectively disabled.

The developer didn't account for the fact that the .transfer() function can revert if sent to a contract address:

either because the gas runs out, .transfer() and .send() will forward a very limited amount of gas\
or by a smart contract that reverts its fallback function, this could be due to lack of a payable fallback function, or an explicit revert such as a conditional.\
Lessons learned:\
check all external call return values\
Avoid interacting with more than one "untrusted" party during a transaction, including msg.sender.

In this case, currentLeader would be a second "untrusted" party. The bug wouldn't have happened if the best practice of implementing "pull payments" was followed. If you come across a contract that implements this, you know you have to spend some time scrutinizing it, looking at all possible failure states and their consequences. Always ensure that a malicious contract cannot interfere, manipulate information or simply cause denial of service. If you came from the B9 Solidified dev course, you will remember the recommendation to never interact with more than one untrusted party at a time. Since msg.sender is untrusted, all other interactions would be better off left for other interactions. A pull payment would've worked in this case. If you bump into a contract that interacts with two or more untrusted parties within the same transaction, you will have to assess what happens if any of the parties simply revert the transaction, what are the failure states: does it keep going, how is the failure accounted for, can someone bring the contract to a complete halt?

If needed, head back to module three, Solidity Best Practices, and review how .send() and .transfer() behave. Keep in mind that although the .transfer() reverts, if .send() was used, the contract would still malfunction, changing the king without paying the old one.

Also, remember this is one of the reasons for using pull payments.

Any external call should have it's result duly checked by the caller. Take extra care if the contract being called is untrusted (a token address provided by the user, for example), the caller must be prepared and have mechanisms that prevent damage from rogue contracts.

Now, go ahead, deploy this contract and make yourself the eternal king.

For more information about King of Ether Throne, take a look at the postmortem of the event.

## Resources

* [https://blog.sigmaprime.io/solidity-security.html#dos](https://blog.sigmaprime.io/solidity-security.html#dos)
* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/dos-revert.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/dos-revert.md)
* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/dos-gas-limit.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/dos-gas-limit.md)
* [https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-block-gas-limit](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-block-gas-limit)
* [https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls](https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls)
