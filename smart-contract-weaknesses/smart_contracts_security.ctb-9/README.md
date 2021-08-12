# Reentrancy

## Introduction

Re-entrancy happens in single-thread computing environments, when the execution stack jumps or calls subroutines, before returning to the original execution.

This type of attack can occur when a contract sends ether to an unknown address. An attacker can carefully construct a contract at an external address that contains malicious code in the fallback function. Thus, when a contract sends ether to this address, it will invoke the malicious code. Typically the malicious code executes a function on the vulnerable contract,

• Fallback functions [can be called by anyone & execute malicious code](https://hackernoon.com/ethernaut-lvl-1-walkthrough-how-to-abuse-the-fallback-function-118057b68b56)  
• Malicious external contracts [can abuse withdrawals](https://medium.com/coinmonks/ethernaut-lvl-9-king-walkthrough-how-bad-contracts-can-abuse-withdrawals-db12754f359b)

### Example

1. A smart contract tracks the balance of a number of external addresses and allows users to retrieve funds with its public `withdraw()` function.  
2. A malicious smart contract uses the `withdraw()` function to retrieve its entire balance.  
3. The victim contract executes the `call.value(amount)()` low level function to send the ether to the malicious contract before updating the balance of the malicious contract.  
4. The malicious contract has a payable `fallback()` function that accepts the funds and then calls back into the victim contract's `withdraw()` function.  
5. This second execution triggers a transfer of funds: remember, the balance of the malicious contract still hasn't been updated from the first withdrawal. As a result, the malicious contract successfully withdraws its entire balance a second time.

### Code Example

The following function contains a function vulnerable to a reentrancy attack. When the low level `call()` function sends ether to the `msg.sender` address, it becomes vulnerable; if the address is a smart contract, the payment will trigger its fallback function with what's left of the transaction gas:

function withdraw\(uint \_amount\) {  
 require\(balances\[msg.sender\] &gt;= \_amount\);  
 msg.sender.call.value\(\_amount\)\(\);  
 balances\[msg.sender\] -= \_amount;  
}

## Resources

#### The DAO

* [https://en.wikipedia.org/wiki/The\_DAO\_\(organization](https://en.wikipedia.org/wiki/The_DAO_%28organization)\) The DAO smart contract Simple DAO code example
* [How Someone Tried to Exploit a Flaw in Our Smart Contract and Steal All of Its Ethe](https://blog.citymayor.co/posts/how-someone-tried-to-exploit-a-flaw-in-our-smart-contract-and-steal-all-of-its-ether/)r
* [http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)

#### Reentrancy

* [Reentrancy Attack On Smart Contracts: How To Identify The Exploitable And An Example Of An Attack Contract](https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4) \[2017\]
* [https://github.com/crytic/not-so-smart-contracts/tree/master/reentrancy](https://github.com/crytic/not-so-smart-contracts/tree/master/reentrancy)
* [https://medium.com/@gus\_tavo\_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4](https://medium.com/@gus_tavo_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4)
* [https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#reentrancy](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#reentrancy)
* [https://blog.sigmaprime.io/solidity-security.html\#reentrancy](https://blog.sigmaprime.io/solidity-security.html#reentrancy) \[2018\]
* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/reentrancy.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/attacks/reentrancy.md)

#### address global functions

* [https://solidity.readthedocs.io/en/latest/units-and-global-variables.html\#address-related](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#address-related)

#### Securing Smart Contracts

* [https://medium.com/loom-network/how-to-secure-your-smart-contracts-6-solidity-vulnerabilities-and-how-to-avoid-them-part-1-c33048d4d17d](https://medium.com/loom-network/how-to-secure-your-smart-contracts-6-solidity-vulnerabilities-and-how-to-avoid-them-part-1-c33048d4d17d)
* [https://solidity.readthedocs.io/en/latest/security-considerations.html\#use-the-checks-effects-interactions-pattern](https://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern)
* [https://consensys.github.io/smart-contract-best-practices/known\_attacks/\#reentrancy](https://consensys.github.io/smart-contract-best-practices/known_attacks/#reentrancy)
* [https://devpost.com/software/ens-login](https://devpost.com/software/ens-login)

#### Ethernaut

* [https://ethernaut.zeppelin.solutions/level/0xf70706db003e94cfe4b5e27ffd891d5c81b39488](https://ethernaut.zeppelin.solutions/level/0xf70706db003e94cfe4b5e27ffd891d5c81b39488)

