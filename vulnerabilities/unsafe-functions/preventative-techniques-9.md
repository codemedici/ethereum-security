# Preventative Techniques

• `tx.origin` should not be used for authorization in smart contracts. This isn’t to say that the `tx.origin` variable should never be used. It does have some legitimate use cases in smart contracts. For example, if one wanted to deny external contracts from calling the current contract, one could implement a require of the form require\(tx.origin == msg.sender\). This prevents intermediate contracts being used to call the current contract, limiting the contract to regular codeless addresses.  
• Keep in mind that `transfer` / `send` can fail, and the specificities of each one.  
• Check for failed sends and the possibility of a denial of service caused by a `send` to an address that always reverts the transaction.  
• And, ensure contracts are safe against reentrancy even when using `send` / `transfer`, without relying on gas prices.

