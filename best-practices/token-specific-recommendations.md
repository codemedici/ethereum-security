# Token Specific Recommendations

In terms of smart contract development, it's important to follow standards. Standards are set to prevent vulnerabilities, and ignoring them can lead to unexpected effects.

Take for example binance's original BNB token. It was marketed as an ERC20 token, but it was later pointed out that it wasn't actually ERC20 compliant for a few reasons:

* [It prevented sending to 0x0](token-specific-recommendations.md#prevent-transferring-tokens-to-the-contract-address)
* [It blocked transfers of 0 value](token-specific-recommendations.md#revert-on-transfer-to-the-zero-address)
* [It didn't return true or false for success or fail](token-specific-recommendations.md#missing-return-values)
* [it is pausable](token-specific-recommendations.md#pausable-tokens)

The main cause for concern with this improper implementation is that if it is used with a smart contract that expects an ERC-20 token, it will behave in unexpected ways. It could even get locked in the contract forever.

Although standards aren't always perfect, and may someday become antiquated, they foster the most secure smart contracts.

## [Weird ERC20 Tokens](https://github.com/d-xo/weird-erc20)

{% hint style="info" %}
Last updated **August 16 2021**. Check the [Github repo](https://github.com/d-xo/weird-erc20) for latest information and code examples on "Weird ERC20 Tokens"
{% endhint %}

This repository contains minimal example implementations in Solidity of ERC20 tokens with behaviour that may be surprising or unexpected. All the tokens in this repo are based on real tokens, many of which have been used to exploit smart contract systems in the past. It is hoped that these example implementations will be of use to developers and auditors.

The `ERC20` "specification" is so loosely defined that it amounts to little more than an interface declaration, and even the few semantic requirements that are imposed are routinely violated by token developers in the wild.

This makes building smart contracts that interface directly with ERC20 tokens challenging to say the least, and smart contract developers should in general default to the following patterns when interaction with external code is required:

1. A contract level allowlist of known good tokens.
2. Direct interaction with tokens should be performed in dedicated wrapper contracts at the edge of the system. This allows the core to assume a consistent and known good semantics for the behaviour of external assets.

In some cases the above patterns are not practical (for example in the case of a permissionless AMM, keeping an on chain allowlist would require the introduction of centralized control or a complex governance system), and in these cases developers must take great care to make these interactions in a highly defensive manner. It should be noted that even if an onchain allowlist is not feasible, an offchain allowlist in the official UI can also protect unsophisticated users from tokens that violate the contracts expectations, while still preserving contract level permissionlessness.

Finally if you are building a token, you are strongly advised to treat the following as a list of behaviours to avoid.

_Additional Resources_

* Trail of Bits [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md).
* Consensys Diligence [token interaction checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/)

### Reentrant Calls

Some tokens allow reentract calls on transfer (e.g. `ERC777` tokens).

This has been exploited in the wild on multiple occasions (e.g. [imBTC uniswap pool drained](https://defirate.com/imbtc-uniswap-hack/), [lendf.me drained](https://defirate.com/dforce-hack/))

_example_: [Reentrant.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Reentrant.sol)

### Missing Return Values

Some tokens do not return a bool (e.g. `USDT`, `BNB`, `OMG`) on ERC20 methods. see [here](https://gist.githubusercontent.com/lukas-berlin/f587086f139df93d22987049f3d8ebd2/raw/1f937dc8eb1d6018da59881cbc633e01c0286fb0/Tokens%20missing%20return%20values%20in%20transfer) for a comprehensive (if somewhat outdated) list.

Some tokens (e.g. `BNB`) may return a `bool` for some methods, but fail to do so for others. This resulted in stuck `BNB` tokens in Uniswap v1 ([details](https://mobile.twitter.com/UniswapProtocol/status/1072286773554876416)).

Some particulary pathological tokens (e.g. Tether Gold) declare a bool return, but then return `false` even when the transfer was successful ([code](https://etherscan.io/address/0x4922a015c4407f87432b179bb209e125432e4a2a#code)).

A good safe transfer abstraction ([example](https://github.com/Uniswap/uniswap-v2-core/blob/4dd59067c76dea4a0e8e4bfdda41877a6b16dedc/contracts/UniswapV2Pair.sol#L44)) can help somewhat, but note that the existance of Tether Gold makes it impossible to correctly handle return values for all tokens.

Two example tokens are provided:

* `MissingReturns`: does not return a bool for any erc20 operation
* `ReturnsFalse`: declares a bool return, but then returns false for every erc20 operation

_example_: [MissingReturns.sol](https://github.com/d-xo/weird-erc20/blob/main/src/MissingReturns.sol)\
_example_: [ReturnsFalse.sol](https://github.com/d-xo/weird-erc20/blob/main/src/ReturnsFalse.sol)

#### Resources

* [https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca)

### Fee on Transfer

Some tokens take a transfer fee (e.g. `STA`, `PAXG`), some do not currently charge a fee but may do so in the future (e.g. `USDT`, `USDC`).

The `STA` transfer fee was used to drain $500k from several balancer pools ([more details](https://medium.com/@1inch.exchange/balancer-hack-2020-a8f7131c980e)).

_example_: [TransferFee.sol](https://github.com/d-xo/weird-erc20/blob/main/src/TransferFee.sol)

### Balance Modifications Outside of Transfers (rebasing / airdrops)

Some tokens may make arbitrary balance modifications outside of transfers (e.g. Ampleforth style rebasing tokens, Compound style airdrops of governance tokens, mintable / burnable tokens).

Some smart contract systems cache token balances (e.g. Balancer, Uniswap-V2), and arbitrary modifications to underlying balances can mean that the contract is operating with outdated information.

In the case of Ampleforth, some Balancer and Uniswap pools are special cased to ensure that the pool's cached balances are atomically updated as part of the rebase prodecure ([details](https://www.ampltalk.org/app/forum/technology-development-17/topic/supported-dex-pools-61/)).

_example_: TODO: implement a rebasing token

### Upgradable Tokens

Some tokens (e.g. `USDC`, `USDT`) are upgradable, allowing the token owners to make arbitrary modifications to the logic of the token at any point in time.

A change to the token semantics can break any smart contract that depends on the past behaviour.

Developers integrating with upgradable tokens should consider introducing logic that will freeze interactions with the token in question if an upgrade is detected. (e.g. the [`TUSD` adapter](https://github.com/makerdao/dss-deploy/blob/7394f6555daf5747686a1b29b2f46c6b2c64b061/src/join.sol#L321) used by MakerDAO).

_example_: [Upgradable.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Upgradable.sol)

### Flash Mintable Tokens

There are some proposed token designs that allow for so called "flash minting", which would allow tokens to be minted for the duration of one transaction only, provided they are returned to the token contract by the end of the transaction.

This is similar to a flash loan, but does not require the tokens that are to be lent to exist before the start of the transaction. A token that can be flash minted could potentially have a total supply of max `uint256`.

A proposal to add such a facility to MakerDAO can be found [here](https://forum.makerdao.com/t/mip13c3-sp2-declaration-of-intent-dai-flash-mint-module/3635).

### Tokens with Blocklists

Some tokens (e.g. `USDC`, `USDT`) have a contract level admin controlled address blocklist. If an address is blocked, then transfers to and from that address are forbidden.

Malicious or compromised token owners can trap funds in a contract by adding the contract address to the blocklist. This could potentially be the result of regulatory action against the contract itself, against a single user of the contract (e.g. a Uniswap LP), or could also be a part of an extortion attempt against users of the blocked contract.

_example_: [BlockList.sol](https://github.com/d-xo/weird-erc20/blob/main/src/BlockList.sol)

### Pausable Tokens

Some tokens can be paused by an admin (e.g. `BNB`, `ZIL`).

Similary to the blocklist issue above, an admin controlled pause feature opens users of the token to risk from a malicious or compromised token owner.

_example_: [Pausable.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Pausable.sol)

### Approval Race Protections

Some tokens (e.g. `USDT`, `KNC`) do not allow approving an amount `M > 0` when an existing amount `N > 0` is already approved. This is to protect from an ERC20 attack vector described [here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA\_jp-RLM/edit#heading=h.b32yfk54vyg9).

[This PR](https://github.com/Uniswap/uniswap-v2-periphery/pull/26#issuecomment-647543138) shows some in the wild problems caused by this issue.

_example_: [Approval.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Approval.sol)

### Revert on Zero Value Transfers

Some tokens (e.g. `LEND`) revert when transfering a zero value amount.

_example_: [RevertZero.sol](https://github.com/d-xo/weird-erc20/blob/main/src/RevertZero.sol)

### Multiple Token Addresses

Some proxied tokens have multiple addresses. For example `TUSD` has two addresses: `0x8dd5fbCe2F6a956C3022bA3663759011Dd51e73E` and `0x0000000000085d4780B73119b644AE5ecd22b376` (calling transfer on either affects your balance on both).

As an example consider the following snippet. `rescueFunds` is intended to allow the contract owner to return non pool tokens that were accidentaly sent to the contract. However, it assumes a single address per token and so would allow the owner to steal all funds in the pool.

```
mapping isPoolToken(address => bool);
constructor(address tokenA, address tokenB) public {
  isPoolToken[tokenA] = true;
  isPoolToken[tokenB] = true;
}
function rescueFunds(address token, uint amount) external nonReentrant onlyOwner {
    require(!isPoolToken[token], "access denied");
    token.transfer(msg.sender, amount);
}
```

_example_: [Proxied.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Proxied.sol)

### Low Decimals

Some tokens have low decimals (e.g. `USDC` has 6). Even more extreme, some tokens like [Gemini USD](https://etherscan.io/token/0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd?a=0x5f65f7b609678448494De4C87521CdF6cEf1e932) only have 2 decimals.

This may result in larger than expected precision loss.

_example_: [LowDecimals.sol](https://github.com/d-xo/weird-erc20/blob/main/src/LowDecimals.sol)

### High Decimals

Some tokens have more than 18 decimals (e.g. `YAM-V2` has 24).

This may trigger unexpected reverts due to overflow, posing a liveness risk to the contract.

_example_: [HighDecimals.sol](https://github.com/d-xo/weird-erc20/blob/main/src/HighDecimals.sol)

### `transferFrom` with `src == msg.sender`

Some token implementations (e.g. `DSToken`) will not attempt to decrease the caller's allowance if the sender is the same as the caller. This gives `transferFrom` the same semantics as `transfer` in this case. Other implementations (e.g. OpenZeppelin, Uniswap-v2) will attempt to decrease the caller's allowance from the sender in `transferFrom` even if the caller and the sender are the same address, giving `transfer(dst, amt)` and `transferFrom(address(this), dst, amt)` a different semantics in this case.

_examples_:

Examples of both semantics are provided:

* [ERC20.sol](https://github.com/d-xo/weird-erc20/blob/main/src/ERC20.sol): does not attempt to decrease allowance
* [TransferFromSelf.sol](https://github.com/d-xo/weird-erc20/blob/main/src/TransferFromSelf.sol): always attempts to decrease the allowance

### Non `string` metadata

Some tokens (e.g. [`MKR`](https://etherscan.io/address/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2#code)) have metadata fields (`name` / `symbol`) encoded as `bytes32` instead of the `string` prescribed by the ERC20 specification.

This may cause issues when trying to consume metadata from these tokens.

_example_: [Bytes32Metadata.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Bytes32Metadata.sol)

### Revert on Transfer to the Zero Address

Some tokens (e.g. openzeppelin) revert when attempting to transfer to `address(0)`.

This may break systems that expect to be able to burn tokens by transfering them to `address(0)`.

_example_: [RevertToZero.sol](https://github.com/d-xo/weird-erc20/blob/main/src/RevertToZero.sol)

### No Revert on Failure

Some tokens do not revert on failure, but instead return `false` (e.g. [ZRX](https://etherscan.io/address/0xe41d2489571d322189246dafa5ebde1f4699f498#code)).

While this is technicaly compliant with the ERC20 standard, it goes against common solidity coding practices and may be overlooked by developers who forget to wrap their calls to `transfer` in a `require`.

_example_: [NoRevert.sol](https://github.com/d-xo/weird-erc20/blob/main/src/NoRevert.sol)

### Revert on Large Approvals & Transfers

Some tokens (e.g. `UNI`, `COMP`) revert if the value passed to `approve` or `transfer` is larger than `uint96`.

Both of the above tokens have special case logic in `approve` that sets `allowance` to `type(uint96).max` if the approval amount is `uint256(-1)`, which may cause issues with systems that expect the value passed to `approve` to be reflected in the `allowances` mapping.

_example_: [Uint96.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Uint96.sol)

### Code Injection Via Token Name

Some malicious tokens have been observed to include malicious javascript in their `name` attribute, allowing attackers to extract private keys from users who choose to interact with these tokens via vulnerable frontends.

This has been used to exploit etherdelta users in the wild ([reference](https://hackernoon.com/how-one-hacker-stole-thousands-of-dollars-worth-of-cryptocurrency-with-a-classic-code-injection-a3aba5d2bff0)).

## [Token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md) (Trail of Bits)

{% hint style="info" %}
Last updated **18 June 2021**. Check the [Github repo](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md) for the most up-to-date version of this particular checklist.
{% endhint %}

Follow this checklist when interacting with arbitrary tokens. Make sure you understand the risks associated with each item, and justify any exceptions to these rules.

For convenience, all Slither [utilities](https://github.com/crytic/slither#tools) can be run directly on a token address, such as:

```
slither-check-erc 0xdac17f958d2ee523a2206206994597c13d831ec7 TetherToken
```

To follow this checklist, you'll want to have this output from Slither for the token:

```
- slither-check-erc [target] [contractName] [optional: --erc ERC_NUMBER]
- slither [target] --print human-summary
- slither [target] --print contract-summary
- slither-prop . --contract ContractName # requires configuration, and use of Echidna and Manticore
```

### General considerations

* [ ] &#x20;**The contract has a security review.** Avoid interacting with contracts that lack a security review. Check the length of the assessment (aka “level of effort”), the reputation of the security firm, and the number and severity of the findings.
* [ ] &#x20;**You have contacted the developers.** You may need to alert their team to an incident. Look for appropriate contacts on [blockchain-security-contacts](https://github.com/crytic/blockchain-security-contacts).
* [ ] &#x20;**They have a security mailing list for critical announcements.** Their team should advise users (like you!) when critical issues are found or when upgrades occur.

### ERC conformity

Slither includes a utility, [slither-check-erc](https://github.com/crytic/slither/wiki/ERC-Conformance), that reviews the conformance of a token to many related ERC standards. Use slither-check-erc to review that:

* [ ] &#x20;**Transfer and transferFrom return a boolean.** Several tokens do not return a boolean on these functions. As a result, their calls in the contract might fail.
* [ ] &#x20;**The name, decimals, and symbol functions are present if used.** These functions are optional in the ERC20 standard and might not be present.
* [ ] &#x20;**Decimals returns a uint8.** Several tokens incorrectly return a uint256. If this is the case, ensure the value returned is below 255.
* [ ] &#x20;**The token mitigates the known **[**ERC20 race condition**](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)**.** The ERC20 standard has a known ERC20 race condition that must be mitigated to prevent attackers from stealing tokens.

Slither includes a utility, [slither-prop](https://github.com/crytic/slither/wiki/Property-generation), that generates unit tests and security properties that can discover many common ERC flaws. Use slither-prop to review that:

* [ ] &#x20;**The contract passes all unit tests and security properties from slither-prop.** Run the generated unit tests, then check the properties with [Echidna](https://github.com/crytic/echidna) and [Manticore](https://manticore.readthedocs.io/en/latest/verifier.html).

### ERC extensions risks

Contracts might have different behaviors from their original ERC specification, manually review that:

* [ ] &#x20;**The token is not an ERC777 token and has no external function call in transfer and transferFrom.** External calls in the transfer functions can lead to reentrancies.
* [ ] &#x20;**Transfer and transferFrom should not take a fee.** Deflationary tokens can lead to unexpected behavior.
* [ ] &#x20;**Potential interest earned from the token is taken into account.** Some tokens distribute interest to token holders. This interest might be trapped in the contract if not taken into account.

### Contract composition

* [ ] &#x20;**The contract avoids unneeded complexity.** The token should be a simple contract; a token with complex code requires a higher standard of review. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#human-summary) to identify complex code.
* [ ] &#x20;**The contract uses SafeMath.** Contracts that do not use SafeMath require a higher standard of review. Inspect the contract by hand for SafeMath usage.
* [ ] &#x20;**The contract has only a few non–token-related functions.** Non–token-related functions increase the likelihood of an issue in the contract. Use Slither’s [contract-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to broadly review the code used in the contract.
* [ ] &#x20;**The token only has one address.** Tokens with multiple entry points for balance updates can break internal bookkeeping based on the address (e.g. `balances[token_address][msg.sender]` might not reflect the actual balance).

### Owner privileges

* [ ] &#x20;**The token is not upgradeable.** Upgradeable contracts might change their rules over time. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to determine if the contract is upgradeable.
* [ ] &#x20;**The owner has limited minting capabilities.** Malicious or compromised owners can abuse minting capabilities. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to review minting capabilities, and consider manually reviewing the code.
* [ ] &#x20;**The token is not pausable.** Malicious or compromised owners can trap contracts relying on pausable tokens. Identify pauseable code by hand.
* [ ] &#x20;**The owner cannot blacklist the contract.** Malicious or compromised owners can trap contracts relying on tokens with a blacklist. Identify blacklisting features by hand.
* [ ] &#x20;**The team behind the token is known and can be held responsible for abuse.** Contracts with anonymous development teams, or that reside in legal shelters should require a higher standard of review.

### Token scarcity

Reviews for issues of token scarcity requires manual review. Check for these conditions:

* &#x20;**No user owns most of the supply.** If a few users own most of the tokens, they can influence operations based on the token's repartition.
* &#x20;**The total supply is sufficient.** Tokens with a low total supply can be easily manipulated.
* &#x20;**The tokens are located in more than a few exchanges.** If all the tokens are in one exchange, a compromise of the exchange can compromise the contract relying on the token.
* &#x20;**Users understand the associated risks of large funds or flash loans.** Contracts relying on the token balance must carefully take in consideration attackers with large funds or attacks through flash loans.
* &#x20;**The token does not allow flash minting**. Flash minting can lead to substantial swings in the balance and the total supply, which neccessitate strict and comprehensive overflow checks in the operation of the token.

## [Token Interaction Checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/) (Consensys)

Tokens have long been a part of the history of blockchain and cryptocurrencies. As far back as the early days of Bitcoin, there were plans of creating ‘[colored coins](https://en.bitcoin.it/wiki/Colored\_Coins)’ to extend functionality to new use cases. Projects such as [Mastercoin](https://bitcoinmagazine.com/articles/mastercoin-a-second-generation-protocol-on-the-bitcoin-blockchain-1383603310), later rebranded as [Omni](https://www.omnilayer.org), popped up to fulfill this vision, eventually inspiring Vitalik Buterin to produce the Ethereum whitepaper.

Today’s ecosystem is a highly composable, vast expanse of tokens with a practically endless list of use cases. Although several token standards have been constructed, the very first token standard, [ERC-20](https://eips.ethereum.org/EIPS/eip-20), remains the most used as a result of the high degree of confidence in its security and simplicity. However, with its unparalleled level of security, the standard has its limitations, thereby inviting the creation of new token standards to increase the range of ever-growing use cases.

### Attempted Solutions <a href="attempted-solutions" id="attempted-solutions"></a>

In order to fix ERC-20 limitations, some other token standards were created focused on making small, fundamental changes to the standard. For example, [ERC-621](https://github.com/ethereum/EIPs/pull/621) was created to allow for a variable totalSupply such that tokens can be minted and/or burned. Another significant early token standard was [ERC-721](https://eips.ethereum.org/EIPS/eip-721) which introduced non-fungible tokens (NFTs). Though these standards were different from ERC-20, they all tried to be backward compatible with ERC20, and they didn’t actually reinvent the wheel, instead they simply added features. The main reason for backward compatibility was the fact that many DApps (DeX, etc) already had ERC20 support in place, which required the standard token interface to interact with those DApps, especially the allowance approval process.

{% embed url="https://youtu.be/qDWkQACtc7Q" %}

Seeing as ERC-20 is so limited, [ERC-777](https://eips.ethereum.org/EIPS/eip-777) was created as a sort of ERC-20 2.0. It was crafted to be backwards compatible with ERC-20 while improving user experience and simplifying smart contract development. Among others, the standard included changes to support functionality for simpler transfers and authorization. However, ERC-777 added extra complexity to the token implementation and raised issues with some of the assumptions DApps had regarding how a token should function.

### The Security Implications <a href="the-security-implications" id="the-security-implications"></a>

Introducing features to a highly composable, base-layer contract is far from a trivial matter. When there are hundreds, if not thousands, of different uses for tokens, one small change can introduce many vulnerabilities. Countless exploits have occurred as a result of developers deviating, even only slightly, from the ERC-20 token standard assumptions. Let’s take a look at some past exploits relating to non-standard token contracts.

Some of these issues resulted from the simplest deviation from the standard, like what and how a function returns a value. As an example, \__transferr_\_ing tokens in some ERC20 implementations return _True_ on a successful transfer, while some others don’t. This can easily [trick a DApp](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca) that is expecting a True return on a successful transfer to assume the transfer has failed. Additionally, a fundamental issue was found in the way allowance approval was implemented in the original ERC20 design, in which a malicious approved attacker can try to [spend more than he is allowed on any allowance changes](https://users.encs.concordia.ca/\~clark/papers/2019\_sb\_erc20.pdf).

In June 2020, [two balancer pools were drained](https://medium.com/balancer-protocol/incident-with-non-standard-erc20-deflationary-tokens-95a0f6d46dea) of funds as a result of the contained tokens’ _deflationary_ nature. The affected pools contained STA and STONK respectively, which burn a portion of tokens with every transfer. This feature allowed the attacker to continuously trade the tokens until the supply of the deflationary tokens was reduced enough that the pricing formula had set an enormously high price per token, allowing the attacker to cheaply trade the token for other assets in the pools. This was an issue mainly as Balancer pool implemented an internal accounting for the user balances, assuming that the token behaved as it should; however, deflationary tokens were never thought of in that process. Accounting discrepancy errors such as this are quite common, and awfully tedious to prevent in tokens with a non-fixed supply, particularly if the token balance changes within contracts that it’s deposited to.

Earlier in 2020, [a vulnerability was exploited on an ERC-777 Uniswap pool](https://defirate.com/imbtc-uniswap-hack/). This attack leveraged the ability of ERC-777 tokens to execute arbitrary code to perform a reentrancy attack. As a result, the liquidity pool was drained of about $300k.

It’s no coincidence that both of these exploits occurred in liquidity pools. This is because the level of smart contract composability is generally correlated with the amount of possible attack vectors and the fact that many liquidity pools accept user-inputted tokens. Outside of liquidity pools that may allow for reentrancy and/or accounting exploits, these token standards are relatively safe to use, as long as the DApp developer has thought about different behaviours in the token designs when interacting with their DApp.

This correlation of complexity and vulnerability is not limited to non-standard tokens. In fact, [ERC-20 has several composability gotchas](https://gist.github.com/maurelian/bfb40cd3c0d40fa03a445357f583c86f#erc20) too, even when entirely standard. For example, it’s possible to [front-run changes to ERC-20’s approve() function](https://blog.smartdec.net/erc20-approve-issue-in-simple-words-a41aaf47bca6) to withdraw more tokens than a user intends to allow. Additionally, external calls can result in a [DoS with an unexpected revert](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert), or if the [return value is unchecked](https://swcregistry.io/docs/SWC-104), execution may resume even if an exception is thrown.

### Token Checklist Table

{% hint style="info" %}
Last updated: **1 April 2021**. Check [this gist](https://gist.github.com/shayanb/cd495e23c7cf1a8b269f8ce7fd198538) for the most up to date version of the following table which is updated often by Consensys.\
The rest of this section's text was copied from the original [blog post](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/) published November 2020.
{% endhint %}

|        Token        |                Feature                |                                     Known Vulnerabilities                                     |                                                                                                                   Resources                                                                                                                   |                                             Examples                                            |
| :-----------------: | :-----------------------------------: | :-------------------------------------------------------------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------: |
|        ERC20        |             **Allowance**             |                               Double withdrawal (front-running)                               |                                                         [Resolving the Multiple Withdrawal Attack on ERC20 Tokens](https://users.encs.concordia.ca/\~clark/papers/2019\_sb\_erc20.pdf)                                                        |                                                                                                 |
|                     |             **decimals()**            |                                The decimals can be more than 18                               |                                                                                                                                                                                                                                               | [YamV2](https://etherscan.io/token/0xaba8cac6866b83ae4eec97dd07ed254282f6ad8a)  has 24 decimals |
|                     |                                       |          Not accounting for the tokens that try to prevent multiple withdrawal attack         |                          [Perpetual Protocol Audit issue 3.12](https://github.com/peckshield/publications/blob/f2c00358fd37332fdeffa12355a9a5d4330c9f95/audit\_reports/perpfi\_audit\_report\_2020\_46\_en\_1\_0.pdf)                         |                                                                                                 |
|                     |                                       |                              Unprotected ‍‍‍‍‍‍‍_transferFrom()_                              |                                                                  [Bancor Network Hack 2020 - 1inch](https://medium.com/@1inch.exchange/bancor-network-hack-2020-3c71444fd59d)                                                                 |                                                                                                 |
|                     |           **External Calls**          |                                  Unchecked Call Return Value                                  |                                                                                       [Unchecked call return value](https://swcregistry.io/docs/SWC-104)                                                                                      |                                                                                                 |
|                     |                                       |                                   DoS with unexpected revert                                  |                                                       [DoS with unexpected revert](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#dos-with-unexpected-revert)                                                      |                                                                                                 |
|                     |             **Transfers**             |                              Might return False instead of Revert                             |                                                                                                                                                                                                                                               |                                                                                                 |
|                     |                                       |                                      Missing return value                                     |                                           [Missing return value bug — At least 130 tokens affected](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca)                                          |                                                                                                 |
|                     |            **BalanceOf()**            |                    Internal Accounting discrepancy with the Actual Balance                    |                                                          [aToken Withdrawal Vulnerability](https://medium.com/trustless-fund/atoken-withdrawal-vulnerability-disclosure-5d8eadc64539)                                                         |                                              aToken                                             |
|                     |           **Blacklistable**           |                      Blacklisted addresses cannot receive or send tokens                      |                   [CENTRE appears to have blacklisted an address holding USDC for the first time](https://www.theblockcrypto.com/linked/70850/centre-appears-to-have-blacklisted-an-address-holding-usdc-for-the-first-time)                  |                                         USDC (FiatToken)                                        |
|                     |        **Mintable / Burnable**        |                            TotalSupply can change by trusted actors                           |                                                                                                                                                                                                                                               |                                                                                                 |
|                     |              **Pausable**             |                      All functionalities can be paused by trusted actors                      |                                                                                                                                                                                                                                               |                                                                                                 |
| Deflationary Tokens |      **Take fees from transfers**     |                    Internal Accounting discrepancy with the Actual Balance                    |                                         [Incident with non-standard ERC20 deflationary tokens](https://medium.com/balancer-protocol/incident-with-non-standard-erc20-deflationary-tokens-95a0f6d46dea)                                        |                                            STA, STONK                                           |
| Inflationary Tokens | **AirDrop interest to token holders** |                    Internal Accounting discrepancy with the Actual Balance                    |                                                                                                                                                                                                                                               |                                             Compound                                            |
|       ERC1400       |       **Permissioned Addresses**      |                         Can block transfers from/to specific addresses                        |                                                                                                                                                                                                                                               |                                         Polymath tokens                                         |
|                     |          **Forced Transfers**         |             Trusted actors have the ability to transfer funds however they choose             |                                                                                                                                                                                                                                               |                                                                                                 |
|        ERC777       |         **Callbacks / Hooks**         |                                           Reentrancy                                          | [Uniswap audit](https://medium.com/consensys-diligence/uniswap-audit-b90335ac007), [OpenZeppelin Example Uniswap exploit](https://github.com/OpenZeppelin/exploit-uniswap), [imBTC Uniswap exploit](https://defirate.com/imbtc-uniswap-hack/) |                                             pTokens                                             |
|                     |                                       |                                    Receiver mining GasToken                                   |                                                                                                                                                                                                                                               |                                                                                                 |
|                     |                                       |                                  Receiver blocks the transfer                                 |                                                                                           In case of iterative push transfer can block all transfers                                                                                          |                                                                                                 |
|       ERC1644       |          **Forced Transfers**         |                           Controller has the ability to steal funds                           |                                                                                                                                                                                                                                               |                                                                                                 |
|        ERC621       |       **Control of totalSupply**      |                          totalSupply can be changed by trusted actors                         |                                                                                                                                                                                                                                               |                                                                                                 |
|        ERC884       |         **Cancel and Reissue**        | Token implementers have the ability to cancel an address and move its tokens to a new address |                                                                                                                                                                                                                                               |                                                                                                 |
|                     |            **Whitelisting**           |                        Tokens can only be sent to whitelisted addresses                       |                                                                                                                                                                                                                                               |                                                                                                 |

### Conclusion <a href="conclusion" id="conclusion"></a>

Although tokens are integral to this ecosystem, they are imperfect, and as such, engineers should carefully consider their flaws and features when working with them. As with all smart contract development, your work can carry significant amounts of real world value, so it’s crucial that you proceed carefully. We believe DApp developers need to count in all token behaviours, and as the result code their DApp in a way that cannot be exploited with different token implementations, at least for the well known token behaviours.

## [Token Implementation Best Practice](https://consensys.github.io/smart-contract-best-practices/tokens/) (Consensys)

{% hint style="info" %}
Last checked: **20 August 2021**. Remember to check the [Github page](https://consensys.github.io/smart-contract-best-practices/tokens/) for any updates to this section.
{% endhint %}

Implementing Tokens should comply with other best practices, but also have some unique considerations.

### Comply with the latest standard[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#comply-with-the-latest-standard) <a href="comply-with-the-latest-standard" id="comply-with-the-latest-standard"></a>

Generally speaking, smart contracts of tokens should follow an accepted and stable standard.

Examples of currently accepted standards include:

* [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [EIP721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md) (non-fungible token)
* More at [eips.ethereum.org](https://eips.ethereum.org/erc#final)

### Be aware of front running attacks on EIP-20[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#be-aware-of-front-running-attacks-on-eip-20) <a href="be-aware-of-front-running-attacks-on-eip-20" id="be-aware-of-front-running-attacks-on-eip-20"></a>

The EIP-20 token's `approve()` function creates the potential for an approved spender to spend more than the intended amount. A [front running attack](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#transaction-ordering-dependence-tod-front-running) can be used, enabling an approved spender to call `transferFrom()` both before and after the call to `approve()` is processed. More details are available on the [EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#approve), and in [this document](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA\_jp-RLM/edit).

### Prevent transferring tokens to the 0x0 address[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#prevent-transferring-tokens-to-the-0x0-address) <a href="prevent-transferring-tokens-to-the-0x0-address" id="prevent-transferring-tokens-to-the-0x0-address"></a>

At the time of writing, the "zero" address ([0x0000000000000000000000000000000000000000](https://etherscan.io/address/0x0000000000000000000000000000000000000000)) holds tokens with a value of more than 80$ million.

### Prevent transferring tokens to the contract address[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#prevent-transferring-tokens-to-the-contract-address) <a href="prevent-transferring-tokens-to-the-contract-address" id="prevent-transferring-tokens-to-the-contract-address"></a>

Consider also preventing the transfer of tokens to the same address of the smart contract.

An example of the potential for loss by leaving this open is the [EOS token smart contract](https://etherscan.io/address/0x86fa049857e0209aa7d9e616f7eb3b3b78ecfdb0) where more than 90,000 tokens are stuck at the contract address.

#### Example[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#example) <a href="example" id="example"></a>

An example of implementing both the above recommendations would be to create the following modifier; validating that the "to" address is neither 0x0 nor the smart contract's own address:

```
    modifier validDestination( address to ) {
        require(to != address(0x0));
        require(to != address(this) );
        _;
    }
```

The modifier should then be applied to the "transfer" and "transferFrom" methods:

```
    function transfer(address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... your logic ...)
    }

    function transferFrom(address _from, address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... your logic ...)
    }
```

## Resources

* [Weird ERC20](https://github.com/d-xo/weird-erc20) - repository contains minimal example implementations in Solidity of ERC20 tokens with behavior that may be surprising or unexpected
* [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md) - by Trail of Bits
* [token interaction checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/) - by Consensys
* [Token Implementation Best Practice](https://consensys.github.io/smart-contract-best-practices/tokens/) - by Consensys
* [Inadherence to Standards Vulnerability](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/inadherence-to-standards.md)
* [Binance Isn’t ERC-20](https://blog.goodaudience.com/binance-isnt-erc-20-7645909069a4)
* [Is BNB really an ERC20?](https://coinrivet.com/is-bnb-really-an-erc-20-token/)
