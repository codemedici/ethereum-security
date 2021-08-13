# Inadherence to Standards

In terms of smart contract development, it's important to follow standards. Standards are set to prevent vulnerabilities, and ignoring them can lead to unexpected effects.

Take for example binance's original BNB token. It was marketed as an ERC20 token, but it was later pointed out that it wasn't actually ERC20 compliant for a few reasons:

* It prevented sending to 0x0
* It blocked transfers of 0 value
* It didn't return true or false for success or fail

The main cause for concern with this improper implementation is that if it is used with a smart contract that expects an ERC-20 token, it will behave in unexpected ways. It could even get locked in the contract forever.

Although standards aren't always perfect, and may someday become antiquated, they foster the most secure smart contracts.

## Weird ERC20 Tokens

This repository contains minimal example implementations in Solidity of ERC20 tokens with behaviour that may be surprising or unexpected. All the tokens in this repo are based on real tokens, many of which have been used to exploit smart contract systems in the past. It is hoped that these example implementations will be of use to developers and auditors.

The `ERC20` "specification" is so loosely defined that it amounts to little more than an interface declaration, and even the few semantic requirements that are imposed are routinely violated by token developers in the wild.

This makes building smart contracts that interface directly with ERC20 tokens challenging to say the least, and smart contract developers should in general default to the following patterns when interaction with external code is required:

1. A contract level allowlist of known good tokens.
2. Direct interaction with tokens should be performed in dedicated wrapper contracts at the edge of the system. This allows the core to assume a consistent and known good semantics for the behaviour of external assets.

In some cases the above patterns are not practical \(for example in the case of a permissionless AMM, keeping an on chain allowlist would require the introduction of centralized control or a complex governance system\), and in these cases developers must take great care to make these interactions in a highly defensive manner. It should be noted that even if an onchain allowlist is not feasible, an offchain allowlist in the official UI can also protect unsophisticated users from tokens that violate the contracts expectations, while still preserving contract level permissionlessness.

Finally if you are building a token, you are strongly advised to treat the following as a list of behaviours to avoid.

_Additional Resources_

* Trail of Bits [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token_integration.md).
* Consensys Diligence [token integration checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/)

## Tokens

### Reentrant Calls

Some tokens allow reentract calls on transfer \(e.g. `ERC777` tokens\).

This has been exploited in the wild on multiple occasions \(e.g. [imBTC uniswap pool drained](https://defirate.com/imbtc-uniswap-hack/), [lendf.me drained](https://defirate.com/dforce-hack/)\)

_example_: [Reentrant.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Reentrant.sol)

### Missing Return Values

Some tokens do not return a bool \(e.g. `USDT`, `BNB`, `OMG`\) on ERC20 methods. see [here](https://gist.githubusercontent.com/lukas-berlin/f587086f139df93d22987049f3d8ebd2/raw/1f937dc8eb1d6018da59881cbc633e01c0286fb0/Tokens%20missing%20return%20values%20in%20transfer) for a comprehensive \(if somewhat outdated\) list.

Some tokens \(e.g. `BNB`\) may return a `bool` for some methods, but fail to do so for others. This resulted in stuck `BNB` tokens in Uniswap v1 \([details](https://mobile.twitter.com/UniswapProtocol/status/1072286773554876416)\).

Some particulary pathological tokens \(e.g. Tether Gold\) declare a bool return, but then return `false` even when the transfer was successful \([code](https://etherscan.io/address/0x4922a015c4407f87432b179bb209e125432e4a2a#code)\).

A good safe transfer abstraction \([example](https://github.com/Uniswap/uniswap-v2-core/blob/4dd59067c76dea4a0e8e4bfdda41877a6b16dedc/contracts/UniswapV2Pair.sol#L44)\) can help somewhat, but note that the existance of Tether Gold makes it impossible to correctly handle return values for all tokens.

Two example tokens are provided:

* `MissingReturns`: does not return a bool for any erc20 operation
* `ReturnsFalse`: declares a bool return, but then returns false for every erc20 operation

_example_: [MissingReturns.sol](https://github.com/d-xo/weird-erc20/blob/main/src/MissingReturns.sol)  
_example_: [ReturnsFalse.sol](https://github.com/d-xo/weird-erc20/blob/main/src/ReturnsFalse.sol)

### Fee on Transfer

Some tokens take a transfer fee \(e.g. `STA`, `PAXG`\), some do not currently charge a fee but may do so in the future \(e.g. `USDT`, `USDC`\).

The `STA` transfer fee was used to drain $500k from several balancer pools \([more details](https://medium.com/@1inch.exchange/balancer-hack-2020-a8f7131c980e)\).

_example_: [TransferFee.sol](https://github.com/d-xo/weird-erc20/blob/main/src/TransferFee.sol)

### Balance Modifications Outside of Transfers \(rebasing / airdrops\)

Some tokens may make arbitrary balance modifications outside of transfers \(e.g. Ampleforth style rebasing tokens, Compound style airdrops of governance tokens, mintable / burnable tokens\).

Some smart contract systems cache token balances \(e.g. Balancer, Uniswap-V2\), and arbitrary modifications to underlying balances can mean that the contract is operating with outdated information.

In the case of Ampleforth, some Balancer and Uniswap pools are special cased to ensure that the pool's cached balances are atomically updated as part of the rebase prodecure \([details](https://www.ampltalk.org/app/forum/technology-development-17/topic/supported-dex-pools-61/)\).

_example_: TODO: implement a rebasing token

### Upgradable Tokens

Some tokens \(e.g. `USDC`, `USDT`\) are upgradable, allowing the token owners to make arbitrary modifications to the logic of the token at any point in time.

A change to the token semantics can break any smart contract that depends on the past behaviour.

Developers integrating with upgradable tokens should consider introducing logic that will freeze interactions with the token in question if an upgrade is detected. \(e.g. the [`TUSD` adapter](https://github.com/makerdao/dss-deploy/blob/7394f6555daf5747686a1b29b2f46c6b2c64b061/src/join.sol#L321) used by MakerDAO\).

_example_: [Upgradable.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Upgradable.sol)

### Flash Mintable Tokens

There are some proposed token designs that allow for so called "flash minting", which would allow tokens to be minted for the duration of one transaction only, provided they are returned to the token contract by the end of the transaction.

This is similar to a flash loan, but does not require the tokens that are to be lent to exist before the start of the transaction. A token that can be flash minted could potentially have a total supply of max `uint256`.

A proposal to add such a facility to MakerDAO can be found [here](https://forum.makerdao.com/t/mip13c3-sp2-declaration-of-intent-dai-flash-mint-module/3635).

### Tokens with Blocklists

Some tokens \(e.g. `USDC`, `USDT`\) have a contract level admin controlled address blocklist. If an address is blocked, then transfers to and from that address are forbidden.

Malicious or compromised token owners can trap funds in a contract by adding the contract address to the blocklist. This could potentially be the result of regulatory action against the contract itself, against a single user of the contract \(e.g. a Uniswap LP\), or could also be a part of an extortion attempt against users of the blocked contract.

_example_: [BlockList.sol](https://github.com/d-xo/weird-erc20/blob/main/src/BlockList.sol)

### Pausable Tokens

Some tokens can be paused by an admin \(e.g. `BNB`, `ZIL`\).

Similary to the blocklist issue above, an admin controlled pause feature opens users of the token to risk from a malicious or compromised token owner.

_example_: [Pausable.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Pausable.sol)

### Approval Race Protections

Some tokens \(e.g. `USDT`, `KNC`\) do not allow approving an amount `M > 0` when an existing amount `N > 0` is already approved. This is to protect from an ERC20 attack vector described [here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.b32yfk54vyg9).

[This PR](https://github.com/Uniswap/uniswap-v2-periphery/pull/26#issuecomment-647543138) shows some in the wild problems caused by this issue.

_example_: [Approval.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Approval.sol)

### Revert on Zero Value Transfers

Some tokens \(e.g. `LEND`\) revert when transfering a zero value amount.

_example_: [RevertZero.sol](https://github.com/d-xo/weird-erc20/blob/main/src/RevertZero.sol)

### Multiple Token Addresses

Some proxied tokens have multiple addresses. For example `TUSD` has two addresses: `0x8dd5fbCe2F6a956C3022bA3663759011Dd51e73E` and `0x0000000000085d4780B73119b644AE5ecd22b376` \(calling transfer on either affects your balance on both\).

As an example consider the following snippet. `rescueFunds` is intended to allow the contract owner to return non pool tokens that were accidentaly sent to the contract. However, it assumes a single address per token and so would allow the owner to steal all funds in the pool.

```text
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

Some tokens have low decimals \(e.g. `USDC` has 6\). Even more extreme, some tokens like [Gemini USD](https://etherscan.io/token/0x056Fd409E1d7A124BD7017459dFEa2F387b6d5Cd?a=0x5f65f7b609678448494De4C87521CdF6cEf1e932) only have 2 decimals.

This may result in larger than expected precision loss.

_example_: [LowDecimals.sol](https://github.com/d-xo/weird-erc20/blob/main/src/LowDecimals.sol)

### High Decimals

Some tokens have more than 18 decimals \(e.g. `YAM-V2` has 24\).

This may trigger unexpected reverts due to overflow, posing a liveness risk to the contract.

_example_: [HighDecimals.sol](https://github.com/d-xo/weird-erc20/blob/main/src/HighDecimals.sol)

### `transferFrom` with `src == msg.sender`

Some token implementations \(e.g. `DSToken`\) will not attempt to decrease the caller's allowance if the sender is the same as the caller. This gives `transferFrom` the same semantics as `transfer` in this case. Other implementations \(e.g. OpenZeppelin, Uniswap-v2\) will attempt to decrease the caller's allowance from the sender in `transferFrom` even if the caller and the sender are the same address, giving `transfer(dst, amt)` and `transferFrom(address(this), dst, amt)` a different semantics in this case.

_examples_:

Examples of both semantics are provided:

* [ERC20.sol](https://github.com/d-xo/weird-erc20/blob/main/src/ERC20.sol): does not attempt to decrease allowance
* [TransferFromSelf.sol](https://github.com/d-xo/weird-erc20/blob/main/src/TransferFromSelf.sol): always attempts to decrease the allowance

### Non `string` metadata

Some tokens \(e.g. [`MKR`](https://etherscan.io/address/0x9f8f72aa9304c8b593d555f12ef6589cc3a579a2#code)\) have metadata fields \(`name` / `symbol`\) encoded as `bytes32` instead of the `string` prescribed by the ERC20 specification.

This may cause issues when trying to consume metadata from these tokens.

_example_: [Bytes32Metadata.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Bytes32Metadata.sol)

### Revert on Transfer to the Zero Address

Some tokens \(e.g. openzeppelin\) revert when attempting to transfer to `address(0)`.

This may break systems that expect to be able to burn tokens by transfering them to `address(0)`.

_example_: [RevertToZero.sol](https://github.com/d-xo/weird-erc20/blob/main/src/RevertToZero.sol)

### No Revert on Failure

Some tokens do not revert on failure, but instead return `false` \(e.g. [ZRX](https://etherscan.io/address/0xe41d2489571d322189246dafa5ebde1f4699f498#code)\).

While this is technicaly compliant with the ERC20 standard, it goes against common solidity coding practices and may be overlooked by developers who forget to wrap their calls to `transfer` in a `require`.

_example_: [NoRevert.sol](https://github.com/d-xo/weird-erc20/blob/main/src/NoRevert.sol)

### Revert on Large Approvals & Transfers

Some tokens \(e.g. `UNI`, `COMP`\) revert if the value passed to `approve` or `transfer` is larger than `uint96`.

Both of the above tokens have special case logic in `approve` that sets `allowance` to `type(uint96).max` if the approval amount is `uint256(-1)`, which may cause issues with systems that expect the value passed to `approve` to be reflected in the `allowances` mapping.

_example_: [Uint96.sol](https://github.com/d-xo/weird-erc20/blob/main/src/Uint96.sol)

### Code Injection Via Token Name

Some malicious tokens have been observed to include malicious javascript in their `name` attribute, allowing attackers to extract private keys from users who choose to interact with these tokens via vulnerable frontends.

This has been used to exploit etherdelta users in the wild \([reference](https://hackernoon.com/how-one-hacker-stole-thousands-of-dollars-worth-of-cryptocurrency-with-a-classic-code-injection-a3aba5d2bff0)\).

## 

## Token integration checklist

Follow this checklist when interacting with arbitrary tokens. Make sure you understand the risks associated with each item, and justify any exceptions to these rules.

For convenience, all Slither [utilities](https://github.com/crytic/slither#tools) can be run directly on a token address, such as:

```text
slither-check-erc 0xdac17f958d2ee523a2206206994597c13d831ec7 TetherToken
```

To follow this checklist, you'll want to have this output from Slither for the token:

```text
- slither-check-erc [target] [contractName] [optional: --erc ERC_NUMBER]
- slither [target] --print human-summary
- slither [target] --print contract-summary
- slither-prop . --contract ContractName # requires configuration, and use of Echidna and Manticore
```

### General considerations

*  **The contract has a security review.** Avoid interacting with contracts that lack a security review. Check the length of the assessment \(aka “level of effort”\), the reputation of the security firm, and the number and severity of the findings.
*  **You have contacted the developers.** You may need to alert their team to an incident. Look for appropriate contacts on [blockchain-security-contacts](https://github.com/crytic/blockchain-security-contacts).
*  **They have a security mailing list for critical announcements.** Their team should advise users \(like you!\) when critical issues are found or when upgrades occur.

### ERC conformity

Slither includes a utility, [slither-check-erc](https://github.com/crytic/slither/wiki/ERC-Conformance), that reviews the conformance of a token to many related ERC standards. Use slither-check-erc to review that:

*  **Transfer and transferFrom return a boolean.** Several tokens do not return a boolean on these functions. As a result, their calls in the contract might fail.
*  **The name, decimals, and symbol functions are present if used.** These functions are optional in the ERC20 standard and might not be present.
*  **Decimals returns a uint8.** Several tokens incorrectly return a uint256. If this is the case, ensure the value returned is below 255.
*  **The token mitigates the known** [**ERC20 race condition**](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)**.** The ERC20 standard has a known ERC20 race condition that must be mitigated to prevent attackers from stealing tokens.

Slither includes a utility, [slither-prop](https://github.com/crytic/slither/wiki/Property-generation), that generates unit tests and security properties that can discover many common ERC flaws. Use slither-prop to review that:

*  **The contract passes all unit tests and security properties from slither-prop.** Run the generated unit tests, then check the properties with [Echidna](https://github.com/crytic/echidna) and [Manticore](https://manticore.readthedocs.io/en/latest/verifier.html).

### ERC extensions risks

Contracts might have different behaviors from their original ERC specification, manually review that:

*  **The token is not an ERC777 token and has no external function call in transfer and transferFrom.** External calls in the transfer functions can lead to reentrancies.
*  **Transfer and transferFrom should not take a fee.** Deflationary tokens can lead to unexpected behavior.
*  **Potential interest earned from the token is taken into account.** Some tokens distribute interest to token holders. This interest might be trapped in the contract if not taken into account.

### Contract composition

*  **The contract avoids unneeded complexity.** The token should be a simple contract; a token with complex code requires a higher standard of review. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#human-summary) to identify complex code.
*  **The contract uses SafeMath.** Contracts that do not use SafeMath require a higher standard of review. Inspect the contract by hand for SafeMath usage.
*  **The contract has only a few non–token-related functions.** Non–token-related functions increase the likelihood of an issue in the contract. Use Slither’s [contract-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to broadly review the code used in the contract.
*  **The token only has one address.** Tokens with multiple entry points for balance updates can break internal bookkeeping based on the address \(e.g. `balances[token_address][msg.sender]` might not reflect the actual balance\).

### Owner privileges

*  **The token is not upgradeable.** Upgradeable contracts might change their rules over time. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to determine if the contract is upgradeable.
*  **The owner has limited minting capabilities.** Malicious or compromised owners can abuse minting capabilities. Use Slither’s [human-summary printer](https://github.com/crytic/slither/wiki/Printer-documentation#contract-summary) to review minting capabilities, and consider manually reviewing the code.
*  **The token is not pausable.** Malicious or compromised owners can trap contracts relying on pausable tokens. Identify pauseable code by hand.
*  **The owner cannot blacklist the contract.** Malicious or compromised owners can trap contracts relying on tokens with a blacklist. Identify blacklisting features by hand.
*  **The team behind the token is known and can be held responsible for abuse.** Contracts with anonymous development teams, or that reside in legal shelters should require a higher standard of review.

### Token scarcity

Reviews for issues of token scarcity requires manual review. Check for these conditions:

*  **No user owns most of the supply.** If a few users own most of the tokens, they can influence operations based on the token's repartition.
*  **The total supply is sufficient.** Tokens with a low total supply can be easily manipulated.
*  **The tokens are located in more than a few exchanges.** If all the tokens are in one exchange, a compromise of the exchange can compromise the contract relying on the token.
*  **Users understand the associated risks of large funds or flash loans.** Contracts relying on the token balance must carefully take in consideration attackers with large funds or attacks through flash loans.
*  **The token does not allow flash minting**. Flash minting can lead to substantial swings in the balance and the total supply, which neccessitate strict and comprehensive overflow checks in the operation of the token.

## Resources

* [https://finance.yahoo.com/news/bnb-really-erc-20-token-160013314.html](https://finance.yahoo.com/news/bnb-really-erc-20-token-160013314.html)
* [https://blog.goodaudience.com/binance-isnt-erc-20-7645909069a4](https://blog.goodaudience.com/binance-isnt-erc-20-7645909069a4)
* [https://github.com/d-xo/weird-erc20](https://github.com/d-xo/weird-erc20)
* Trail of Bits [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token_integration.md)
* Consensys Diligence [token integration checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/)

