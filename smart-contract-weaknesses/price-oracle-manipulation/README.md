# Price Oracle Manipulation

## **Introduction**

On Ethereum, where everything is a smart contract, so too are price oracles. As such, it’s more useful to distinguish between how the price oracle gets its price information. In one approach, you can simply take the existing off-chain price data from price APIs or exchanges and bring it on-chain. In the other, you can calculate the instantaneous price by consulting on-chain decentralized exchanges.

![](https://samczsun.com/content/images/2020/11/image-24.png)

Both options have their respective advantages and disadvantages. Off-chain data is generally slower to react to volatility, which may be good or bad depending on what you’re trying to use it for. It typically requires a handful of privileged users to push the data on-chain though, so you have to trust that they won’t turn evil and can’t be coerced into pushing bad updates. **On-chain data doesn’t require any privileged access and is always up-to-date, but this means that it’s easily manipulated by attackers.**

## What is decentralized lending? <a id="what-is-decentralized-lending"></a>

First, let's talk about traditional lending. When you take out a loan, you typically need to provide some sort of collateral so that if you default on your loan, the lender can then seize the collateral. In order to determine how much collateral you need to supply, the lender typically knows or can reliably calculate the fair market value \(FMV\) of the collateral.

In decentralized lending, the same process occurs except now the lender is a smart contract that is isolated from the outside world. This means that it can't simply "know" the FMV of whatever collateral you're trying to provide.

To solve this problem, developers instruct the smart contract to query an _oracle_, which accepts the address of a token and returns the current price of that token in a desired currency \(for example, ETH or USD\). Different DeFi projects have taken different approaches to implementing this oracle, but they can generally all be classified in one of five ways \(although some implementations blur the lines more than others\):

1. Off-chain Centralized Oracle This type of oracle simply accepts new prices from an off-chain source, typically an account controlled by the project. Due to the need to quickly update the oracle with new exchange rates, the account is typically an EOA and not a multisig. There may be some sanity checking to ensure that prices don't fluctuate too wildly. [Compound Finance](https://compound.finance/) and [Synthetix](https://www.synthetix.io/) mainly use this type of oracle for most assets
2. Off-chain Decentralized Oracle This type of oracle accepts new prices from multiple off-chain sources and merges the values through a mathematical function, such as an average. In this model, a multisig wallet is typically used to manage the list of authorized sources. [Maker](https://makerdao.com/feeds/) uses this type of oracle for ETH and other assets
3. On-chain Centralized Oracle This type of oracle determines the price of assets using an on-chain source, such as a DEX. However, only a central authority can trigger the oracle to read from the on-chain source. Like an off-chain centralized oracle, this type of oracle requires rapid updates and as such the triggering account is likely an EOA and not a multisig. [dYdX](https://dydx.exchange/) and [Nuo](https://nuo.network/) use this type of oracle for certain assets
4. On-chain Decentralized Oracle This type of oracle determines the price of assets using an on-chain source, but can be updated by anyone. There may be some sanity checking to ensure that prices don't fluctuate too wildly. [DDEX](https://margin.ddex.io/) uses this type oracle for DAI, while [bZx](https://bzx.network/) uses this type of oracle for all assets
5. Constant Oracle This type of oracle simply returns a constant value, and is typically used for stablecoins. Nearly all projects mentioned above use this type of oracle for USDC due to its guaranteed peg

## Undercollateralized Loans

Imagine you wanted to bring decentralized lending to the blockchain. Users are allowed to deposit assets as collateral and borrow other assets up to a certain amount determined by the value of the assets they’ve deposited. Let’s assume that a user wants to borrow USD using ETH as collateral, that the current price of ETH is 400 USD, and that the collateralization ratio is 150%.

If the user deposits 375 ETH, they’ll have deposited 150,000 USD of collateral. They can borrow 1 USD for every 1.5 USD of collateral, so they’ll be able to borrow a maximum 100,000 USD from the system.

But of course, on the blockchain it’s not as simple as simply declaring that 1 ETH is worth 400 USD because a malicious user could simply declare that 1 ETH is worth 1,000 USD and then take all the money from the system. As such, it’s tempting for developers to reach for the nearest price oracle shaped interface, such as the current spot price on Uniswap, Kyber, or another decentralized exchange.

At first glance, this appears to be the correct thing to do. After all, Uniswap prices are always roughly correct whenever you want to buy or sell ETH as any deviations are quickly correct by arbitrageurs. However, as it turns out, **the spot price on a decentralized exchange may be wildly incorrect during a transaction as shown in the example below.**

Consider how a Uniswap reserve functions. **The price is calculated based on the amount of assets held by the reserve**, but the assets held by the reserve changes as users trade between ETH and USD. **What if a malicious user performs a trade before and after taking a loan from your platform?**

Before the user takes out a loan, they buy 5,000 ETH for 2,000,000 USD. The Uniswap exchange now calculates the price to be 1 ETH = 1,733.33 USD. Now, their 375 ETH can act as collateral for up to 433,333.33 USD worth of assets, which they borrow. Finally, they trade back the 5,000 ETH for their original 2,000,000 USD, which resets the price. **The net result is that your loan platform just allowed the user to borrow an additional 333,333.33 USD without putting up any collateral.**

![](../../.gitbook/assets/image%20%2810%29.png)

This case study illustrates the most common mistake when using a decentralized exchange as a price oracle - **an attacker has almost full control over the price during a transaction** and trying to read that price accurately is like reading the weight on a scale before it’s finished settling. You’ll probably get the wrong number and depending on the situation it might cost you a lot of money.

## Key Takeaways <a id="key-takeaways"></a>

### Don't use an on-chain decentralized oracle without some sort of validation <a id="don-t-use-an-on-chain-decentralized-oracle-without-some-sort-of-validation"></a>

Due to the nature of on-chain decentralized oracles, ensure that you're validating the rate being returned, whether it's by taking the order \(thereby nullifying any gains which may have been realized\), **comparing the rate against known good rates \(in the case of DAI\), or comparing the rate in both directions**.

### Consider the implications of dependencies on third-party projects <a id="consider-the-implications-of-dependencies-on-third-party-projects"></a>

In both cases, DDEX and bZx assumed that Uniswap and Kyber would be a source of accurate price data. However, **an accurate rate for a DEX means that a trade can be made using that rate, while an accurate rate for a DeFi project means that it is close to or equal to the FMV**. In other words, an accurate rate for a DeFi project is an accurate rate for a DEX, but the opposite might not be true.

Furthermore, bZx's second attempt at solving this problem was insufficient due to a misunderstanding in how the Kyber Network internally calculates the exchange rate between two non-ETH tokens.

As such, before introducing a dependency on a third-party project, consider not only whether the project has been audited, but also whether the project's specifications and threat model align with your own. If you have the time, taking an in-depth look at their contracts also doesn't hurt.

## Preventative Techniques

By now, I hope that you’ve learned to recognize the common thread - it's not always obvious that you're using a price oracle and if you don't follow the proper precautions, an attacker could trick your protocol into sending them all of your money. While there’s no one-size-fits-all fix that can be prescribed, here are a few solutions that have worked for other projects in the past. Maybe one of them will apply to you too.

### Shallow Markets, No Diving <a id="shallow-markets-no-diving"></a>

Like diving into the shallow end of a pool, diving into a shallow market is painful and might result in significant expenses which will change your life forever. Before you even consider the intricacies of the specific price oracle you’re planning to use, consider whether the token is liquid enough to warrant integration with your platform.

### A Bird in the Hand is Worth Two in the Bush <a id="a-bird-in-the-hand-is-worth-two-in-the-bush"></a>

It may be mesmerizing to see the potential exchange rate on Uniswap, but nothing’s final until you actually click trade and the tokens are sitting in your wallet. Similarly, the best way to know for sure the exchange rate between two assets is to simply swap the assets directly. This approach is great because there’s no take-backs and no what-ifs. However, it may not work for protocols such as lending platforms which are required to hold on to the original asset.

### Almost Decentralized Oracles <a id="almost-decentralized-oracles"></a>

One way to summarize the problem with oracles that rely on on-chain data is that they’re a little too up-to-date. If that’s the case, why not introduce a bit of artificial delay? Write a contract which updates itself with the latest price from a decentralized exchange like Uniswap, but only when requested by a small group of privileged users. Now even if an attacker can manipulate the price, they can’t get your protocol to actually use it.

This approach is really simple to implement and is a quick win, but there are a few drawbacks - in times of chain congestion you might not be able to update the price as quickly as you’d like, and you’re still vulnerable to sandwich attacks. Also, now your users need to trust that you’ll actually keep the price updated.

### Speed Bumps <a id="speed-bumps"></a>

Manipulating price oracles is a time-sensitive operation because arbitrageurs are always watching and would love the opportunity to optimize any suboptimal markets. If an attacker wants to minimize risk, they’ll want to do the two trades required to manipulate a price oracle in a single transaction so there’s no chance that an arbitrageur can jump in the middle. As a protocol developer, if your system supports it, it may be enough to simply implement a delay of as short as 1 block between a user entering and exiting your system.

Of course, this might impact composability and miner collaboration with traders is on the rise. In the future, it may be possible for bad actors to perform price oracle manipulation across multiple transactions knowing that the miner they’ve partnered with will guarantee that no one can jump in the middle and take a bite out of their earnings.

### Time-Weighted Average Price \(TWAP\) <a id="time-weighted-average-price-twap-"></a>

Uniswap V2 introduced a TWAP oracle for on-chain developers to use. The [documentation](https://uniswap.org/docs/v2/core-concepts/oracles/) goes into more detail on the exact security guarantees that the oracle provides, but in general for large pools over a long period of time with no chain congestion, the TWAP oracle is highly resistant to oracle manipulation attacks. However, due to the nature of its implementation, it may not respond quickly enough to moments of high market volatility and only works for assets for which there is already a liquid token on-chain.

### M-of-N Reporters <a id="m-of-n-reporters"></a>

Sometimes they say that if you want something done right, you do it yourself. What if you gather up N trusted friends and ask them to submit what they think is the right price on-chain, and the best M answers becomes the current price?

This approach is used by many large projects today: Maker runs a set of [price feeds](https://developer.makerdao.com/feeds/) operated by trusted entities, Compound created the [Open Oracle](https://medium.com/compound-finance/announcing-compound-open-oracle-development-cff36f06aad3) and features reporters such as [Coinbase](https://blog.coinbase.com/introducing-the-coinbase-price-oracle-6d1ee22c7068), and Chainlink aggregates price data from Chainlink operators and exposes it on-chain. Just keep in mind that if you choose to use one of these solutions, you’ve now delegated trust to a third party and your users will have to do the same. Requiring reporters to manually post updates on-chain also means that during times of high market volatility and chain congestion, price updates may not arrive on time.

## Resources

* [https://samczsun.com/so-you-want-to-use-a-price-oracle/](https://samczsun.com/so-you-want-to-use-a-price-oracle/)
* [https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/)
* [https://shouldiusespotpriceasmyoracle.com/](https://shouldiusespotpriceasmyoracle.com/)
* [https://ethereum.org/en/developers/docs/oracles/](https://ethereum.org/en/developers/docs/oracles/)
* [https://docs.uniswap.org/protocol/concepts/V3-overview/oracle](https://docs.uniswap.org/protocol/concepts/V3-overview/oracle)
* [https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/](https://blog.trailofbits.com/2020/08/05/accidentally-stepping-on-a-defi-lego/)
* [https://www.palkeo.com/en/projets/ethereum/bzx.html](https://www.palkeo.com/en/projets/ethereum/bzx.html)
* [https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217](https://medium.com/harvest-finance/harvest-flashloan-economic-attack-post-mortem-3cf900d65217)

