# Design Flaws

## Design Flaws

This is a category of bugs that includes unforseen consequences from the logic of a smart contract's poor design choices, i.e. vulnerable by design. These are often critical vulnerabilities that developers tend to dismiss, because they cannot be fixed, or fixes would require redesigning evertything from scratch.  
examples include poor incentives structure, unfair advantages, amplification attacks...

## The DAO examples

The following are examples from [A Call for a Temporary Moratorium on The DAO](http://hackingdistributed.com/2016/05/27/dao-call-for-moratorium/)

### The Affirmative Bias, and the Disincentive to Vote No

The current DAO has a strong positive bias to vote YES on proposals and to suppress NO votes as a side-effect of the way in which it restricts users’ range of options following the casting of a vote. Specifically, the current DAO restricts the ability of a token holder to split from the DAO once they have voted on a proposal until the outcome of the vote is determined. Thus, a voter who believes a proposal has negative expected value is in a quandary: they can split from The DAO immediately without taking any risk, or else they can vote NO and hope that the proposal fails to be funded. A NO vote is therefore inherently risky for an investor who perceives the proposal to be -EV, in a way that voting YES is not for a +EV voter. As a consequence, The DAO voting is likely to exhibit a bias: YES votes will arrive throughout the voting period, while a strategic token holder will want to cast their NO vote only when they have some assurance of success. Because strategic NO voters will cast their votes only after gaining information on others’ negative perception of the same proposal, the voting process itself will not yield uniform information about the token holders’ preferences over time. Preferences of the positive voters will be visible early on, but the negative sentiment will be suppressed during the voting process -- a problematic outcome for a crowd-funding organization based on measuring the sentiment of the crowd through votes.

### The Stalking Attack

Splitting from The DAO \(the only viable method of extracting one’s Ether holdings from the main DAO contract\) is currently open to a “stalking attack.” Recall that a user who splits from The DAO initiates a new DAO contract in which they are the sole investor and curator. The intent is that a user can extract his funds by whitelisting a proposal to pay himself the entire contents of this sub-contract, voting on it with 100% support, and then extracting the funds by executing the approved proposal. However, recall that the split and the resulting sub-contract creation takes place on a public blockchain. Consequently, an attacker can pursue a targeted individual into such sub-contracts. Since a splitting user is the new curator of the nascent sub-contract, a stalker cannot actually steal funds; the stalkee can refuse to whitelist proposals by the stalker \(though note that, due to potential for confusion and human error, the expected outcome from such attacks is still positive\). If the stalker commits funds that correspond to 53% or more of the sub-contract, he can effectively block the stalkee from withdrawing their funds out of the contract back into ether. Subsequent attempts by the victim to split from the sub-contract \(to create a sub-sub-contract\) can be followed recursively, effectively trapping the victim’s funds and prohibiting conversion back to ether. The attacker places no funds at risk, because she can split from the child-DAO at any time before the depth limit is reached. This creates the possibility for ransom and blackmail. While some remedies have been suggested for preventing and counterattacking during a stalker attack, they require unusual technical sophistication and diligence on behalf of the token holders.

### The Ambush Attack

In an ambush, a large investor takes advantage of the bias for DAO users to avoid voting NO by adding a large percent of YES votes at the last minute to fund a self-serving proposal. Recall that under the current DAO structure, a rational actor who believes a proposal is -EV is likely to refrain from voting, since doing so would restrict his ability to split his funds in the case that the proposal succeeds. This is especially true when the investor observes that sufficiently many NO votes already exist to reject the proposal. Consequently, even proposals that provide absurdly low returns to The DAO may garner NO votes that are barely sufficient to defeat them.  
This kind of behavior opens the door to potential attack: A sufficiently large voting bloc can take advantage of this reticence by voting YES at the last possible moment to fund the proposal. Such attacks are very difficult to detect and defend against because they leave little to no time for The DAO token holders to withdraw their funds. Among the current DAO investors, there is already a whale who invested 888,888 Ether. This investor currently commands 7.7% of all outstanding votes in The DAO. For a proposal that requires only a 20% quorum, this investor already has 77% of the required YES votes to pass the proposal, and just needs to conspire with 2.3%+1 of the token holders, in return for paying the conspirators out from the stolen funds.

### The Token Raid

In a token raid, a large investor stands to benefit by driving TDTs lower in value, either to profit from such price motion directly \(e.g. via shorts or put options\), or to purchase TDTs back in the open market in order to acquire a larger share of The DAO. A token raid is most successful if the attacker can \(i\) incentivize a large portion of token holders not to split, but instead sell their TDT directly on exchanges, and \(ii\) incentivize a large portion of the public not to purchase TDT on exchanges. An attacker can achieve \(i\) by implementing the stalker attack on anyone who splits and then making that attack public on social media. Worse, since the existence of the stalker attack is now well-known, the attacker need not attack any real entity, but can instead create fictitious entities who post stories of being stalked in order to sow panic among The DAO investors.  
An attacker can achieve \(ii\) by creating a self-serving proposal widely understood to be -EV, waiting for the 6th day before voting ends, and then voting YES on it with a large block of votes. This action has the effect of discouraging rational market actors from buying TDT tokens because \(a\) if the attacker's proposal succeeds they will lose their money, and \(b\) they don’t have enough time to buy TDTs on an exchange and convert them back into Ether before the attacker's proposal ends, thus eliminating any chance of risk-free arbitrage profits. The combined result of \(i\) and \(ii\) means that there will be net selling pressure on TDT, leading to lower prices. The attacker can then buy up cheap TDT on exchanges for a risk free profit, because she is the only TDT buyer who has no risk if the attacking proposal actually manages to pass.

### The extraBalance Attack

The extraBalance Attack is one in which an attacker tries to scare token holders into splitting from The DAO so that book value of TDT increases. The book value of TDT increases through splits because token holders who split can not recover any extraBalance, so the extraBalance becomes a larger percentage of the total balance, thus increasing the book value of the TDT. Currently the extraBalance is 203,257.65 Ether, which means the book value of TDT should be 1.02. If the Attacker can scare away half the token holders, the TDT will increase in value to 1.04. If the Attacker can scare away ~95% of the token holders, the book value of the remaining TDT will be roughly 2.00. In this attack, the attacking whale would do the opposite of the token raid by creating a self-serving proposal with a negative return and then immediately voting YES on it with a large voting block of TDT, thus scaring all the token holders, and then giving them 14 days until the end of the voting period so that they have more than enough time to safely split. In this scenario, splitting will be risk free \(assuming that it is not coupled with a stalking attack\), since voting NO could result in losses if the attackers end up having enough YES votes.

### The Split Majority Takeover Attack

Even though the DAO white paper specifically identifies the majority takeover attack and introduces the concept of curators to deter it, it is not clear that the deterrence mechanism is sufficient. Recall that in the majority takeover attack outlined in the DAO whitepaper, a large voting bloc, of size 53% or more, votes to award 100% of the funds to a proposal that benefits solely that bloc. Curators are expected to detect such instances by tracking identities of the beneficiaries. Yet it is not clear how a curator can detect such an attack if the voting bloc, made up of a cartel of multiple entities, proposes not just a single proposal for 100% of the funds, but multiple different proposals. The constituents of the voting bloc can achieve their goal of emptying out the fund piecemeal. Fundamentally, this attack is indistinguishable “on the wire” from a number of investment opportunities that seem appealing to a majority. The key distinguishing factor here is the conflict of interest: the direct beneficiaries of the proposals are also token holders of The DAO.

### The Concurrent Tie-Down Attack

The structure of The DAO can create undesirable dynamics in the presence of concurrent proposals. In particular, recall that a TDT holder who votes YES on a proposal is blocked from splitting or transferring until the end of the voting period on that proposal. This provides an attack amplification vector, where an attacker collects votes on a proposal with a long voting period, in effect trapping the voters' shares in The DAO. She can then issue an attacking proposal with a much shorter voting period. The attack, if successful, is guaranteed to impact the funds from the voters who were trapped. Trapped voters are forced to take active measures to defend their investments.

### Independence Assumption

A critical implicit assumption in the discussion so far was that the proposals are independent. That is, their chances of success, and their returns, are not interlinked or dependent on each other. It is quite possible for simultaneous proposals to The DAO to be synergistic, or even antagonistic; for instance, a cluster of competing projects in the same space may affect each others’ chances of success and thus, collective returns. Similarly, cooperating projects, if funded together, might create sufficient excitement to yield excess returns; evidence from social science indicates that social processes are driven by non-linear systems.  
Yet the nature of voting on proposals in The DAO provide no way for investors to express complex, dependent preferences. For instance, an investor cannot indicate a conditional preference \(e.g. “vote YES on this proposal if this other proposal is not funded or also funded”\). In general, the construction of market mechanisms to elicit such preferences, and appropriate programmatic APIs for expressing them, requires a more detailed and nuanced contract. This does not constitute an attack vector, but it does indicate that we might see strategic voting behavior even in the absence of any ill will by participants.

## Uninitialized contract

### Parity wallet, 2nd hack

It was a “library”: code that had been abstracted out of the original wallet contract in an attempt to save gas \(ethereum’s unit of “fuel” used to pay for computations\). Their intention was to re-use a single deployment of the repetitive code shared by each instance of the wallet. Storage is one of the most expensive resources in an Ethereum network, and redeploying the same code repeatedly can be considered wasteful in this regard. This valid engineering decision, however, became a fatal flaw in conjunction with some critical oversights.

The library had been based on one of Parity’s earlier wallet implementations. As such, it contained code which enabled the library to be turned into an actual wallet. Code meant for the initialization of wallets was not removed in the process of writing the wallet library. It is believed that devops199—a self-professed non-programmer and newcomer to smart contracts—was trying to understand and perhaps exploit an earlier Parity wallet hack when they made themselves an owner of the library by calling this initialization function. Devops199 then called the library’s kill function \(another function which should have been removed in the development of the library\), likely misunderstanding the implications.

The 587 deployed wallets dependant on this WalletLibrary contract became unusable. All meaningful functionality of the wallet stubs called code in the library, and the immutability of ethereum smart contracts meant there was no way to repair the damage. With no code at the address being delegated to, every function call fails and reverts. 513,774 Ether and many more ERC20 tokens were locked, an estimated $160 million dollars of assets at the time of the incident.

// kills the contract sending everything to \`\_to\`.  
function kill\(address \_to\) onlymanyowners\(sha3\(msg.data\)\) external {  
 suicide\(\_to\);  
}  
The “suicide” in the above function refers to a solidity function which “deletes” the contract from the the blockchain, and forwards its entire ether balance to the passed address. The keyword has since been renamed to the more appropriate “selfdestruct”.

Ironically, the vulnerability devops199 exploited to gain ownership of the library had been reported by Github user “3esmit” \(Ricardo Guilherme Schmidt\) three months prior. It appears the Parity team misunderstood the issue at hand, pushed an ineffectual fix, and closed the issue.
