# Harvest Finance Hack

On October 26, 2020, an unknown user hacked the Harvest Finance pools using a technique that you can probably guess by now. You can read the official post-mortem [here](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217), but once again I’ll summarize it for you: **the attacker deflated the price of USDC in the Curve pool by performing a trade, entered the Harvest pool at the reduced price, restored the price by reversing the earlier trade, and exited the Harvest pool at a higher price. This resulted in over 33MM USD of losses**

## **What happened** <a id="563a"></a>

On October 26 2020, an attacker executed a theft of funds from the USDC and USDT vaults of Harvest Finance. The attacker exploited an arbitrage and impermanent loss that influences the value of individual assets inside the Y pool of Curve.fi, which is where the funds of Harvest’s vaults were invested. The following mechanics of the protocol allowed for executing such an attack:

1. Harvest’s investment strategies calculate the real-time value of assets invested in the underlying real-time protocols. **The vaults use the value of the assets to calculate the number of shares to be issued to the user depositing the funds. They also use the value of the assets when users remove funds from the vaults to calculate the payout that a user should receive upon exit.**
2. The assets inside some of the vaults \(including USDC and USDT\) are deposited into shared pools of underlying DeFi protocols \(such as the Y pool on Curve.fi\). **The assets inside such pools are subject to market effects such as** [**impermanent loss**](https://academy.binance.com/en/articles/impermanent-loss-explained)**, arbitrage, and slippage**. Thus, their value can be manipulated via market trades with a large volume.

The attacker repeatedly exploited the effects of [impermanent loss](https://academy.binance.com/en/articles/impermanent-loss-explained) of USDC and USDT inside the Y pool on Curve.fi. They used the manipulated asset value to deposit funds into the Harvest’s vaults and obtain vault shares for a beneficial price, and later exit the vault at a regular share price generating a profit. The following chain of events tracks the attack:

1. The attacker’s wallet address is 0xf224ab004461540778a914ea397c589b677e27bb. It deployed a contract 0xc6028a9fa486f52efd2b95b949ac630d287ce0af through which they carried out the entire attack on October 26, 2020, 02:53:31 AM +UTC. **The 10 ETH for the attack was sourced through Tornado in transaction** [0x4b7b9e387a79289720a0226f695913d1d11dbdc681b7218a432136cc089363c4](https://etherscan.io/tx/0x4b7b9e387a79289720a0226f695913d1d11dbdc681b7218a432136cc089363c4).
2. The attack itself initiated in transaction [0x35f8d2f572fceaac9288e5d462117850ef2694786992a8c3f6d02612277b0877](https://etherscan.io/tx/0x35f8d2f572fceaac9288e5d462117850ef2694786992a8c3f6d02612277b0877). Within the context of a single transaction:
3. The attacker sourced a large amount of USDT \(18,308,555.417594\) and USDC \(50,000,000\) from Uniswap into the attacking contract.
4. The contract converted 17,222,012.640506 USDT into USDC via a swap inside Y pool. The effect of the swap was a higher value of USDC inside the Y pool as the other assets incurred impermanent loss. The smart contract obtained a roughly equivalent amount of 17,216,703.208672 USDC.
5. The attacker deposited 49,977,468.555526 USDC into Harvest’s USDC vault, receiving the total of 51,456,280.788906 fUSDC at 0.97126080216 USDC per share. The price of a share before the attack was 0.980007 USDC, so the attacker decreased the value of the share by approximately 1%. The arbitrage check inside Harvest’s strategy did not exceed the threshold of 3% and thus did not revert the transaction.
6. The attacker exchanged 17,239,234.653146 of USDC back into USDT via the Y pool. The result was obtaining the original lower value of USDC inside the Y pool due to reverting of the impermanent loss effect. The attacker received 17,230,747.185604 USDT back.
7. The attacker withdrew from Harvest’s USDC vault trading all fUSDC shares back for 50,596,877.367825 USDC. The price of a share was 0.98329837664 USDC as the value of USDC inside the Y pool decreased. The USDC was paid entirely by the buffer of the Harvest’s USDC vault, not interacting with Y pool at all. The net profit \(not accounting for the flash loan fees\) was 619408.812299 USDC.The attacker repeated the process several times within the same transaction.
8. After executing 17 attack transactions aimed at the USDC vault within 4 minutes, the attacker repeated the process in the analogous way for the USDT vault starting with transaction [0x0fc6d2ca064fc841bc9b1c1fad1fbb97bcea5c9a1b2b66ef837f1227e06519a6](https://etherscan.io/tx/0x0fc6d2ca064fc841bc9b1c1fad1fbb97bcea5c9a1b2b66ef837f1227e06519a6). They executed 13 transactions targeting the USDT vault within another 3 minutes.
9. At the end of the process at October 26, 2020, 03:01:48 AM +UTC, the attacker transferred 13,000,000 USDC and 11,000,000 USDT from the attacking contract to address 0x3811765a53c3188c24d412daec3f60faad5f119b in transaction [0x53fae6f1d6b8a76a666a0bf7f9c724e6006465e544f89f1515b939d8911e8c58](https://etherscan.io/tx/0x53fae6f1d6b8a76a666a0bf7f9c724e6006465e544f89f1515b939d8911e8c58).
10. The attacker transferred some funds back to the Harvest deployer in transaction [0x25119cd54a4562aa427d9770af383512f9cb5e8e4d17232ad96b69dc293a3510](https://etherscan.io/tx/0x25119cd54a4562aa427d9770af383512f9cb5e8e4d17232ad96b69dc293a3510). This was 1,761,898.396474 USDC and 718,914.048541 USDT.

The log of transactions pertaining to the attack is visible on the attacker’s address [0xf224ab004461540778a914ea397c589b677e27bb](https://etherscan.io/address/0xf224ab004461540778a914ea397c589b677e27bb).

## **Possible mitigations in the future** <a id="ee92"></a>

The Harvest Finance team is committed to evaluating possible mitigation strategies and implementing them alongside with any necessary UX changes in the upcoming releases. We will use the **upgradability feature of new vaults and timelock-based investment strategy** replacement and communicate mitigation strategies well in advance of release.

The possible remediation techniques include the following options:

### **Implementing a commit-and-reveal mechanism for deposits**

This would **remove the ability to perform deposits and withdrawals within a single transaction** and therefore, make flash-loan based attacks infeasible. On users’ side, this would mean that during depositing, their tokens will be transferred into Harvest in one transaction. The users would subsequently claim their share in another transaction, ideally inside a different block. This would constitute a UX change and potentially incur a higher, yet still acceptable, gas cost for the depositors.

### **A stricter configuration of the existing deposit arb check in the strategies**

The current threshold was set to 3%, and it was not sufficient to protect the vault against such an attack. A stricter threshold can make such an attack economically infeasible, however, it may be limiting deposits in the case of a natural impermanent loss effects. Sunday’s events over 7 minutes show that this measure is not effective enough and thus should be seen as complementary to other ones.

### **Withdrawals in an underlying asset**

When users deposit into vaults that use share pools \(such as the Y pool\), they effectively trade their single asset for the pool asset \(such as the yCurve\). If the users withdrew only the underlying asset, they would be able to trade them for an asset combination as per the current market conditions. If the market was manipulated, the trade would be also subject to such manipulation, which would prevent the attacking entity from generating a profit. From a regular user’s perspective, withdrawing yCRV could be followed by conversion into a stablecoin in a separate transaction. While this requires a UX change, this could also address the social slippage problem, and thus could be beneficial for the protocol. The disadvantage of this approach is that it binds the vault withdrawal mechanism to the strategy that is currently being used: if a strategy is switched to another strategy that does not use the shared underlying pool, or uses a different pool, the asset produced by the withdrawal would change too.

### **Using oracles for determining asset price**

While an approximate asset price may be effectively determined from external oracles \(provided by Chainlink or Maker\), it would have a very loose connection to the real share price. If the value of assets inside the underlying DeFi protocol differed from the value reported by the oracle, the vault would be exposed to free arbitrage and a flashloan attack. This is not a solution for Harvest, however, oracles will be considered in the system design and possible mitigation strategies \(as they have been considered up to this point\).

## Resources

* [https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217)
* [https://www.paradigm.xyz/2020/11/so-you-want-to-use-a-price-oracle/](https://www.paradigm.xyz/2020/11/so-you-want-to-use-a-price-oracle/)

