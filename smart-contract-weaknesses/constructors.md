---
description: https://swcregistry.io/docs/SWC-105
---

# Constructors

Constructors are special functions which often perform critical, privileged tasks when initialising contracts. Before solidity `v0.4.22` constructors were defined as functions that had the same name as the contract that contained them. Thus, when a contract name gets changed in development, if the constructor name isn't changed, it becomes a normal, callable function. As you can imagine, this can (and has) lead to some interesting contract hacks.

For further reading, I suggest the reader attempt the [Ethernaught Challenges](https://github.com/OpenZeppelin/ethernaut) (in particular the Fallout level).

Without adequate access controls, bad actors may be able to withdraw some or all Ether from a contract. This can be caused by misnaming a function intended to be a contructor, giving anyone access to re-initialize the contract. To avoid this vulnerability, only allow withdrawals to be triggered by those authorized, or as intended, and name your constructor appropriately.

Before Solidity `0.4.22`, the only way to define a constructor was by creating a function with the contract name. In some cases this was problematic. For example, if a smart contract is re-used with a different name but the constructor function isn't also changed it simply becomes a regular, callable function.

Now with modern versions of Solidity, you can define the constructor with the `constructor` keyword, effectively deprecating this vulnerability. Thus the solution to this problem is simply to use modern Solidity compiler versions.

#### The Vulnerability <a href="#constructors-vuln" id="constructors-vuln"></a>

**If the contract name gets modified, or there is a typo in the constructor's name such that it no longer matches the name of the contract**, the constructor will behave like a normal function. This can lead to dire consequences, especially if the constructor is performing privileged operations. Consider the following contract

```solidity
contract OwnerWallet {
    address public owner;

    //constructor
    function ownerWallet(address _owner) public {
        owner = _owner;
    }

    // fallback. Collect ether.
    function () payable {}

    function withdraw() public {
        require(msg.sender == owner);
        msg.sender.transfer(this.balance);
    }
}
```

This contract collects ether and only allows the owner to withdraw all the ether by calling the `withdraw()` function. The issue arises due to the fact that the constructor is not exactly named after the contract. Specifically, `ownerWallet` is not the same as `OwnerWallet`. Thus, any user can call the `ownerWallet()` function, set themselves as the owner and then take all the ether in the contract by calling `withdraw()`.

#### Preventative Techniques <a href="#constructors-prev" id="constructors-prev"></a>

This issue has been primarily addressed in the Solidity compiler in version `0.4.22`. This version introduced a `constructor` keyword which specifies the constructor, rather than requiring the name of the function to match the contract name. Using this keyword to specify constructors is recommended to prevent naming issues as highlighted above.

#### Real-World Example: Rubixi <a href="#constructors-example" id="constructors-example"></a>

Rubixi ([contract code](https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be#code)) was another pyramid scheme that exhibited this kind of vulnerability. It was originally called `DynamicPyramid` but the contract name was changed before deployment to `Rubixi`. The constructor's name wasn't changed, allowing any user to become the `creator`. Some interesting discussion related to this bug can be found on this [Bitcoin Thread](https://bitcointalk.org/index.php?topic=1400536.60). Ultimately, it allowed users to fight for `creator` status to claim the fees from the pyramid scheme. More detail on this particular bug can be found [here](https://applicature.com/blog/history-of-ethereum-security-vulnerabilities-hacks-and-their-fixes).

## Resources

* [https://swcregistry.io/docs/SWC-105](https://swcregistry.io/docs/SWC-105)
* [https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be#code](https://etherscan.io/address/0xe82719202e5965Cf5D9B6673B7503a3b92DE20be#code)
* [https://swcregistry.io/docs/SWC-118](https://swcregistry.io/docs/SWC-118)
* [https://blog.sigmaprime.io/solidity-security.html#constructors](https://blog.sigmaprime.io/solidity-security.html#constructors)
