# Rekt

## Synthetix sKRW Oracle Malfunction

Synthetix is a derivatives platform which allows users to be exposed to assets such as other currencies. To facilitate this, **Synthetix \(at the time\) relied on a custom off-chain price feed implementation wherein an aggregate price calculated from a secret set of price feeds was posted on-chain at a fixed interval.** These prices then allowed users to take long or short positions against supported assets.

On June 25, 2019, one of the price feeds that Synthetix relied on mis-reported the price of the Korean Won to be 1000x higher than the true rate. Due to [additional errors](https://blog.synthetix.io/response-to-oracle-incident/) elsewhere in the price oracle system, this price was accepted by the system and posted on-chain, where **a trading bot quickly traded in and out of the sKRW market.** In total, the bot was able to earn a profit of over 1B USD, although the Synthetix team was able to negotiate with the trader to return the funds in exchange for a bug bounty.

Synthetix correctly implemented the oracle contract and pulled prices from multiple sources in order to prevent traders from predicting price changes before they were published on-chain. However, an isolated case of one upstream price feed malfunctioning resulted in a devastating attack. This illustrates the risk of using a price oracle which uses off-chain data: you don't know how the price is calculated, so your system must be carefully designed such that all potential failure modes are handled properly.

##  Synthetix MKR Manipulation

In December 2019, Synthetix suffered another attack as a result of price oracle manipulation. What’s notable about this one is that it crossed the barrier between on-chain price data and off-chain price data.

Reddit user u/MusaTheRedGuard [observed](https://www.reddit.com/r/ethfinance/comments/eexbfa/daily_general_discussion_december_24_2019/fby3i6n/) that an attacker was making some very suspicious trades against sMKR and iMKR \(inverse MKR\). The attacker first purchased a long position on MKR by buying sMKR, then purchased large quantities of MKR from the Uniswap ETH/MKR pair. After waiting a while, the attacker sold their sMKR for iMKR and sold their MKR back to Uniswap. They then repeated this process.

Behind the scenes, the attacker’s trades through Uniswap allowed them to move the price of MKR on Synthetix at will. This was likely because **the off-chain price feed that Synthetix relied on was in fact relying on the on-chain price of MKR,** and there wasn’t enough liquidity for arbitrageurs to reset the market back to optimal conditions.

![](../../.gitbook/assets/image%20%289%29.png)

This incident illustrates the fact that even if you think you’re using off-chain price data, you may still actually be using on-chain price data and you may still be exposed to the intricacies involved with using that data.

## bZx Second Hack

In February 2020, bZx was hacked twice over the span of several days for approximately 1MM USD. You can find an excellent technical analysis of both hacks written by palkeo [here](https://www.palkeo.com/en/projets/ethereum/bzx.html), but we will only be looking at the second hack.

In the second hack, the attacker first purchased nearly all of the sUSD on Kyber using ETH. Then, the attacker purchased a second batch of sUSD from Synthetix itself and deposited it on bZx. Using the sUSD as collateral, the attacker borrowed the maximum amount of ETH they were allowed to. They then sold back the sUSD to Kyber.

If you’ve been paying attention, you’ll recognize this as essentially the same undercollateralized loan attack, but using a different collateral and a different decentralized exchange.

## yVault Bug

On July 25, 2020, I reported a bug to yEarn regarding the launch of their new yVault contracts. You can read the official writeup about this bug [here](https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/), but I will briefly summarize it below.

The yVault system allows users to deposit a token and earn yield on it without needing to manage it themselves. Internally, the vault tracks the total amount of yVault tokens minted as well as the total amount of underlying tokens deposited. The worth of a single yVault token is given by the ratio of tokens minted to tokens deposited. Any yield the vault earns is spread across all minted yVault tokens \(and therefore, across all yVault token holders\).

The first yVault allowed users to earn yield on USDC by supplying liquidity to the Balancer MUSD/USDC pool. When a user supplies liquidity to Balancer pools, they receive BPT in return which can be redeemed for a proportion of the pool. As such, the yVault calculated the value of its holdings based on the amount of MUSD/USDC which could be redeemed with its BPT.

This seems like the correct implementation, but unfortunately the same principle as given before applies - the state of the Balancer pool during a transaction is not stable and cannot be trusted. In this case, because of the bonding curve that Balancer chose, a user who swaps between from USDC to MUSD will not receive a 1:1 exchange rate, but will in fact leave behind some MUSD in the pool. This means that the value of BPT can be temporarily inflated, which allows an attacker to manipulate the price at will and subsequently drain the vault.

![](https://samczsun.com/content/images/2020/11/image-30.png)

This incident shows that price oracles are not always conveniently labelled as such, and that developers need to be vigilant about what sort of data they’re ingesting and consider whether that data can be easily manipulated by an unprivileged user.

## Harvest Finance Hack

On October 26, 2020, an unknown user hacked the Harvest Finance pools using a technique that you can probably guess by now. You can read the official post-mortem [here](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217), but once again I’ll summarize it for you: the attacker deflated the price of USDC in the Curve pool by performing a trade, entered the Harvest pool at the reduced price, restored the price by reversing the earlier trade, and exited the Harvest pool at a higher price. This resulted in over 33MM USD of losses

