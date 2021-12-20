---
description: https://swcregistry.io/docs/SWC-115
---

# Tx.origin Authentication

Solidity has a global variable, `tx.origin`, which traverses the entire call stack and contains the address of the account that originally sent the call (or transaction). **Using this variable for authentication in a smart contract leaves the contract vulnerable to a phishing-like attack**.

> **Note**: For further reading, see dbryson's Ethereum [Stack Exchange question](http://bit.ly/2PxU1UM), “[Tx.Origin and Ethereum Oh My!](http://bit.ly/2qm7ocJ)” by Peter Vessenes, and “[Solidity: Tx Origin Attacks](http://bit.ly/2P3KVA4)” by Chris Coverdale.

## The Vulnerability

Contracts that authorize users using the `tx.origin` variable are typically vulnerable to phishing attacks that can trick users into performing authenticated actions on the vulnerable contract.

Consider the simple contract in Phishable.sol.

Example 13. Phishable.sol

```solidity
contract Phishable {

    address public owner;
    
    constructor (address _owner) {
        owner = _owner;
    }
    
    function () external payable {} // collect ether
    
    function withdrawAll(address _recipient) public {
        require(tx.origin == owner);
        _recipient.transfer(this.balance);
    }
}
```

Notice that on line 11 the contract authorizes the withdrawAll function using tx.origin. This contract allows for an attacker to create an attacking contract of the form:

```solidity
import "Phishable.sol";

contract AttackContract {

    Phishable phishableContract;
    address attacker; // The attacker's address to receive funds
    
    constructor (Phishable _phishableContract, address _attackerAddress) {
        phishableContract = _phishableContract;
        attacker = _attackerAddress;
    }
    
    function () payable {
        phishableContract.withdrawAll(attacker);
    }
}
```

**The attacker might disguise this contract as their own private address and socially engineer the victim** (the owner of the Phishable contract) to send some form of transaction to the address—perhaps sending this contract some amount of ether. The victim, unless careful, may not notice that there is code at the attacker’s address, or the attacker might pass it off as being a multisignature wallet or some advanced storage wallet (remember that the source code of public contracts is not available by default).

In any case, **if the victim sends a transaction with enough gas to the AttackContract address, it will invoke the fallback function, which in turn calls the withdrawAll function of the Phishable contract** with the parameter attacker. This will result in the withdrawal of all funds from the Phishable contract to the attacker address. This is because the address that first initialized the call was the victim (i.e., the owner of the Phishable contract). Therefore, tx.origin will be equal to owner and the require on line 11 of the Phishable contract will pass.

## Preventative Techniques

tx.origin should not be used for authorization in smart contracts. This isn’t to say that the tx.origin variable should never be used. It does have some legitimate use cases in smart contracts. For example, if one wanted to deny external contracts from calling the current contract, one could implement a require of the form require(tx.origin == msg.sender). This prevents intermediate contracts being used to call the current contract, limiting the contract to regular codeless addresses.

## Resources

* [https://blog.sigmaprime.io/solidity-security.html#tx-origin](https://blog.sigmaprime.io/solidity-security.html#tx-origin)
* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/authorization-txorigin.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/authorization-txorigin.md)
