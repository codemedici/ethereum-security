# Synthetix MKR Manipulation

In December 2019, Synthetix suffered another attack as a result of price oracle manipulation. What’s notable about this one is that it crossed the barrier between on-chain price data and off-chain price data.

Reddit user u/MusaTheRedGuard [observed](https://www.reddit.com/r/ethfinance/comments/eexbfa/daily_general_discussion_december_24_2019/fby3i6n/) that an attacker was making some very suspicious trades against sMKR and iMKR \(inverse MKR\). The attacker first purchased a long position on MKR by buying sMKR, then purchased large quantities of MKR from the Uniswap ETH/MKR pair. After waiting a while, the attacker sold their sMKR for iMKR and sold their MKR back to Uniswap. They then repeated this process.

Behind the scenes, the attacker’s trades through Uniswap allowed them to move the price of MKR on Synthetix at will. This was likely because **the off-chain price feed that Synthetix relied on was in fact relying on the on-chain price of MKR,** and there wasn’t enough liquidity for arbitrageurs to reset the market back to optimal conditions.

![](../../.gitbook/assets/image%20%289%29.png)

This incident illustrates the fact that even if you think you’re using off-chain price data, you may still actually be using on-chain price data and you may still be exposed to the intricacies involved with using that data.

