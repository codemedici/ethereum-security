# bZx Second Hack

## Introduction

> By relying on an on-chain decentralized price oracle without validating the rates returned, [DDEX](https://margin.ddex.io/) and [bZx](https://bzx.network/) were susceptible to atomic price manipulation. This would have resulted in the loss of liquid ETH in the ETH/DAI market for DDEX, and loss of all liquid funds in bZx. Fortunately, no funds were actually lost.

In February 2020, bZx was hacked twice over the span of several days for approximately 1MM USD. You can find an excellent technical analysis of both hacks written by palkeo [here](https://www.palkeo.com/en/projets/ethereum/bzx.html), but we will only be looking at the second hack.

In the second hack, the attacker first purchased nearly all of the sUSD on Kyber using ETH. Then, the attacker purchased a second batch of sUSD from Synthetix itself and deposited it on bZx. Using the sUSD as collateral, the attacker borrowed the maximum amount of ETH they were allowed to. They then sold back the sUSD to Kyber.

If you’ve been paying attention, you’ll recognize this as essentially the same [undercollateralized loan attack](./#undercollateralized-loans), but using a different collateral and a different decentralized exchange.

Let’s now look at the second transaction [0x7628…](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/), which happened on Tuesday 18th February 2020 at 03:13:58 UTC.

It caused the same effect as the first one, namely opening an under-collateralized position on bZx. However it uses a completely different method, and is more straightforward to understand.

## High-level overview

Again, I recommend you use either [Oko](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/) or [EthDecoder](http://ethtx.info/mainnet/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15).

Here are the main calls that happened during it:

> 1. The attacker borrows 7500 ETH from bZx \(flash borrow\)
> 2. The attacker repeatedly calls Kyber to convert 900 ETH to 155,994 sUSD \(distorting the Kyber sUSD prices\)
> 3. The attacker uses the Synthetix depot contract to convert 3518 ETH to 943,837 sUSD
> 4. The attacker borrows 6796 ETH on bZx, sending only 1,099,841 sUSD \(oracle attack\)
> 5. The attacker transfers back 7500 ETH to bZx to repay their flash loan

At the end the attacker ends up with 2378 ETH in their [attack contract](https://oko.palkeo.com/0x360f85F0B74326CDDfF33A812B05353BC537747B/). They transfer it to their EOA [shortly after](https://oko.palkeo.com/0x2e05b36f4e1afd92366dfded4f7fb7a11eab4d681e313caab6068b1a5879067f/).

## Walkthrough of the transaction

Before you start, I recommend you take a look at the [decompiled contract code](https://oko.palkeo.com/0x360f85F0B74326CDDfF33A812B05353BC537747B/code/), you can see the sequence of calls to be made hardcoded there.

Let’s now go through each step of the transaction:

### **A. The bZx flash borrow**

[This step](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0) is comparable with the first step of the first exploit. Only it uses bZx instead of DyDx.

Again, **the goal is to borrow enough money to be able to pull off the exploit**, and again, the rest of the attacker activity happen inside of [a callback to the attacker contract](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3), initiated by bZx.

### **B. Distorting the Kyber prices**

Kyber uses “reserves”, which provides liquidity. For the ETH-sUSD pair there are two reserves:

> * Uniswap.
> * A Synthetix one, that implements a [LiquidityConversionRates](https://developer.kyber.network/docs/API_ABI-LiquidityConversionRates/) that automatically adjusts the price \(conceptually similar to Uniswap\).

A trade will necessarily either hit one of them \(depending on which one gives the best price\).

**The attacker contract buys most of the sUSD liquidity available on both reserves**. For that, they do 19 successive buys:

> * [540 ETH to 92k sUSD](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_1), hitting the Uniswap Kyber reserve.
> * [20 ETH to 5.2 sUSD](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_2), hitting the sUSD Kyber reserve.
> * [20 ETH to 4.9 sUSD](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_3), hitting the sUSD Kyber reserve.
> * [20 ETH to 4.7 sUSD](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_4), hitting the sUSD Kyber reserve.
> * …

You can see **each trade getting a worse price**. That’s the attacker skewing the prices by eating all available liquidity.

We get **from 270 ETH/sUSD \(normal rate\) to a price of 111 ETH/sUSD in Kyber.**

### **C. Buying a lot of sUSD at a normal rate**

This is done using the [Synthetix Depot contract](https://oko.palkeo.com/0x172E09691DfBbC035E37c73B62095caa16Ee2388/) which has a lot of liquidity that you can access.

The attacker calls the [exchangeEtherForSynths\(\)](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_20) function to **exchange 6000 ETH to 943,837 sUSD**.

**The rate is 157 ETH/sUSD. It’s a bad rate, but significantly better than the distorted rate that Kyber now returns** \(see above\).

### **D. Borrowing ETH for sUSD on bZx \(oracle attack\)**

This is where the oracle attack is executed.

I strongly recommend that you read [this article from Sam Sun](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/) about oracle attacks, which explain how it works.

The idea is that **bZx queries Kyber for the current ETH/sUSD rate**, but now that the attacker distorted the market it will get an erroneous rate! **This allows the attacker to borrow much more ETH than they could normally** with this amount of sUSD, because bZx is fed a wrong price oracle.

To do it, they simply call the iETH contract’s [borrowTokenFromDeposit\(\)](https://oko.palkeo.com/0x762881b07feb63c436dee38edd4ff1f7a74c33091e534af56c9f7d49b5ecac15/#call_0_3_0_23_0) function. **They send 1,099,841 sUSD \(they bought the bought from the Synthetix Depot, and some more while distorting the Kyber prices**\), and are able to borrow 6796 ETH.

#### **Effect on bZx pool**

We can compute that bZx sent them 1.7mm$ and received only 1.1mm$ worth of sUSD. That’s an equity loss of around 600k$.

#### **Wait, why did it work?**

If you read [the article](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/) about oracle attacks, you probably wonder how come the exploit worked. The article describes a very similar attack, and it was fixed.

Let’s quote the [last fix](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/#solution-3) that was implemented after that disclosure:

> The bZx team reverted their changes for the previous attack and instead implemented a spread check, such that if the spread was above a certain threshold then the loan would be rejected. This solution handles the generic case so long as both tokens being queried has at least one non-manipulable reserve on Kyber, which is currently the case for all whitelisted tokens.

The check was implemented [here](https://github.com/bZxNetwork/bZx-monorepo/blob/c5fdab1eb7e0f158841671c78d324045cb438f3c/packages/contracts/contracts/oracle/BZxOracle.sol#L1388):

```text
require(
   spreadPercentage <= maxSpread,
   "bad price"
);
```

This means that bZx will check the price in both directions and look at the difference. However, here both reserves are Uniswap-like and both got their price manipulated.

So, yes, bZx was supposed to check for a small enough spread. But both these reserves do have a constant, small spread \(that depends on their fees\). So the checks did pass.

### **E. Settling the debt**

Now that the attacker realized a profit in ETH, they can pay back the 7500 ETH, so the transaction can terminate correctly because the flash loan has been paid back.

Also note that if they wanted, the attacker could have pulled back the price that they skewed first for more profit. But if you look at the numbers you notice they spent 900 ETH to skew the price, compared to the 3518 ETH worth of sUSD that they bought from the Synthetix Depot and leveraged on bZx. Because the money they spent on skewing the price was only 10% of the amount they magically multiplied later, they didn’t need to care.

## Conclusion

The second attack was much simpler than the first one, and this was indeed an oracle attack.

Both attacks \(see [bZx first hack](bzx-first-hack.md)\) exploited the fact that it is possible to borrow huge amounts of liquidity for the duration of a single transaction \(“flash loan”\). Part of this liquidity ends up doing a massive buy/sell on an on-chain exchange \(like Kyber\): this order has a huge spread and shifts a lot the market price.

But the comparison stops here. The attack vector is very different:

> * In the first attack, the attacker makes bZx do a bad trade by opening a leveraged position, and then profit off arbitraging it back.
> * In the second one, the attacker first skews the price, and then borrows ETH to trigger their oracle attack on bZx \(no internal Kyber trade happens\).

## Resources

* [https://www.palkeo.com/en/projets/ethereum/bzx.html](https://www.palkeo.com/en/projets/ethereum/bzx.html)
* [https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/)
* [https://www.paradigm.xyz/2020/11/so-you-want-to-use-a-price-oracle/](https://www.paradigm.xyz/2020/11/so-you-want-to-use-a-price-oracle/)



