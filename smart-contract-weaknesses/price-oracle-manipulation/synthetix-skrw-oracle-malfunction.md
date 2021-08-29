# Synthetix sKRW Oracle Malfunction

Synthetix is a derivatives platform which allows users to be exposed to assets such as other currencies. To facilitate this, **Synthetix \(at the time\) relied on a custom off-chain price feed implementation wherein an aggregate price calculated from a secret set of price feeds was posted on-chain at a fixed interval.** These prices then allowed users to take long or short positions against supported assets.

On June 25, 2019, one of the price feeds that Synthetix relied on mis-reported the price of the Korean Won to be 1000x higher than the true rate. Due to [additional errors](https://blog.synthetix.io/response-to-oracle-incident/) elsewhere in the price oracle system, this price was accepted by the system and posted on-chain, where **a trading bot quickly traded in and out of the sKRW market.** In total, the bot was able to earn a profit of over 1B USD, although the Synthetix team was able to negotiate with the trader to return the funds in exchange for a bug bounty.

Synthetix correctly implemented the oracle contract and pulled prices from multiple sources in order to prevent traders from predicting price changes before they were published on-chain. However, an isolated case of one upstream price feed malfunctioning resulted in a devastating attack. This illustrates the risk of using a price oracle which uses off-chain data: you don't know how the price is calculated, so your system must be carefully designed such that all potential failure modes are handled properly.

