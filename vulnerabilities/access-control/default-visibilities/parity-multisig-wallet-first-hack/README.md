# Parity Multisig Wallet \(First Hack\)

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

## Resources

[https://www.parity.io/the-multi-sig-hack-a-postmortem/](https://www.parity.io/the-multi-sig-hack-a-postmortem/)

