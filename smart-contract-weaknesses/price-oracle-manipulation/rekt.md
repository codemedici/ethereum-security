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

