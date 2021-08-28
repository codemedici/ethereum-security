# Preventative Techniques

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

