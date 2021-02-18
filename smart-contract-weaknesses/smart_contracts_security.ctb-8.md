# Integer Under/Overflow

## Integer Overflows/Underflows

Think about how the uint variable changes state, and who has authority to make such changes. If a uint is made to be less than zero, it will cause an underflow and get set to its maximum value. Keep in mind that the commonly used unsigned integer type, along with all it's variations, truncate the leftmost \(binary\) digit when performing arithmetic operations. This causes the now well-known integer overflow issue.

Be careful with the smaller data-types like `uint8`, `uint16`, `uint24`...etc: they can even more easily hit their maximum value.

Be aware there are around [20 cases for overflow and underflow](https://github.com/ethereum/solidity/issues/796#issuecomment-253578925)!

### Use of

### `var`

###  to declare variables

`var` is deprecated since version `0.4.20`, but is still be available in versions below `0.5.0` for compatibility. The issue with var is that the type of the variable is assumed from it's first assigned value. This can cause a number of issues, as assigning 0 or any low value using var will result in an uint8, which will overflow if it exceeds 255. This causes an infinite loop in the case below:

for \(var i = 0; i &lt; 500; i++\) {  
    // do something  
}

The issue would be resolved with `uint i`.

See what happens when you decrement from 0? The result is the biggest possible value in a uint256. Now just imagine if that happened to a user balance.

contract Overflow {  
    uint public value = 0;

    function increment \(\) public {  
        value++;  
    }

    function decrement \(\) public {  
        value--;  
    }

}  
Check all math operations, especially if a parameter is given by the user. For contracts that perform several arithmetic operations the use of OpenZeppelin's SafeMath4 is encouraged.

## Preventative Techniques

Remediation

One simple solution to mitigate the common mistakes for overflow and underflow is to use SafeMath.sol library for arithmetic functions.  
See [SWC-101](https://swcregistry.io/docs/SWC-101)

