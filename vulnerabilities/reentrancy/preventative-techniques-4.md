# Preventative Techniques

## Reentrancy on a Single Function

// INSECURE  
mapping \(address =&gt; uint\) private userBalances;

function withdrawBalance\(\) public {  
    uint amountToWithdraw = userBalances\[msg.sender\];  
    require\(msg.sender.call.value\(amountToWithdraw\)\(\)\); // At this point, the caller's code is executed, and can call withdrawBalance again  
    userBalances\[msg.sender\] = 0;  
}

### Preventative Techniques

In the example given, the best mitigation method is to [use](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [send\(\)](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [instead of](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value) [call.value\(\)\(\)](https://consensys.github.io/smart-contract-best-practices/recommendations#send-vs-call-value). This will limit any external code from being executed.

However, if you can't remove the external call, the next simplest way to prevent this attack is to make sure you don't call an external function until you've done all the internal work you need to do:

**`userBalances[msg.sender] = 0;`**  
`require(msg.sender.call.value(amountToWithdraw)()); // The user's balance is already 0, so future invocations won't withdraw anything  
}`

Note that if you had another function which called withdrawBalance\(\), it would be potentially subject to the same attack, so you must treat any function which calls an untrusted contract as itself untrusted. See below for further discussion of potential solutions.

## Cross-function Reentrancy

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

