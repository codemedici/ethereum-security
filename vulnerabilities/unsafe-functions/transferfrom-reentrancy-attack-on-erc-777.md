# transferFrom\(\) Reentrancy Attack on ERC-777

## transferFrom\(\) Reentrancy Attack on ERC-777

## transferFrom\(\)

### incomplete compatibility between the lending protocol and the ERC-777 token standard

The condition required to enable this attack is simple to describe:  
• The token makes an external call in the transferFrom function to an attacker-controlled address.

#### What types of tokens enable this attack?

External calls inside transfer/transferFrom function can be called either before or after the token balances were changed. This is important, depending on the mentioned condition, there can be two different ways to steal funds from the exchange. Let’s look at some scenarios where reentrancy can be done:  
If the external call is made in transfer function to the recipient \(or any other account controlled by the attacker\) before the balances were updated.  
If the external call is made in transferFrom function to the token spender \(or any other account controlled by the attacker\) before or after the balances were updated. It’s important to notice that all transferFrom functions are always called with the exchange as the recipient, so callback to the recipient which is common in different ERC20 extensions is not dangerous.  
The first example is quite rare, but the second one can be observed in multiple places:  
Every ERC-777 token should have a callback to the spender before the balances are changed and to the recipient after. That allows everyone to make malicious reentrancy to an exchange contract with ERC-777 token.  
Regulated/controlled tokens typically have an external call to the controller to perform additional validation checks before \(or rarely after\) every transaction. This is done for KYC or other regulatory requirements. A malicious controller contract could be used in such an attack. Although the owner of a security token contract is usually a public entity who is not interested in stealing tokens, this also provides an additional incentive for a hacker to gain access to the owner’s private keys. Additionally, in some regulated tokens, controllers already have the power to transfer any amount of tokens from any address.

### Resources

[https://medium.com/consensys-diligence/uniswap-audit-b90335ac007](https://medium.com/consensys-diligence/uniswap-audit-b90335ac007)  
[https://github.com/ConsenSys/Uniswap-audit-report-2018-12\#31-liquidity-pool-can-be-stolen-in-some-tokens-eg-erc-777-29](https://github.com/ConsenSys/Uniswap-audit-report-2018-12#31-liquidity-pool-can-be-stolen-in-some-tokens-eg-erc-777-29)  
[https://codefi.consensys.net/blog/security-risks-in-ethereum-defi](https://codefi.consensys.net/blog/security-risks-in-ethereum-defi)

### Remediation

We recommend adding a mutex to all trading functions in order to prevent reentrancy. That would slightly increase the gas cost of transactions but will make the system more secure toward tokens with more complex structure.  
ConsenSys Diligence  
ConsenSys Diligence has the mission of solving Ethereum…

