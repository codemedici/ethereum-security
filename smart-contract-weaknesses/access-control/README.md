# Access Control

## Access Control

Access Control issues are common in all programs, not just smart contracts. In fact, it's number 5 on the OWASP top 10. One usually accesses a contract's functionality through its public or external functions. While insecure visibility settings give attackers straightforward ways to access a contract's private values or logic, access control bypasses are sometimes more subtle. These vulnerabilities can occur when contracts use the deprecated tx.origin to validate callers, handle large authorization logic with lengthy require and make reckless use of delegatecall in [proxy libraries](https://blog.openzeppelin.com/proxy-libraries-in-solidity-79fbe4b970fd/) or [proxy contracts](https://blog.indorse.io/ethereum-upgradeable-smart-contract-strategies-456350d0557c).

Loss: estimated at 150,000 ETH \(~30M USD at the time\)

Real World Impact:  
• Parity Multi-sig bug 1  
• Parity Multi-sig bug 2  
• Rubixi

Example:  
A smart contract designates the address which initializes it as the contract's owner. This is a common pattern for granting special privileges such as the ability to withdraw the contract's funds.  
Unfortunately, the initialization function can be called by anyone — even after it has already been called. Allowing anyone to become the owner of the contract and take its funds.  
Code Example:

In the following example, the contract's initialization function sets the caller of the function as its owner. However, the logic is detached from the contract's constructor, and it does not keep track of the fact that it has already been called.

function initContract\(\) public {  
 owner = msg.sender;  
}

In the Parity multi-sig wallet, this initialization function was detached from the wallets themselves and defined in a "library" contract. Users were expected to initialize their own wallet by calling the library's function via a delegateCall. Unfortunately, as in our example, the function did not check if the wallet had already been initialized. Worse, [since the library was a smart contract, anyone could initialize the library itself and call for its destruction](https://github.com/paritytech/parity-ethereum/issues/6995#issuecomment-342409816).

### Resources:

[https://dasp.co/\#item-2](https://dasp.co/#item-2)

## tx.origin authentication

## Tx.Origin Authentication

[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#txorigin-authentication](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#txorigin-authentication)  
Solidity has a global variable, `tx.origin`, which traverses the entire call stack and contains the address of the account that originally sent the call \(or transaction\). Using this variable for authentication in a smart contract leaves the contract vulnerable to a phishing-like attack.  
Note: For further reading, see dbryson’s [Ethereum Stack Exchange questio](http://bit.ly/2PxU1UM)n, “[Tx.Origin and Ethereum Oh My!](http://bit.ly/2qm7ocJ)” by Peter Vessenes, and “[Solidity: Tx Origin Attacks](http://bit.ly/2P3KVA4)” by Chris Coverdale.

### The Vulnerability

Contracts that authorize users using the `tx.origin` variable are typically vulnerable to phishing attacks that can trick users into performing authenticated actions on the vulnerable contract.  
Consider the simple contract in [Phishable.sol](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#phishable_security).

contract Phishable {  
    address public owner;

    constructor \(address \_owner\) {  
        owner = \_owner;  
    }

    function \(\) public payable {} // collect ether

    function withdrawAll\(address \_recipient\) public {  
        require\(tx.origin == owner\);  
        \_recipient.transfer\(this.balance\);  
    }  
}

Notice that on line 11 the contract authorizes the `withdrawAll()` function using `tx.origin`. This contract allows for an attacker to create an attacking contract of the form:

import "Phishable.sol";

contract AttackContract {

    Phishable phishableContract;  
    address attacker; // The attacker's address to receive funds

    constructor \(Phishable \_phishableContract, address \_attackerAddress\) {  
        phishableContract = \_phishableContract;  
        attacker = \_attackerAddress;  
    }

    function \(\) payable {  
        phishableContract.withdrawAll\(attacker\);  
    }  
}

The attacker might disguise this contract as their own private address and socially engineer the victim \(the owner of the Phishable contract\) to send some form of transaction to the address—perhaps sending this contract some amount of ether. The victim, unless careful, may not notice that there is code at the attacker’s address, or the attacker might pass it off as being a multisignature wallet or some advanced storage wallet \(remember that the source code of public contracts is not available by default\).  
In any case, if the victim sends a transaction with enough gas to the AttackContract address, it will invoke the fallback function, which in turn calls the withdrawAll function of the Phishable contract with the parameter attacker. This will result in the withdrawal of all funds from the Phishable contract to the attacker address. This is because the address that first initialized the call was the victim \(i.e., the owner of the Phishable contract\). Therefore, tx.origin will be equal to owner and the require on line 11 of the Phishable contract will pass.

### Preventative Techniques

tx.origin should not be used for authorization in smart contracts. This isn’t to say that the tx.origin variable should never be used. It does have some legitimate use cases in smart contracts. For example, if one wanted to deny external contracts from calling the current contract, one could implement a require of the form require\(tx.origin == msg.sender\). This prevents intermediate contracts being used to call the current contract, limiting the contract to regular codeless addresses.

## Default Visibilities

### Function Visibility

Since pragma `0.5.0`, function visibility must be explicitly marked. In previous versions the visibility modifier in Solidity defaults to public for functions and internal for variables. Public access to functions intended for internal use only can be disastrous, so keep an eye on that when dealing with older compilers.  
Also note that gas cost varies with visibility. In some cases, public functions will spend considerably more gas than an external function would, so always favor external visibility, when possible.  
There are four visibility specifiers, which are described in detail in the [Solidity docs](http://bit.ly/2ABiv7j).  
Example:

contract HashForEther {

    function withdrawWinnings\(\) {  
        // Winner if the last 8 hex characters of the address are 0  
        require\(uint32\(msg.sender\) == 0\);  
        \_sendWinnings\(\);  
     }

     function \_sendWinnings\(\) {  
         msg.sender.transfer\(this.balance\);  
     }  
}

Unfortunately, the visibility of the functions has not been specified. In particular, the `_sendWinnings` function is public \(the default\), and thus any address can call this function to steal the bounty.

### Resources:

[https://programtheblockchain.com/posts/2018/01/02/making-smart-contracts-with-public-variables/](https://programtheblockchain.com/posts/2018/01/02/making-smart-contracts-with-public-variables/)  
[https://medium.com/swlh/ethereum-aint-hiding-your-secrets-703e89088937](https://medium.com/swlh/ethereum-aint-hiding-your-secrets-703e89088937)  
[https://solidity.readthedocs.io/en/develop/miscellaneous.html\#layout-of-state-variables-in-storage](https://solidity.readthedocs.io/en/develop/miscellaneous.html#layout-of-state-variables-in-storage)  
[https://solidity.readthedocs.io/en/develop/contracts.html\#visibility-and-getters](https://solidity.readthedocs.io/en/develop/contracts.html#visibility-and-getters)  
[https://github.com/paritytech/parity-ethereum/issues/6995\#issuecomment-342409816](https://github.com/paritytech/parity-ethereum/issues/6995#issuecomment-342409816)  


## Preventative Techniques

## Preventative Techniques

It is good practice to always specify the visibility of all functions in a contract, even if they are intentionally `public`. Recent versions of solc show a warning for functions that have no explicit visibility set, to encourage this practice.

## Parity Multisig Wallet \(First Hack\)

In the first Parity multisig hack, about $31M worth of Ether was stolen, mostly from three wallets. A good recap of exactly how this was done is given by [Haseeb Qureshi](https://bit.ly/2vHiuJQ).  
Essentially, the multisig wallet is constructed from a base `Wallet` contract, which calls a library contract containing the core functionality \(as described in [Real-World Example: Parity Multisig Wallet \(Second Hack\)](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#multisig_secondhack)\). The library contract contains the code to initialize the wallet, as can be seen from the following snippet:

contract WalletLibrary is WalletEvents {  
  ...  
  // METHODS  
  ...  
  // constructor is given number of sigs required to do protected  
  // "onlymanyowners" transactionsas well as the selection of addresses  
  // capable of confirming them  
  function initMultiowned\(address\[\] \_owners, uint \_required\) {  
    m\_numOwners = \_owners.length + 1;  
    m\_owners\[1\] = uint\(msg.sender\);  
    m\_ownerIndex\[uint\(msg.sender\)\] = 1;  
    for \(uint i = 0; i &lt; \_owners.length; ++i\)  
    {  
      m\_owners\[2 + i\] = uint\(\_owners\[i\]\);  
      m\_ownerIndex\[uint\(\_owners\[i\]\)\] = 2 + i;  
    }  
    m\_required = \_required;  
  }  
  ...  
  // constructor - just pass on the owner array to multiowned and  
  // the limit to daylimit  
  function initWallet\(address\[\] \_owners, uint \_required, uint \_daylimit\) {  
    initDaylimit\(\_daylimit\);  
    initMultiowned\(\_owners, \_required\);  
  }  
}

Note that neither of the functions specifies their visibility, so both default to `public`. The `initWallet` function is called in the wallet’s constructor, and sets the owners for the multisig wallet as can be seen in the `initMultiowned` function. Because these functions were accidentally left `public`, an attacker was able to call these functions on deployed contracts, resetting the ownership to the attacker’s address. Being the owner, the attacker then drained the wallets of all their ether.

### Resources

[https://www.parity.io/the-multi-sig-hack-a-postmortem/](https://www.parity.io/the-multi-sig-hack-a-postmortem/)

## Postmortem

## The Multi-sig Hack: A Postmortem

[https://www.parity.io/the-multi-sig-hack-a-postmortem/](https://www.parity.io/the-multi-sig-hack-a-postmortem/)

On Wednesday 19th July, 2017 a bug found in the multi-signature wallet \("multi-sig"\) code used as part of Parity Wallet software was exploited by parties unknown. As of the time of writing, three wallet accounts holding large balances of ETH have been compromised and the balances moved into accounts held by the attacker. The self-titled "White Hat Group" used the same exploit to secure the other compromised wallets within Ethereum, with the stated intention of returning control to the original owners.

All other functionality of Parity, including all normal, non multi-sig accounts have no known vulnerabilities. To give some perspective: 3 multi-sig wallets were exploited from of a total of 596 vulnerable multi-sig wallets \(the rest were commandeered by the White Hat Group\), which themselves are a tiny fraction of Parity accounts.

The compromised accounts can be viewed on Etherscan. At the time of writing, the thief is attempting to launder the stolen funds through exchanges.

We already disabled the use of the broken code \(it requires use of an on-chain registered resource which we were able to quickly unregister\), which means future multi-sig wallets created in all versions of Parity Wallet have no known exploits. There are phishing attempts afoot; please ensure you download Parity only via our official Github repository, https://github.com/paritytech/parity.

#### WHAT WAS THE NATURE OF THE BUG?

The bug was in a pair of extremely sensitive functions designed to allow the set-up of "multi-sig" wallets in the Parity Wallet software.  
The functions should have been protected in order that they be usable only in one specific circumstance, as the contract was being created. However, they were entirely unguarded, which allowed the attacker to reset the ownership and usage parameters of existing wallets arbitrarily.

#### WAS THE WALLET NOT AUDITED?

The original "Foundation" multi-sig wallet code was created and audited by the Ethereum Foundation's DEV team, Parity and others from the community. It is used extensively and underwent extensive peer review. This body of code continues to have no known security issues.

#### WHAT CODE IS USED IN PARITY WALLET?

The code used in Parity Wallet is a modified form of the original multi-sig wallet code. It was restructured into a lightweight "stub" contract which is deployed to the network every time a wallet is created, together with a much heavier "library" contract, containing the majority of the wallet's logic and which is deployed only once. \(By splitting the code in this way, deploying a wallet is substantially cheaper in terms of gas costs.\)

The bug was introduced during this restructuring.

HOW DID THE BUG ESCAPE NOTICE?  
In general, we treat security and consensus code extremely seriously at Parity.

All code changes undergo at least one peer review which we manage through a largely manual system of tracking and tagging, commonly found in open source projects. Alterations that are understood to involve core code require sign-off by core developers; whereas changes to UI code require sign-off by the UI team. Changes which are understood to be sensitive \(such as with contracts or core consensus code\) are tagged as such and undergo a greater level of review, generally two or more expert reviews.

The restructuring of the original multi-sig wallet contract happened as part of a much larger change, namely the introduction of the Wallet user-interface, a 4000-line, almost entirely Javascript, CSS and HTML alteration. The depth and nature of changes made \(and thus the severity of the potential bug\) was misunderstood by the rest of the team. It was erroneously tagged as a \(purely\) UI change and thus received only one review before merge. A later audit by a Solidity expert also missed the issue.

Though the code was open and public, and thus the bug could have been discovered, reported and fixed before any damage done, there was no incentivisation mechanism to ensure good-natured eyes from the community inspected it.

WHAT WILL PARITY DO TO ENSURE THIS DOESN'T HAPPEN AGAIN?  
While there is no fool-proof means of practically ensuring software contains no bugs, Parity Technologies is committed to minimising the chances that its software contains exploits. In response to the present exploit we will refine our development processes and CI system.

The first and biggest change will be to ensure that any alterations to the codebase that involve live contract code \(which can be generally identified through .sol files\) be reviewed by Solidity experts. At present the multi-sig wallet is the only Solidity code that is user-deployable and in wide use within Parity.

Secondly, we will be revisiting the design of Parity Wallet's Multi-sig at a high level. Part of the cause of this bug, and the reason it went unnoticed, was due to the large amount of complexity within the preexisting and well-audited multi-sig wallet. We will consider adding an additional, extremely simple, contract to sit between the more complex multi-sig and any assets it controls. We have already started work on this and a draft of the idea can be found in our contracts repository, the contract named CoreVeto. It is essentially a very simple veto-able time-lock contract, which gives a last defence against these kinds of black swan events where the unknown unknowns can occur. Had funds been sitting behind this contract rather than in the multi-sig directly, no theft would have been possible.

Thirdly, there is wisdom in the idea that the best disinfectant is direct sunlight. Had there been a greater degree of community ownership over the wallet code, then the greater number of eyes may have seen the bug before anyone with bad intentions had managed to exploit it. To this end we will be rearchitecting the UI into a more general and lightweight platform with the idea that modules are well-isolated and more inviting to the casual developer.

Fourth, some blame for this bug lies with the Solidity language and, in its current incarnation, the difficulty with which one can understand the execution permissions over functions. We will submit some suggestions to the Solidity team that would help to minimise bugs of this kind. We believe one or both of two ideas would help. One would be to change the default access mode of functions to "private", rather than the eminently insecure "public". Another, even safer, change would be to make it illegal to include functions that have neither an explicit access modifier nor a guard modifier. Almost all functions in production Solidity code have one or both. Either of these rules would have resulted in the buggy multi-sig library not compiling. Having written this, I see Emin Gün Sirer has had a similar appraisal.

Going forward, Parity will try to arrange a bug-bounty programme. Unfortunately, since Parity is a small, minimally-funded start-up, we have not the resources to do this alone. Outside of a few tips to the shoe fund, Parity has received no funding whatsoever from any organisations within the Ethereum ecosystem. So we appeal to the community and those that use our software in their well-funded businesses and projects: help us set up a fund to help ensure this doesn't happen again.

## Extropy

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

## Preventative Techniques

## Preventative Techniques

Solidity provides the **`library`** keyword for implementing library contracts \([see the docs](https://solidity.readthedocs.io/en/latest/contracts.html?highlight=library#libraries) for further details\). This ensures the library contract is stateless and non-self-destructable. Forcing libraries to be stateless mitigates the complexities of storage context demonstrated in this section. Stateless libraries also prevent attacks wherein attackers modify the state of the library directly in order to affect the contracts that depend on the library’s code.  
As a general rule of thumb, when using `DELEGATECALL` pay careful attention to the possible calling context of both the library contract and the calling contract, and whenever possible build stateless libraries.

## Parity second hack

## Real-World Example: Parity Multisig Wallet \(SecondHack\)

The Second Parity Multisig Wallet hack is an example of how well-written library code can be exploited if run outside its intended context. There are a number of good explanations of this hack, such as “[Parity Multisig Hacked. Again](https://medium.com/chain-cloud-company-blog/parity-multisig-hack-again-b46771eaa838)” and “[An In-Depth Look at the Parity Multisig Bug](http://hackingdistributed.com/2017/07/22/deep-dive-parity-bug/)”.

To add to these references, let’s explore the contracts that were exploited. The library and wallet contracts [can be found on GitHub](https://github.com/paritytech/parity-ethereum/blob/b640df8fbb964da7538eef268dffc125b081a82f/js/src/contracts/snippets/enhanced-wallet.sol).

The library contract is as follows:

contract WalletLibrary is WalletEvents {

  ...

  // throw unless the contract is not yet initialized.  
  modifier only\_uninitialized { if \(m\_numOwners &gt; 0\) throw; \_; }

  // constructor - just pass on the owner array to multiowned and  
  // the limit to daylimit  
  function initWallet\(address\[\] \_owners, uint \_required, uint \_daylimit\)  
      only\_uninitialized {  
    initDaylimit\(\_daylimit\);  
    initMultiowned\(\_owners, \_required\);  
  }

  // kills the contract sending everything to \`\_to\`.  
  function kill\(address \_to\) onlymanyowners\(sha3\(msg.data\)\) external {  
    suicide\(\_to\);  
  }

  ...

}

And here’s the wallet contract:

contract Wallet is WalletEvents {

  ...

  // METHODS

  // gets called when no other function matches  
  function\(\) payable {  
    // just being sent some cash?  
    if \(msg.value &gt; 0\)  
      Deposit\(msg.sender, msg.value\);  
    else if \(msg.data.length &gt; 0\)  
      \_walletLibrary.delegatecall\(msg.data\);  
  }

  ...

  // FIELDS  
  address constant \_walletLibrary =  
    0xcafecafecafecafecafecafecafecafecafecafe;  
}

Notice that the Wallet contract essentially passes all calls to the WalletLibrary contract via a delegate call. The constant \_walletLibrary address in this code snippet acts as a placeholder for the actually deployed WalletLibrary contract \(which was at 0x863DF6BFa4469f3ead0bE8f9F2AAE51c91A907b4\).

The intended operation of these contracts was to have a simple low-cost deployable Wallet contract whose codebase and main functionality were in the WalletLibrary contract. Unfortunately, the WalletLibrary contract is itself a contract and maintains its own state. Can you see why this might be an issue?

It is possible to send calls to the WalletLibrary contract itself. Specifically, the WalletLibrary contract could be initialized and become owned. In fact, a user did this, calling the initWallet function on the WalletLibrary contract and becoming an owner of the library contract. The same user subsequently called the kill function. Because the user was an owner of the library contract, the modifier passed and the library contract self-destructed. As all Wallet contracts in existence refer to this library contract and contain no method to change this reference, all of their functionality, including the ability to withdraw ether, was lost along with the WalletLibrary contract. As a result, all ether in all Parity multisig wallets of this type instantly became lost or permanently unrecoverable.

## Prevent contracts from executing

## Prevent contracts from executing

In some applications, there's a business requirement that other contracts aren't allowed to interact with your own contract and calls should be limited to externally owned accounts \(EOA\). When that happens there's a widely used function to check whether or not the msg.sender is an address:

function isContract\(address account\) internal view returns \(bool\) {  
    bytes32 codehash;  
    bytes32 accountHash = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;  
    assembly { codehash := extcodehash\(account\) }  
    return \(codehash != 0x0 && codehash != accountHash\);  
}

It's a mistake to rely on these kinds of checks, because the caller might be a constructor, in which case the function will return false, since code is stored on chain only at the end of the execution. Another aspect is that there's no guarantee that the passed address won't become a contract in the future.

