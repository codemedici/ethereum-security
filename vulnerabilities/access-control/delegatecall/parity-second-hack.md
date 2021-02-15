# Parity second hack

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

