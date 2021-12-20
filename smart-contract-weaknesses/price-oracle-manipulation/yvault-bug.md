# yVault (Bug Bounty)

On July 25, 2020, I reported a bug to yEarn regarding the launch of their new yVault contracts. You can read the official writeup about this bug [here](https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/), but I will briefly summarize it below.

The yVault system allows users to deposit a token and earn yield on it without needing to manage it themselves. Internally, the vault tracks the total amount of yVault tokens minted as well as the total amount of underlying tokens deposited. **The worth of a single yVault token is given by the ratio of tokens minted to tokens deposited.** Any yield the vault earns is spread across all minted yVault tokens (and therefore, across all yVault token holders).

The first yVault allowed users to earn yield on USDC by supplying liquidity to the Balancer MUSD/USDC pool. When a user supplies liquidity to Balancer pools, they receive BPT in return which can be redeemed for a proportion of the pool. As such, **the yVault calculated the value of its holdings based on the amount of MUSD/USDC which could be redeemed with its BPT.**

This seems like the correct implementation, but unfortunately the same principle as given before applies - **the state of the Balancer pool during a transaction is not stable and cannot be trusted**. In this case, because of the bonding curve that Balancer chose, a user who swaps between from USDC to MUSD will not receive a 1:1 exchange rate, but will in fact leave behind some MUSD in the pool. This means that the value of BPT can be temporarily inflated, which allows an attacker to manipulate the price at will and subsequently drain the vault.

![](https://samczsun.com/content/images/2020/11/image-30.png)

This incident shows that price oracles are not always conveniently labelled as such, and that developers need to be vigilant about what sort of data they’re ingesting and consider whether that data can be easily manipulated by an unprivileged user.

Since the initial release integrated with Balancer, let’s consider how Balancer works. **Balancer removes the need for liquidity providers to manually rebalance their portfolio** by incentivizing rational market actors to do so instead. If a token goes up in price, the pool will become unbalanced. While normally a liquidity provider may need to pay fees in order to sell a token that has increased in value, Balancer incentivizes external users to pay a fee for the privilege of purchasing the token at a profit instead. The fees paid are then distributed to the liquidity providers.

Figure 1 presents the equation used to calculate the amount of tokens received based on the state of the Balancer pool and the amount of tokens sent. For the remainder of this post, let’s refer to the MUSD/USDC 50/50 pool. The swap fee is 0.05%.

```
/**********************************************************************************************
// calcOutGivenIn                                                                            //
// aO = tokenAmountOut                                                                       //
// bO = tokenBalanceOut                                                                      //
// bI = tokenBalanceIn              /      /            bI             \    (wI / wO) \      //
// aI = tokenAmountIn    aO = bO * |  1 - | --------------------------  | ^            |     //
// wI = tokenWeightIn               \      \ ( bI + ( aI * ( 1 - sF )) /              /      //
// wO = tokenWeightOut                                                                       //
// sF = swapFee                                                                              //
**********************************************************************************************/
```

First, to get a sense of how this function behaves, we’ll see what happens when a rational market actor swaps a pool back into balance and when an irrational market actor swaps a pool out of balance.

{% hint style="info" %}
The Balancer pool incentivizes arbitrageurs to arb the price between the currency pair until a 1:1 ratio is reached.
{% endhint %}

Suppose the pool is currently out of balance and contains 1,100,000 USDC and 900,000 MUSD. If a rational market actor pays 90,000 MUSD, they’ll receive 99,954 USDC in exchange and make 9,954 USDC in profit. A very good deal!

Now **suppose the pool is currently balanced and contains 1,000,000 USDC and 1,000,000 MUSD**. What happens if an irrational market actor pays 100,000 USDC? Well, they would receive 90,867 MUSD for a loss of 9,133 MUSD. Not such a great deal.

{% hint style="info" %}
If the pool is in balance, there is a disincentive in performing swaps, because the trader will receive less money in order to maintain that ratio in the pool.&#x20;
{% endhint %}

Although the second trade results in an immediate loss and thus seems rather useless, pairing it with the first trade results in some interesting behavior.

Consider a user who first performs The Bad Trade: The user converts 100,000 USDC to 90,867 MUSD, losing 9,133 USD in the process. Then, the user performs The Good Trade and converts 90,867 MUSD to 99,908 USDC, earning 9,041 USD in the process. This results in a net loss of 92 USD. Not ideal, but certainly not as bad as the loss of 9,200 USD.

Now consider the valuation of BPT during this process. If you held 1% of the total BPT, at the start of the transaction your tokens would have been worth 1% of 2,000,000 USD, or 20,000 USD. At the end of the transaction, your tokens would have been worth 1% of 2,000,092 USD, or 20,000.96 USD. Yet for a magical moment, **right in the middle of the transaction, your tokens were worth 1% of 2,009,133 USD, or 20,091.33 USD. This is the crux of the vulnerability at hand.**

Knowing this, I applied the same process behavior to yVault. Before The Bad Trade, the vault holds some BPT worth some amount of USD. After The Good Trade, the vault holds the same amount of BPT worth a slightly larger amount of USD. However, **between The Bad Trade and The Good Trade, the vault holds some BPT worth a significantly larger amount of USD.**

{% hint style="danger" %}
**Recall that the value of yUSDC is directly proportional to the value of the BPT it holds. If we bought yUSDC before The Bad Trade and sold yUSDC before The Good Trade, we would instantaneously make a profit.** **Repeat this enough times, and we would drain the vault.**
{% endhint %}

### How was it fixed?

It turns out that accurately calculating the true value of BPT and preventing attackers from extracting profit from slippage is a difficult problem to solve. Instead, the developer, Andre, deployed a new strategy that simply converts USDC to MUSD and supplies it to the mStable savings account was deployed and activated.

## Resources

* [https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/](https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/)
