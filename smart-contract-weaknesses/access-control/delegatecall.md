# delegatecall

State or storage variables (variables that persist over individual transactions) are placed into slots sequentially as they are introduced in the contract. See [Solidity docs](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage) for more details.\
Solidity stores variables to storage or memory, depending on the type. Uninitialized storage pointers will, by default, point to the initial storage position (0) and can be used to alter the stored value.\
**`delegatecall`** preserves contract context. This means that code that is executed via delegatecall will act on the state (i.e., storage) of the calling contract.

`Delegatecall` is a special variant of a message call. It is almost identical to a regular message call except the target address is executed in the context of the calling contract and `msg.sender` and `msg.value` remain the same. Essentially, `delegatecall` delegates other contracts to modify the calling contract's storage.

Since `delegatecall` gives so much control over a contract, it's very important to only use this with trusted contracts such as your own. If the target address comes from user input, be sure to verify that it is a trusted contract.



Consider the library in FibonacciLib.sol, which can generate the Fibonacci sequence and sequences of similar form. ( Note: this code was modified from [https://bit.ly/2MReuii.](https://bit.ly/2MReuii.) )

```
// library contract - calculates Fibonacci-like numbers
contract FibonacciLib {
    // initializing the standard Fibonacci sequence
    uint public start;
    uint public calculatedFibNumber;
    // modify the zeroth number in the sequence
    function setStart(uint _start) public {
        start = _start;
    }
    function setFibonacci(uint n) public {
        calculatedFibNumber = fibonacci(n);
    }
    function fibonacci(uint n) internal returns (uint) {
        if (n == 0) return start;
        else if (n == 1) return start + 1;
        else return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```

Let us now consider a contract that utilizes this library, shown in FibonacciBalance.sol.

```
contract FibonacciBalance {
    address public fibonacciLibrary;
    // the current Fibonacci number to withdraw
    uint public calculatedFibNumber;
    // the starting Fibonacci sequence number
    uint public start = 3;
    uint public withdrawalCounter;
    // the Fibonancci function selector
    bytes4 constant fibSig = bytes4(sha3("setFibonacci(uint256)"));
    // constructor - loads the contract with ether
    constructor(address _fibonacciLibrary) public payable {
        fibonacciLibrary = _fibonacciLibrary;
    }
    function withdraw() {
        withdrawalCounter += 1;
        // calculate the Fibonacci number for the current withdrawal user-
        // this sets calculatedFibNumber
        require(fibonacciLibrary.delegatecall(fibSig, withdrawalCounter));
        msg.sender.transfer(calculatedFibNumber * 1 ether);
    }
    // allow users to call Fibonacci library functions
    function() public {
        require(fibonacciLibrary.delegatecall(msg.data));
    }
}
```



Storage slot\[0] now corresponds to the fibonacciLibrary address, and slot\[1] corresponds to calculatedFibNumber. It is in this incorrect mapping that the vulnerability occurs.\
recall that the start variable in the FibonacciLib contract is located in storage slot\[0], which is the fibonacciLibrary address in the current contract. This means that the function fibonacci will give an unexpected result. This is because it references start (slot\[0]), which in the current calling context is the fibonacciLibrary address (which will often be quite large, when interpreted as a uint). Thus it is likely that the withdraw function will revert, as it will not contain uint(fibonacciLibrary) amount of ether, which is what calculatedFibNumber will return.

Even worse, the FibonacciBalance contract allows users to call all of the fibonacciLibrary functions via the fallback function at line 26. As we discussed earlier, this includes the setStart function. We discussed that this function allows anyone to modify or set storage slot\[0]. In this case, storage slot\[0] is the fibonacciLibrary address. Therefore, an attacker could create a malicious contract, convert the address to a uint (this can be done in Python easily using int('\<address>',16)), and then call setStart(\<attack\_contract\_address\_as\_uint>). This will change fibonacciLibrary to the address of the attack contract.

contract Attack {\
&#x20;   uint storageSlot0; // corresponds to fibonacciLibrary\
&#x20;   uint storageSlot1; // corresponds to calculatedFibNumber

&#x20;   // fallback - this will run if a specified function is not found\
&#x20;   function() public {\
&#x20;       storageSlot1 = 0; // we set calculatedFibNumber to 0, so if withdraw\
&#x20;       // is called we don't send out any ether\
&#x20;       \<attacker\_address>.transfer(this.balance); // we take all the ether\
&#x20;   }\
&#x20;}

## Preventative Techniques

Solidity provides the **`library`** keyword for implementing library contracts ([see the docs](https://solidity.readthedocs.io/en/latest/contracts.html?highlight=library#libraries) for further details). This ensures the library contract is stateless and non-self-destructable. Forcing libraries to be stateless mitigates the complexities of storage context demonstrated in this section. Stateless libraries also prevent attacks wherein attackers modify the state of the library directly in order to affect the contracts that depend on the library’s code.\
As a general rule of thumb, when using `DELEGATECALL` pay careful attention to the possible calling context of both the library contract and the calling contract, and whenever possible build stateless libraries.

## Parity second hack

## Real-World Example: Parity Multisig Wallet (SecondHack)

The Second Parity Multisig Wallet hack is an example of how well-written library code can be exploited if run outside its intended context. There are a number of good explanations of this hack, such as “[Parity Multisig Hacked. Again](https://medium.com/chain-cloud-company-blog/parity-multisig-hack-again-b46771eaa838)” and “[An In-Depth Look at the Parity Multisig Bug](http://hackingdistributed.com/2017/07/22/deep-dive-parity-bug/)”.

To add to these references, let’s explore the contracts that were exploited. The library and wallet contracts [can be found on GitHub](https://github.com/paritytech/parity-ethereum/blob/b640df8fbb964da7538eef268dffc125b081a82f/js/src/contracts/snippets/enhanced-wallet.sol).

The library contract is as follows:

```
contract WalletLibrary is WalletEvents {
  ...
  // throw unless the contract is not yet initialized.
  modifier only_uninitialized { if (m_numOwners > 0) throw; _; }
  // constructor - just pass on the owner array to multiowned and
  // the limit to daylimit
  function initWallet(address[] _owners, uint _required, uint _daylimit)
      only_uninitialized {
    initDaylimit(_daylimit);
    initMultiowned(_owners, _required);
  }
  // kills the contract sending everything to `_to`.
  function kill(address _to) onlymanyowners(sha3(msg.data)) external {
    suicide(_to);
  }
  ...
}
```

And here’s the wallet contract:

```
contract Wallet is WalletEvents {
  ...
  // METHODS
  // gets called when no other function matches
  function() payable {
    // just being sent some cash?
    if (msg.value > 0)
      Deposit(msg.sender, msg.value);
    else if (msg.data.length > 0)
      _walletLibrary.delegatecall(msg.data);
  }
  ...
  // FIELDS
  address constant _walletLibrary =
    0xcafecafecafecafecafecafecafecafecafecafe;
}
```

Notice that the Wallet contract essentially passes all calls to the WalletLibrary contract via a delegate call. The constant `_walletLibrary` address in this code snippet acts as a placeholder for the actually deployed WalletLibrary contract (which was at `0x863DF6BFa4469f3ead0bE8f9F2AAE51c91A907b4`).

The intended operation of these contracts was to have a simple low-cost deployable Wallet contract whose codebase and main functionality were in the WalletLibrary contract. Unfortunately, the WalletLibrary contract is itself a contract and maintains its own state. Can you see why this might be an issue?

It is possible to send calls to the WalletLibrary contract itself. Specifically, the WalletLibrary contract could be initialized and become owned. In fact, a user did this, calling the initWallet function on the WalletLibrary contract and becoming an owner of the library contract. The same user subsequently called the kill function. Because the user was an owner of the library contract, the modifier passed and the library contract self-destructed. As all Wallet contracts in existence refer to this library contract and contain no method to change this reference, all of their functionality, including the ability to withdraw ether, was lost along with the WalletLibrary contract. As a result, all ether in all Parity multisig wallets of this type instantly became lost or permanently unrecoverable.

## Prevent contracts from executing

## Prevent contracts from executing

In some applications, there's a business requirement that other contracts aren't allowed to interact with your own contract and calls should be limited to externally owned accounts (EOA). When that happens there's a widely used function to check whether or not the msg.sender is an address:

function isContract(address account) internal view returns (bool) {\
&#x20;   bytes32 codehash;\
&#x20;   bytes32 accountHash = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;\
&#x20;   assembly { codehash := extcodehash(account) }\
&#x20;   return (codehash != 0x0 && codehash != accountHash);\
}

It's a mistake to rely on these kinds of checks, because the caller might be a constructor, in which case the function will return false, since code is stored on chain only at the end of the execution. Another aspect is that there's no guarantee that the passed address won't become a contract in the future.



## Resources

* [https://blog.sigmaprime.io/solidity-security.html#delegatecall](https://blog.sigmaprime.io/solidity-security.html#delegatecall)
* [https://docs.soliditylang.org/en/latest/contracts.html?highlight=library#libraries%0Ahttps://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage](https://docs.soliditylang.org/en/latest/contracts.html?highlight=library#libraries%0Ahttps://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage)
* [https://swcregistry.io/docs/SWC-112](https://swcregistry.io/docs/SWC-112)
* [https://solidity.readthedocs.io/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries](https://solidity.readthedocs.io/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries)
* [https://blog.sigmaprime.io/solidity-security.html#delegatecall](https://blog.sigmaprime.io/solidity-security.html#delegatecall)
* [https://ethereum.stackexchange.com/questions/3667/difference-between-call-callcode-and-delegatecall](https://ethereum.stackexchange.com/questions/3667/difference-between-call-callcode-and-delegatecall)
