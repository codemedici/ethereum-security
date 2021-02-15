# Floating Point and Precision

## Floating Point and Precision

## Floating Point and Precision

As of this writing \(v0.4.24\), Solidity does not support fixed-point and floating-point numbers. This means that floating-point representations must be constructed with integer types in Solidity. This can lead to errors and vulnerabilities if not implemented correctly.

Note: For further reading, see the [Ethereum Contract Security Techniques and Tips wiki](http://bit.ly/2Ogp2Ia).

### The Vulnerability

As there is no fixed-point type in Solidity, developers are required to implement their own using the standard integer data types. There are a number of pitfalls developers can run into during this process. We will try to highlight some of these in this section.  
Let’s begin with a code example \(we’ll ignore over/underflow issues, discussed earlier in this chapter, for simplicity\):

contract FunWithNumbers {  
    uint constant public tokensPerEth = 10;  
    uint constant public weiPerEth = 1e18;  
    mapping\(address =&gt; uint\) public balances;

    function buyTokens\(\) public payable {  
        // convert wei to eth, then multiply by token rate  
        uint tokens = msg.value/weiPerEth\*tokensPerEth;  
        balances\[msg.sender\] += tokens;  
    }

    function sellTokens\(uint tokens\) public {  
        require\(balances\[msg.sender\] &gt;= tokens\);  
        uint eth = tokens/tokensPerEth;  
        balances\[msg.sender\] -= tokens;  
        msg.sender.transfer\(eth\*weiPerEth\);  
    }  
}

This simple token buying/selling contract has some obvious problems. Although the mathematical calculations for buying and selling tokens are correct, the lack of floating-point numbers will give erroneous results. For example, when buying tokens on line 8, if the value is less than 1 ether the initial division will result in 0, leaving the result of the final multiplication as 0 \(e.g., 200 wei divided by 1e18 weiPerEth equals 0\). Similarly, when selling tokens, any number of tokens less than 10 will also result in 0 ether. In fact, rounding here is always down, so selling 29 tokens will result in 2 ether.

The issue with this contract is that the precision is only to the nearest ether \(i.e., 1e18 wei\). This can get tricky when dealing with decimals in ERC20 tokens when you need higher precision.

Real-World Example: Ethstick  
The Ethstick contract does not use extended precision; however, it deals with wei. So, this contract will have issues of rounding, but only at the wei level of precision. It has some more serious flaws, but these relate back to the difficulty in getting entropy on the blockchain \(see Entropy Illusion\). For a further discussion of the Ethstick contract, we’ll refer you to another post by Peter Vessenes, “Ethereum Contracts Are Going to Be Candy for Hackers”.

