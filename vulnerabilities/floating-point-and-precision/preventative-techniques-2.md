# Preventative Techniques

## Preventative Techniques

## Preventative Techniques

Keeping the right precision in your smart contracts is very important, especially when dealing with ratios and rates that reflect economic decisions.

You should ensure that any ratios or rates you are using allow for large numerators in fractions. For example, we used the rate tokensPerEth in our example. It would have been better to use weiPerTokens, which would be a large number. To calculate the corresponding number of tokens we could do msg.sender/weiPerTokens. This would give a more precise result.

Another tactic to keep in mind is to be mindful of order of operations. In our example, the calculation to purchase tokens was msg.value/weiPerEth\*tokenPerEth. Notice that the division occurs before the multiplication. \(Solidity, unlike some languages, guarantees to perform operations in the order in which they are written.\) This example would have achieved a greater precision if the calculation performed the multiplication first and then the division; i.e., msg.value\*tokenPerEth/weiPerEth.

Finally, when defining arbitrary precision for numbers it can be a good idea to convert values to higher precision, perform all mathematical operations, then finally convert back down to the precision required for output. Typically uint256s are used \(as they are optimal for gas usage\); these give approximately 60 orders of magnitude in their range, some of which can be dedicated to the precision of mathematical operations. It may be the case that it is better to keep all variables in high precision in Solidity and convert back to lower precisions in external apps \(this is essentially how the decimals variable works in ERC20 token contracts\). To see an example of how this can be done, we recommend looking at [DS-Math](https://github.com/dapphub/ds-math). It uses some funky naming \(“wads” and “rays”\), but the concept is useful.

