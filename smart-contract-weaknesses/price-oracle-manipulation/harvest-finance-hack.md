# Harvest Finance Hack

On October 26, 2020, an unknown user hacked the Harvest Finance pools using a technique that you can probably guess by now. You can read the official post-mortem [here](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217), but once again I’ll summarize it for you: **the attacker deflated the price of USDC in the Curve pool by performing a trade, entered the Harvest pool at the reduced price, restored the price by reversing the earlier trade, and exited the Harvest pool at a higher price. This resulted in over 33MM USD of losses**

