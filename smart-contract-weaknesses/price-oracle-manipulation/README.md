# Price Oracle Manipulation

## Price Oracle Manipulation

On Ethereum, where everything is a smart contract, so too are price oracles. As such, it’s more useful to distinguish between how the price oracle gets its price information. In one approach, you can simply take the existing off-chain price data from price APIs or exchanges and bring it on-chain. In the other, you can calculate the instantaneous price by consulting on-chain decentralized exchanges.

![](https://samczsun.com/content/images/2020/11/image-24.png)

Both options have their respective advantages and disadvantages. Off-chain data is generally slower to react to volatility, which may be good or bad depending on what you’re trying to use it for. It typically requires a handful of privileged users to push the data on-chain though, so you have to trust that they won’t turn evil and can’t be coerced into pushing bad updates. **On-chain data doesn’t require any privileged access and is always up-to-date, but this means that it’s easily manipulated by attackers.**

## Undercollateralized Loans

Imagine you wanted to bring decentralized lending to the blockchain. Users are allowed to deposit assets as collateral and borrow other assets up to a certain amount determined by the value of the assets they’ve deposited. Let’s assume that a user wants to borrow USD using ETH as collateral, that the current price of ETH is 400 USD, and that the collateralization ratio is 150%.

If the user deposits 375 ETH, they’ll have deposited 150,000 USD of collateral. They can borrow 1 USD for every 1.5 USD of collateral, so they’ll be able to borrow a maximum 100,000 USD from the system.

But of course, on the blockchain it’s not as simple as simply declaring that 1 ETH is worth 400 USD because a malicious user could simply declare that 1 ETH is worth 1,000 USD and then take all the money from the system. As such, it’s tempting for developers to reach for the nearest price oracle shaped interface, such as the current spot price on Uniswap, Kyber, or another decentralized exchange.

At first glance, this appears to be the correct thing to do. After all, Uniswap prices are always roughly correct whenever you want to buy or sell ETH as any deviations are quickly correct by arbitrageurs. However, as it turns out, **the spot price on a decentralized exchange may be wildly incorrect during a transaction as shown in the example below.**

Consider how a Uniswap reserve functions. **The price is calculated based on the amount of assets held by the reserve**, but the assets held by the reserve changes as users trade between ETH and USD. **What if a malicious user performs a trade before and after taking a loan from your platform?**

Before the user takes out a loan, they buy 5,000 ETH for 2,000,000 USD. The Uniswap exchange now calculates the price to be 1 ETH = 1,733.33 USD. Now, their 375 ETH can act as collateral for up to 433,333.33 USD worth of assets, which they borrow. Finally, they trade back the 5,000 ETH for their original 2,000,000 USD, which resets the price. **The net result is that your loan platform just allowed the user to borrow an additional 333,333.33 USD without putting up any collateral.**

![](../../.gitbook/assets/image%20%2810%29.png)

This case study illustrates the most common mistake when using a decentralized exchange as a price oracle - **an attacker has almost full control over the price during a transaction** and trying to read that price accurately is like reading the weight on a scale before it’s finished settling. You’ll probably get the wrong number and depending on the situation it might cost you a lot of money.

## Resources

* [https://samczsun.com/so-you-want-to-use-a-price-oracle/](https://samczsun.com/so-you-want-to-use-a-price-oracle/)
* [https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/)
* [https://shouldiusespotpriceasmyoracle.com/](https://shouldiusespotpriceasmyoracle.com/)
* [https://ethereum.org/en/developers/docs/oracles/](https://ethereum.org/en/developers/docs/oracles/)
* [https://docs.uniswap.org/protocol/concepts/V3-overview/oracle](https://docs.uniswap.org/protocol/concepts/V3-overview/oracle)

