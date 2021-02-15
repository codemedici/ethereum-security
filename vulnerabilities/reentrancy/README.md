# Reentrancy

## Reentrancy

## Reentrancy

Re-entrancy happens in single-thread computing environments, when the execution stack jumps or calls subroutines, before returning to the original execution.

This type of attack can occur when a contract sends ether to an unknown address. An attacker can carefully construct a contract at an external address that contains malicious code in the fallback function. Thus, when a contract sends ether to this address, it will invoke the malicious code. Typically the malicious code executes a function on the vulnerable contract,

• Fallback functions [can be called by anyone & execute malicious code](https://hackernoon.com/ethernaut-lvl-1-walkthrough-how-to-abuse-the-fallback-function-118057b68b56)  
• Malicious external contracts [can abuse withdrawals](https://medium.com/coinmonks/ethernaut-lvl-9-king-walkthrough-how-bad-contracts-can-abuse-withdrawals-db12754f359b)

### Example:

1. A smart contract tracks the balance of a number of external addresses and allows users to retrieve funds with its public `withdraw()` function.  
2. A malicious smart contract uses the `withdraw()` function to retrieve its entire balance.  
3. The victim contract executes the `call.value(amount)()` low level function to send the ether to the malicious contract before updating the balance of the malicious contract.  
4. The malicious contract has a payable `fallback()` function that accepts the funds and then calls back into the victim contract's `withdraw()` function.  
5. This second execution triggers a transfer of funds: remember, the balance of the malicious contract still hasn't been updated from the first withdrawal. As a result, the malicious contract successfully withdraws its entire balance a second time.

### Code Example:

The following function contains a function vulnerable to a reentrancy attack. When the low level `call()` function sends ether to the `msg.sender` address, it becomes vulnerable; if the address is a smart contract, the payment will trigger its fallback function with what's left of the transaction gas:

function withdraw\(uint \_amount\) {  
 require\(balances\[msg.sender\] &gt;= \_amount\);  
 msg.sender.call.value\(\_amount\)\(\);  
 balances\[msg.sender\] -= \_amount;  
}

