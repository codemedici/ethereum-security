# Token Specific Recommendations

In terms of smart contract development, it's important to follow standards. Standards are set to prevent vulnerabilities, and ignoring them can lead to unexpected effects.

Take for example binance's original BNB token. It was marketed as an ERC20 token, but it was later pointed out that it wasn't actually ERC20 compliant for a few reasons:

* [It prevented sending to 0x0](./#prevent-transferring-tokens-to-the-contract-address)
* [It blocked transfers of 0 value](./#revert-on-transfer-to-the-zero-address)
* [It didn't return true or false for success or fail](./#missing-return-values)
* [it is pausable](./#pausable-tokens)

The main cause for concern with this improper implementation is that if it is used with a smart contract that expects an ERC-20 token, it will behave in unexpected ways. It could even get locked in the contract forever.

Although standards aren't always perfect, and may someday become antiquated, they foster the most secure smart contracts.

## Resources

* [Weird ERC20](https://github.com/d-xo/weird-erc20) - repository contains minimal example implementations in Solidity of ERC20 tokens with behavior that may be surprising or unexpected
* [token integration checklist](https://github.com/crytic/building-secure-contracts/blob/master/development-guidelines/token\_integration.md) - by Trail of Bits
* [token interaction checklist](https://consensys.net/diligence/blog/2020/11/token-interaction-checklist/) - by Consensys
* [Token Implementation Best Practice](https://consensys.github.io/smart-contract-best-practices/tokens/) - by Consensys
* [Inadherence to Standards Vulnerability](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/inadherence-to-standards.md)
* [Binance Isn’t ERC-20](https://blog.goodaudience.com/binance-isnt-erc-20-7645909069a4)
* [Is BNB really an ERC20?](https://coinrivet.com/is-bnb-really-an-erc-20-token/)
* [Awesome Buggy ERC20 Tokens](https://github.com/sec-bit/awesome-buggy-erc20-tokens) - List of real world erc20 buggy contracts
