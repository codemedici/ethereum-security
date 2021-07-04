# Pitfalls in Reentrancy Solutions

Since reentrancy can occur across multiple functions, and even multiple contracts, any solution aimed at preventing reentrancy with a single function will not be sufficient.

Instead, we have recommended finishing all internal work \(ie. state changes\) first, and only then calling the external function. This rule, if followed carefully, will allow you to avoid vulnerabilities due to reentrancy. However, you need to not only avoid calling external functions too soon, but also avoid calling functions which call external functions. For example, the following is insecure:

```text
// INSECURE
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

//UNTRUSTED
function withdrawReward(address recipient) public {
    uint amountToWithdraw = rewardsForA[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)()); // REENTRANCY
}

//UNTRUSTED
function getFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // Each recipient should only be able to claim the bonus once

    rewardsForA[recipient] += 100; //
    withdrawReward(recipient); // ! At this point, the caller will be able to execute getFirstWithdrawalBonus again.
    claimedBonus[recipient] = true; // state change
}
```

Even though getFirstWithdrawalBonus\(\) doesn't directly call an external contract, the call in withdrawReward\(\) is enough to make it vulnerable to a reentrancy. You therefore need to treat withdrawReward\(\) as if it were also untrusted.

```text
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function untrustedWithdrawReward(address recipient) public {
    uint amountToWithdraw = rewardsForA[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)());
}

function untrustedGetFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]); // Each recipient should only be able to claim the bonus once

    claimedBonus[recipient] = true; // state change first
    rewardsForA[recipient] += 100;
    untrustedWithdrawReward(recipient); // claimedBonus has been set to true, so reentry is impossible
}
```

In addition to the fix making reentry impossible, untrusted functions have been marked. this same pattern repeats at every level: since untrustedGetFirstWithdrawalBonus\(\) calls untrustedWithdrawReward\(\), which calls an external contract, you must also treat untrustedGetFirstWithdrawalBonus\(\) as insecure.

Another solution often suggested is a mutex. This allows you to "lock" some state so it can only be changed by the owner of the lock. A simple example might look like this:

```text
// Note: This is a rudimentary example, and mutexes are particularly useful where there is substantial logic and/or shared state
mapping (address => uint) private balances;
bool private lockBalances;

function deposit() payable public returns (bool) {
    require(!lockBalances);
    lockBalances = true;
    balances[msg.sender] += msg.value;
    lockBalances = false;
    return true;
}

function withdraw(uint amount) payable public returns (bool) {
    require(!lockBalances && amount > 0 && balances[msg.sender] >= amount);
    lockBalances = true;

    if (msg.sender.call(amount)()) { // Normally insecure, but the mutex saves it
      balances[msg.sender] -= amount;
    }

    lockBalances = false;
    return true;
}
```

If the user tries to call withdraw\(\) again before the first call finishes, the lock will prevent it from having any effect. This can be an effective pattern, but it gets tricky when you have multiple contracts that need to cooperate. The following is insecure:

```text
// INSECURE
contract StateHolder {
    uint private n;
    address private lockHolder;

    function getLock() {
        require(lockHolder == address(0));
        lockHolder = msg.sender;
    }

    function releaseLock() {
        require(msg.sender == lockHolder);
        lockHolder = address(0);
    }

    function set(uint newState) {
        require(msg.sender == lockHolder);
        n = newState;
    }
}
```

An attacker can call getLock\(\), and then never call releaseLock\(\). If they do this, then the contract will be locked forever, and no further changes will be able to be made. If you use mutexes to protect against reentrancy, you will need to carefully ensure that there are no ways for a lock to be claimed and never released. \(There are other potential dangers when programming with mutexes, such as deadlocks and livelocks. You should consult the large amount of literature already written on mutexes, if you decide to go this route.\)

