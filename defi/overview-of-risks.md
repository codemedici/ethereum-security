# Overview of Risks

This article introduces a model for assessing risk levels in various permissionless lending protocols. The first part of the article presents the DeFi risk categories with examples, and the second part introduces some risk remediations and prevention measures.

## DeFi Risks

Returns in DeFi are never risk free, particularly on lending platforms such as Compound; the high yields are a reflection of the significant risks taken on by the liquidity provider. These categories of risk include but are not limited to:

* Smart-contract Risks
  * Code Security
  * Code Openness
* Centralized Points of Failure
  * Protocol Administration and Governance
  * Oracles
* Financial Risk
  * Colateral - Exchange Rate and Liquidation
  * Liquidity - Bank Runs

DeFi is an immature area, being only about one year old and yet over the past few months we have seen its various components being exploited multiple times, a non comprehensive list of exploits is:

* Jun 2019:
  * Synthetix --- sETH $37m
* Feb 2020:
  * bZx --- $900k
* Mar 2020:
  * iEarn --- $280k
  * MakerDAO Black Thursday $9M
* Apr 2020:
  * LendfMe --- $25m USD stolen through a reentrancy attack vector; funds are re-issued after team’s negotiation with hacker.
  * imBTC --- Uniswap Pool $300k USD stolen through a reentrancy attack vector
  * Curve --- A stablecoin exchange platform, revealed that they found and solved a bug in the sUSD reserve contract.
  * PegNet --- A cross-chain DeFi platform, suffered a 51% attack when 4 miners in their network controlled 70% hashrate.
  * Hegic --- 28k USD of liquidity locked in expired options contract by a bug in contract, for which the team promised to compensate affected users with their own funds.
* Jun 2020:
  * Balancer --- $500k ETH
* Jun 2020:
  * Liquid network --- $16m BTC (avoided)

One likely reason for this recent explosion in the number of hacks is due to the rise of flashloans; most of these exploits would not have been possible without flashloans as they require large amounts of capital which attackers need in advance, for example, in order to manipulate price oracles of pump and dump the value of a cryptocurrency.

One of the largest attack vectors is through undercollateralized loans where hackers pump the values of some assets temporarily with the currency borrowed from a flashloan and deposit them to a lending protocol and then borrow a different asset using the manipulated price coming from a price oracle.

### Smart contract risks

With the expansion of the DeFi space and the enormous volume of liquidity being poured into applications thanks to yield farming, the incentive for bad actors to exploit smart contracts is ever-increasing.

#### Code Security

Smart Contract Risk is the main contributor to counterparty risk in DeFi. The Uniswap and Lendf.me incidents are recent examples of DeFi platforms that were hacked due to poor code security; the hacker(s) exploited a reentrancy vulnerability that arose from the incompatibility between the ERC-777 token standard and the DeFi protocols. Broadly speaking, the reentrancy vulnerability allowed the hacker to essentially re-spend initial deposits of imBTC, effectively providing them with unlimited capital to enact trades or borrows.

**Uniswap**

One recent example happened to the well-known DEX Uniswap, which was hacked via an attack vector already known to Uniswap and to the crypto community at large. Almost exactly a year before the Uniswap attack, ConsenSys Diligence - the security audit service offered by ConsenSys - [identified and published](https://medium.com/consensys-diligence/uniswap-audit-b90335ac007) the ERC-777 reentrancy attack vector.

The attack was made possible because Uniswap V1 does not have measures in place to guard against this type of reentrancy attack when interacting with the ERC-777 standard. In total, the hacker made away with \~$300k USD in imBTC and ETH.

**Lendf.me**

The Lendf.me incident exploited the same reentrancy vulnerability made available by the incomplete compatibility between the lending protocol and the ERC-777 token standard, but to a far more extensive degree of success. Nearly 100% of Lendf.me’s funds - over $24m USD - was drained during the attack on April 19.

#### Code Openness

While DeFi is often referred to as trustless, a user of a DeFi platform must trust the smart contract they are interacting with. A smart contract could be opaque to a user, which means a user is trusting the contract code in the same way a user trusts any web 2.0 infrastructure.

### Centralized Points of Failure

Centralization risk is an important risk to consider when lending money with DeFi protocols.

#### Protocol Administration and Governance

One of the biggest contributors to centralization risk in DeFi protocols is the use of admin keys. Admin keys allow protocol developers to change different parameters of their smart contract systems like oracles, interest rates and potentially more, these private keys controlling a smart contract's governance could get hacked or misused in ways such as draining a lending pool from its funds or freezing the smart contract rendering it useless.

**Compound**

For example, the lending platform Compound is a CUSTODIAL system, all lending pools can be trivially drained if their admin private key is compromised; during a security review in 2019 of the Compund smart contracts, [samczsun](https://samczsun.com/) found that Compound v2 has four different administrative positions which are set to three distinct addresses, if any of those keys would have been compromised, an attacker could have drained all of the cTokens from the Compound lending tool, among other attacks such as preventing minting and transfering of cTokens.

**Liquid Network**

Another recent example involves the Liquid network, which as pointed out [on Twitter](https://twitter.com/\_prestwich/status/1276300994989572096) by James Prestwich, for just under and hour, three administrative private keys controlled 870 Bitcoin due to a timelock issue; this violates liquid's security model.

#### Oracle risk

Another large element of centralization risk in these protocols is oracle centralization. Oracles provide data to smart contracts that is then used in the execution of functions. Price feeds are one of the critical pieces of infrastructure in decentralized finance and their failure or exploitation can lead to negative outcomes for users and platforms.

DeFi project, bZx, famously suffered from an oracle attack with the hackers stealing some 630,000 ETH. Chainlink and other decentralised oracle networks are helping to mitigate this risk.

**Synthetix Hack**

In this case, a Synthetix oracle, responsible for providing external data to Synthetix's smart contracts, transmitted false data on June 25th 2019, which a bot took advantage of. This particular bot was able to take advantage of the mispricing issue immediately, and exploit it repeatedly. The company contacted the owner of the arbitrage bot that unintentionally hacked the oracle and agreed on a bounty deal with him in order to return the funds.

**bZx second hack**

This is an example of an exploit relying on oracle manipulation, made possible by the large amount of ETH available to the attacker as she takes advantage of the bZx flashloan feature to borrow 7,500 ETH. See this article for an in depth explaination.

With the flashloan, the exploit swaps 900 ETH in two batches for sUSD through Kyber. The sell-off of these two batches effectively drives the price of sUSD to around 2.5x higher when compared to the average ETH/sUSD market price.

The attacker this time takes the approach of first collateralizing the collected sUSD back into bZx and then borrowing from it 6,796 ETH. As bZx relies on Kyber for the price feed, with the spiked sUSD/ETH price, the collection of sUSD allows for the borrow of 6796 ETH, which indicates that this loan is now underwater with insufficient collateralization.

With the borrowed 6,796 ETH (3,082 ETH leftover), the attacker is able to repay the 7,500 ETH flashloan back to bZx with the profit of 2,378 ETH.

### Financial Risk

DeFi contains many of the same risks as legacy finance. While most lending platforms use over-collateralization to reduce credit risk, over-collateralization does not completely remove credit risk. Crypto assets are notoriously volatile and these platforms have no method to recover from system insolvency caused by volatile collateral assets. The main sources of financial risk in DeFi are collateral and liquidity.

#### Colateral - Exchange Rate and Liquidation Risk

Without a widely accepted approach to on-chain reputation or identity, the only method to avoid unwanted amounts of credit risk in DeFi money market platforms is to use over-collateralization, however, the assets used for yield farming are often highly volatile. This volatility can lead to large capital losses over the period that someone wishes to farm yield. For instance, in the borrowing and lending platform, Compound, a user farming COMP may find that their position is liquidated as the value of their collateral falls below the required amount due to an unfavourable exchange rate movement.

#### Liquidity - Bank Runs Risk

The currently scoped platforms all attempt to incentive liquidity by using dynamic interest rate models which produce varying rates depending on the level of liquidity in each asset pool. However, incentivized liquidity does not mean guaranteed liquidity. A user takes on risk that they will not be able to withdraw their lent out assets on demand because all the assets are currently lent out.

For example, when you lend on o DeFi platform such as Compound, you are not guaranteed to be able to withdraw whenever you want. If you try to withdraw your funds and all the money is locked up in outstanding loans, your withdrawal transaction will fail; each loan shrinks the pool of liquidity. For example, 95% of the DAI deposited by lenders on Compound in 2019 was being loaned out to borrowers. Only 5% of the DAI was available for withdrawals, so only a small fraction of the lenders would be able to recover their DAI if they wanted to.

A common understanding is that suppliers are free to withdraw their principal and interest at any given point — i.e. Compound will remain liquid enough for suppliers to access their funds whenever they wish. However, both borrowers and lenders are limited by the size of the liquidity pools.

A bank run happens when suppliers, suddenly anxious about a market’s stability, attempt to rapidly and simultaneously withdraw more funds than are available in the platform, causing further panic and distrust of the system.

**Black Thursday**

A recent episode of a bank run that affected the collateralised loans market, was a problem for the DAI stable coin when it faced the collapse of Ether exchange rate by more than 55% in a single day, this episode became known as black Thursday, and was partly in response to the COVID-19 pandemic and the concurrent crash of the stock market as well. DAI depends on overcollateralization, where you must maintain 150% of collateral in order to back your DAI as an absolute minimum, htus even contracts that were in theory 300% collateralised ended up undercollateralised because of the fall in value of the collateral.

In normal circumstances the response would be that through automated systems as well as manual intervention, users would add DAI back into the system in order to re-collateralize their loans, or put ETH as additional collateral in order to refund or re-collateralize their loans. However, during the time when the value of ETH dropped 55%, the gas price increased and it became difficult to get transactions accepted. The situation was compounded by a bug in the auction system that allowed some users to buy some of these loans for close to zero and liquidate them. In the end the damage was only about 5 million dollars, and most accounts affected were reimbursed.

Although this was a black swan event, it shows how a cascade failure of multiple problems all occurring at the same time can happen and therefore is a risk that should be taken into account.

## DeFi Risk Mitigation

The first step to establishing DeFi as a safe haven versus centralized financial systems is to make sure that DeFi systems are secure, resilient and robust. That means they must be ready to withstand attacks and hackers, but also external market pressures and high-volume activity spikes that might occur during times of stress.

[DeFi Score](https://defiscore.io/), a model developed by ConsenSys for assessing risk levels in various permissionless lending protocols, assesses code security by looking at three pieces of off-chain but public data:

1. Smart Contract Risk
   * Time on Mainnet: Normalized time since the protocol first launched on mainnet
   * No Critical Vulnerabilities: No vulnerabilities have been exploited
   * Four Engineer Weeks 4 or more engineer weeks have been dedicated to auditing the protocol
   * Public Audit: Has the audit report been made public
   * Recent Audit: Has there been an audit in the last 12 months OR have no code changes been made
   * Bounty Program: Does the development team offers a public bug bounty program?
2. Financial Risk
   * Collateral Makeup CVaR
   * Utilization Ratio
   * Absolute Liquidity
3. Centralization Risk
   * Protocol Administration
   * Oracles

Users can easily make a security assessment of the risk involved in staking their tokens in a liquidity pool by comparing scores on a 1-10 scale, scores are given to each currency for every lending pool they are listed on:

The following screenshot shows an overview of the Compound protocol overview as shown on the [DefiScore website](https://defiscore.io/), showing a final weighted score of 8.6 out of 10.

### Mitigating Smart Contract Risks

#### Code Audits

While no smart contract can be guaranteed as safe and free of bugs, a thorough code audit and formal verification process from a reputable security firm helps uncover critical, high severity bugs that otherwise could result in financial harm to users.

#### Bug Bounties

Bug bounty programs are another positive indicator that the development team takes security seriously by incentivizing independent security researchers to discover protocol bugs, ultimately allowing for a more widespread security review.

#### Best Practices

Creating security and risk management solutions that ensure that teams are following best practices and identifying and resolving their critical risks

**TODO**

[https://github.com/crytic/building-secure-contracts](https://github.com/crytic/building-secure-contracts)

### Mitigating Illiquidity & Bank Run Risk

#### Live security monitoring

With proper monitoring tools, users can better protect themselves from being the unwilling recipients of automated liquidations on lending platforms. Users can track the collateral ratio and decide to repay the borrows or add deposits to keep the vault in a safe condition and out of range of liquidation.

A successful alerts and monitoring product:

* makes sense of data from the blockchain and decentralized applications
* repeatedly informs and promotes decision/action
* measures the effects of decisions/actions and integrates those effects back into the data

#### Increased transparency

Improvement in the understanding and execution of security testing to identify known vulnerabilities, and the use of bug bounties and white hat hackers to identify undiscovered system issues before they cause problems

#### Insurance

Investigating insurance and other risk hedging options which might provide financial coverage in the case of system or user losses incurred from any scenario.

Blockchain-based insurance has been around for a while, but has been brought sharply into focus these past few months. Nexus Mutual - an blockchain insurance veteran who acted as the first respondent for victims in bZx exploit - and more recently Opyn have (re)emerged as top players in this adjacent DeFi industry, serving as hedge options against the protected assets.

### Mitigating Centralization Risks

Measures like timelocks and multi-signature wallets help mitigate the risk of financial loss due to centralized elements. Mult-signature wallets help mitigate this risk by distributing control to a larger numb.er of developers, meaning that the loss or compromise of a single private key cannot compromise the entire system. Timelocks help mitigate risk by allowing protocol users to exit their positions before a change can take place.

### Conclusion

security audits, focusing on live security monitoring, increased transparency, and insurance will be the way forward to increasing DeFi’s potential.

As DeFi protocols grow in number, complexity, and interconnectedness, more security vulnerabilities and compromises are likely to occur. Though regrettable, these incidents are crucial to the secure development of any emerging technology. The more we can use the services and tools available to us to identify and protect against these attack vectors, the more confidently people will interact with the emerging open financial ecosystem.
