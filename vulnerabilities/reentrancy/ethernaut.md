# Ethernaut

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

