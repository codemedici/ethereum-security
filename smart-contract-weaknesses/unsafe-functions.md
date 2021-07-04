# Unsafe Functions

### tx.origin

The address of the originating EOA for the transaction. WARNING: unsafe! This is different from msg.sender which is not necessarily the EOA that initiated the transaction. When checking who is the caller of the contract for authentication purposes always use msg.sender, as an attacker could use a contract to perform a man in the middle attack. The best practice for authentication, in general, is to rely on msg.sender. Be sure to account for the facts above when you encounter tx.origin.

### delegatecall

Whenever a contract calls another contract, the values of all the attrubutes of msg change to reflect the new caller's information. The only exception is delegatecall functionm which runs the code of another contract/library within the original msg context.

### address.send\(amount\)

Similar to transfer, only instead of throwing an excepton, it returns false on error. WARNING: always check the return value of send, as this can cause a number of issues \(i.e.: having a balance updated without executing the payout to a user\).

### address.call.value\(\)\(\)

can also be used to transfer Ether, but the transaction is sent with all the remaining gas value. Due to this fact it should only be used in very specific situations \(i.e.: When you need to call a payable function in another trusted contract\). The return value of .call.value\(\)\(\) should also be verified.

### address.call\(payload\)

Low-level CALL function—can construct an arbitrary message call with a data payload. Returns false on error. WARNING: unsafe—recipient can \(accidentally or maliciously\) use up all your gas, causing your contract to halt with an OOG exception; always check the return value of call.

### address.callcode\(payload\)

Low-level CALLCODE function, like address\(this\).call\(...\) but with this contract’s code replaced with that of address. Returns false on error. WARNING: advanced use only!

### address.delegatecall\(\)

Low-level DELEGATECALL function, like callcode\(...\) but with the full msg context seen by the current contract. Returns false on error. WARNING: advanced use only!

## Preventative Techniques

tx.origin should not be used for authorization in smart contracts. This isn’t to say that the tx.origin variable should never be used. It does have some legitimate use cases in smart contracts. For example, if one wanted to deny external contracts from calling the current contract, one could implement a require of the form require\(tx.origin == msg.sender\). This prevents intermediate contracts being used to call the current contract, limiting the contract to regular codeless addresses.

Keep in mind that transfer / send can fail, and the specificities of each one.

Check for failed sends and the possibility of a denial of service caused by a send to an address that always reverts the transaction.

And, ensure contracts are safe against reentrancy even when using send / transfer, without relying on gas prices.

