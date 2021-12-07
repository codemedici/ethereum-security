# Floating Point and Precision

## Floating Point and Precision

As of this writing (v0.4.24), Solidity does not support fixed-point and floating-point numbers. This means that floating-point representations must be constructed with integer types in Solidity. This can lead to errors and vulnerabilities if not implemented correctly.

Note: For further reading, see the [Ethereum Contract Security Techniques and Tips wiki](http://bit.ly/2Ogp2Ia).

### The Vulnerability

As there is no fixed-point type in Solidity, developers are required to implement their own using the standard integer data types. There are a number of pitfalls developers can run into during this process. We will try to highlight some of these in this section.\
Let’s begin with a code example (we’ll ignore over/underflow issues, discussed earlier in this chapter, for simplicity):

```
contract FunWithNumbers {
    uint constant public tokensPerEth = 10;
    uint constant public weiPerEth = 1e18;
    mapping(address => uint) public balances;
    function buyTokens() public payable {
        // convert wei to eth, then multiply by token rate
        uint tokens = msg.value/weiPerEth*tokensPerEth;
        balances[msg.sender] += tokens;
    }
    function sellTokens(uint tokens) public {
        require(balances[msg.sender] >= tokens);
        uint eth = tokens/tokensPerEth;
        balances[msg.sender] -= tokens;
        msg.sender.transfer(eth*weiPerEth);
    }
}
```

This simple token buying/selling contract has some obvious problems. Although the mathematical calculations for buying and selling tokens are correct, the lack of floating-point numbers will give erroneous results. For example, when buying tokens on line 8, if the value is less than 1 ether the initial division will result in 0, leaving the result of the final multiplication as 0 (e.g., 200 wei divided by 1e18 weiPerEth equals 0). Similarly, when selling tokens, any number of tokens less than 10 will also result in 0 ether. In fact, rounding here is always down, so selling 29 tokens will result in 2 ether.

The issue with this contract is that the precision is only to the nearest ether (i.e., 1e18 wei). This can get tricky when dealing with decimals in ERC20 tokens when you need higher precision.

#### Real-World Example: Ethstick

The Ethstick contract does not use extended precision; however, it deals with wei. So, this contract will have issues of rounding, but only at the wei level of precision. It has some more serious flaws, but these relate back to the difficulty in getting entropy on the blockchain (see Entropy Illusion). For a further discussion of the Ethstick contract, we’ll refer you to another post by Peter Vessenes, “Ethereum Contracts Are Going to Be Candy for Hackers”.

## Preventative Techniques

Keeping the right precision in your smart contracts is very important, especially when dealing with ratios and rates that reflect economic decisions.

You should ensure that any ratios or rates you are using allow for large numerators in fractions. For example, we used the rate tokensPerEth in our example. It would have been better to use weiPerTokens, which would be a large number. To calculate the corresponding number of tokens we could do msg.sender/weiPerTokens. This would give a more precise result.

Another tactic to keep in mind is to be mindful of order of operations. In our example, the calculation to purchase tokens was msg.value/weiPerEth\*tokenPerEth. Notice that the division occurs before the multiplication. (Solidity, unlike some languages, guarantees to perform operations in the order in which they are written.) This example would have achieved a greater precision if the calculation performed the multiplication first and then the division; i.e., msg.value\*tokenPerEth/weiPerEth.

Finally, when defining arbitrary precision for numbers it can be a good idea to convert values to higher precision, perform all mathematical operations, then finally convert back down to the precision required for output. Typically uint256s are used (as they are optimal for gas usage); these give approximately 60 orders of magnitude in their range, some of which can be dedicated to the precision of mathematical operations. It may be the case that it is better to keep all variables in high precision in Solidity and convert back to lower precisions in external apps (this is essentially how the decimals variable works in ERC20 token contracts). To see an example of how this can be done, we recommend looking at [DS-Math](https://github.com/dapphub/ds-math). It uses some funky naming (“wads” and “rays”), but the concept is useful.

## Real-world Example

## Real-World Example: Ethstick

The [Ethstick](http://bit.ly/2Qb7PSB) contract does not use extended precision; however, it deals with wei. So, this contract will have issues of rounding, but only at the wei level of precision. It has some more serious flaws, but these relate back to the difficulty in getting entropy on the blockchain (see [Entropy Illusion](https://app.gitbook.com/s/-MT\_1xrE7o28ej0HwOCr/smart-contract-weaknesses/Vulnerabilities--Floating\_Point\_and\_Precision\_HTML/Vulnerabilities--Entropy\_Illusion.html)). For a further discussion of the Ethstick contract, we’ll refer you to another post by Peter Vessenes, “[Ethereum Contracts Are Going to Be Candy for Hackers](http://bit.ly/2SwDnE0)”.

## Resources

* [https://blog.sigmaprime.io/solidity-security.html#precision](https://blog.sigmaprime.io/solidity-security.html#precision)
