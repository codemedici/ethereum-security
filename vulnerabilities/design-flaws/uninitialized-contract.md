# Uninitialized contract

## Parity wallet, 2nd hack

It was a “library”: code that had been abstracted out of the original wallet contract in an attempt to save gas \(ethereum’s unit of “fuel” used to pay for computations\). Their intention was to re-use a single deployment of the repetitive code shared by each instance of the wallet. Storage is one of the most expensive resources in an Ethereum network, and redeploying the same code repeatedly can be considered wasteful in this regard. This valid engineering decision, however, became a fatal flaw in conjunction with some critical oversights.

The library had been based on one of Parity’s earlier wallet implementations. As such, it contained code which enabled the library to be turned into an actual wallet. Code meant for the initialization of wallets was not removed in the process of writing the wallet library. It is believed that devops199—a self-professed non-programmer and newcomer to smart contracts—was trying to understand and perhaps exploit an earlier Parity wallet hack when they made themselves an owner of the library by calling this initialization function. Devops199 then called the library’s kill function \(another function which should have been removed in the development of the library\), likely misunderstanding the implications.

The 587 deployed wallets dependant on this WalletLibrary contract became unusable. All meaningful functionality of the wallet stubs called code in the library, and the immutability of ethereum smart contracts meant there was no way to repair the damage. With no code at the address being delegated to, every function call fails and reverts. 513,774 Ether and many more ERC20 tokens were locked, an estimated $160 million dollars of assets at the time of the incident.

// kills the contract sending everything to \`\_to\`.  
function kill\(address \_to\) onlymanyowners\(sha3\(msg.data\)\) external {  
 suicide\(\_to\);  
}  
The “suicide” in the above function refers to a solidity function which “deletes” the contract from the the blockchain, and forwards its entire ether balance to the passed address. The keyword has since been renamed to the more appropriate “selfdestruct”.

Ironically, the vulnerability devops199 exploited to gain ownership of the library had been reported by Github user “3esmit” \(Ricardo Guilherme Schmidt\) three months prior. It appears the Parity team misunderstood the issue at hand, pushed an ineffectual fix, and closed the issue.

