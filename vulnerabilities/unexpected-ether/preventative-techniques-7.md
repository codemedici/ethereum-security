# Preventative Techniques

• Never use the contract's balance for checks, or assume the contract is empty, because it is possible to fund a contract by unusual methods.  
• Ether can be sent to the contract address before deployment, as the address of a smart contract is deterministically generated before it is actually deployed. The address is the result of a keccak hash of the address of the deployer and the nonce of the transaction.  
• Even if no Ether was sent before deployment, Ether can be forced into contracts with selfdestruct or by designating a contract the recipient of mining rewards.  
• Critically examine logic that depends on contract balance.

