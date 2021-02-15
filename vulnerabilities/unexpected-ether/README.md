# Unexpected Ether

## Unexpected Ether

## Forcibly Sending Ether to a Contract¶

It is possible to forcibly send Ether to a contract without triggering its fallback function. This is an important consideration when placing important logic in the fallback function or making calculations based on a contract's balance. Take the following example:

`contract Vulnerable {  
 function () payable {  
 revert();  
 }  
 function somethingBad() {  
 require(this.balance > 0);  
 // Do something bad  
 }  
}`  
Contract logic seems to disallow payments to the contract and therefore disallow "something bad" from happening. However, a few methods exist for forcibly sending ether to the contract and therefore making its balance greater than zero.

The selfdestruct contract method allows a user to specify a beneficiary to send any excess ether. selfdestruct does not trigger a contract's fallback function.

Warning

It is also possible to precompute a contract's address and send Ether to that address before deploying the contract.

### Unexpected Ether

Typically, when ether is sent to a contract it must execute either the fallback function or another function defined in the contract. There are two exceptions to this, where ether can exist in a contract without having executed any code. Contracts that rely on code execution for all ether sent to them can be vulnerable to attacks where ether is forcibly sent.

For further reading on this, see “How to Secure Your Smart Contracts” and “Solidity Security Patterns - Forcing Ether to a Contract”.

#### The Vulnerability

A common defensive programming technique that is useful in enforcing correct state transitions or validating operations is invariant checking. This technique involves defining a set of invariants \(metrics or parameters that should not change\) and checking that they remain unchanged after a single \(or many\) operation\(s\). This is typically good design, provided the invariants being checked are in fact invariants. One example of an invariant is the totalSupply of a fixed-issuance ERC20 token. As no function should modify this invariant, one could add a check to the transfer function that ensures the totalSupply remains unmodified, to guarantee the function is working as expected.

In particular, there is one apparent invariant that it may be tempting to use but that can in fact be manipulated by external users \(regardless of the rules put in place in the smart contract\). This is the current ether stored in the contract. Often when developers first learn Solidity they have the misconception that a contract can only accept or obtain ether via payable functions. This misconception can lead to contracts that have false assumptions about the ether balance within them, which can lead to a range of vulnerabilities. The smoking gun for this vulnerability is the \(incorrect\) use of this.balance.

There are two ways in which ether can \(forcibly\) be sent to a contract without using a payable function or executing any code on the contract:

#### Self-destruct/suicide

Any contract is able to implement the selfdestruct function, which removes all bytecode from the contract address and sends all ether stored there to the parameter-specified address. If this specified address is also a contract, no functions \(including the fallback\) get called. Therefore, the selfdestruct function can be used to forcibly send ether to any contract regardless of any code that may exist in the contract, even contracts with no payable functions. This means any attacker can create a contract with a selfdestruct function, send ether to it, call selfdestruct\(target\) and force ether to be sent to a target contract. Martin Swende has an excellent blog post describing some quirks of the self-destruct opcode \(Quirk \#2\) along with an account of how client nodes were checking incorrect invariants, which could have led to a rather catastrophic crash of the Ethereum network.

#### Pre-sent ether

Another way to get ether into a contract is to preload the contract address with ether. Contract addresses are deterministic—in fact, the address is calculated from the Keccak-256 \(commonly synonymous with SHA-3\) hash of the address creating the contract and the transaction nonce that creates the contract. Specifically, it is of the form address = sha3\(rlp.encode\(\[account\_address,transaction\_nonce\]\)\) \(see Adrian Manning’s discussion of “Keyless Ether” for some fun use cases of this\). This means anyone can calculate what a contract’s address will be before it is created and send ether to that address. When the contract is created it will have a nonzero ether balance.

Let’s explore some pitfalls that can arise given this knowledge. Consider the overly simple contract in EtherGame.sol.

Example 5. EtherGame.sol

contract EtherGame {

    uint public payoutMileStone1 = 3 ether;  
    uint public mileStone1Reward = 2 ether;  
    uint public payoutMileStone2 = 5 ether;  
    uint public mileStone2Reward = 3 ether;  
    uint public finalMileStone = 10 ether;  
    uint public finalReward = 5 ether;

    mapping\(address =&gt; uint\) redeemableEther;  
    // Users pay 0.5 ether. At specific milestones, credit their accounts.  
    function play\(\) external payable {  
        require\(msg.value == 0.5 ether\); // each play is 0.5 ether  
        uint currentBalance = this.balance + msg.value;  
        // ensure no players after the game has finished  
        require\(currentBalance &lt;= finalMileStone\);  
        // if at a milestone, credit the player's account  
        if \(currentBalance == payoutMileStone1\) {  
            redeemableEther\[msg.sender\] += mileStone1Reward;  
        }  
        else if \(currentBalance == payoutMileStone2\) {  
            redeemableEther\[msg.sender\] += mileStone2Reward;  
        }  
        else if \(currentBalance == finalMileStone \) {  
            redeemableEther\[msg.sender\] += finalReward;  
        }  
        return;  
    }

    function claimReward\(\) public {  
        // ensure the game is complete  
        require\(this.balance == finalMileStone\);  
        // ensure there is a reward to give  
        require\(redeemableEther\[msg.sender\] &gt; 0\);  
        redeemableEther\[msg.sender\] = 0;  
        msg.sender.transfer\(transferValue\);  
    }  
 }

This contract represents a simple game \(which would naturally involve race conditions\) where players send 0.5 ether to the contract in the hopes of being the player that reaches one of three milestones first. Milestones are denominated in ether. The first to reach the milestone may claim a portion of the ether when the game has ended. The game ends when the final milestone \(10 ether\) is reached; users can then claim their rewards.

The issues with the EtherGame contract come from the poor use of this.balance in both lines 14 \(and by association 16\) and 32. A mischievous attacker could forcibly send a small amount of ether—say, 0.1 ether—via the selfdestruct function \(discussed earlier\) to prevent any future players from reaching a milestone. this.balance will never be a multiple of 0.5 ether thanks to this 0.1 ether contribution, because all legitimate players can only send 0.5-ether increments. This prevents all the if conditions on lines 18, 21, and 24 from being true.

Even worse, a vengeful attacker who missed a milestone could forcibly send 10 ether \(or an equivalent amount of ether that pushes the contract’s balance above the finalMileStone\), which would lock all rewards in the contract forever. This is because the claimReward function will always revert, due to the require on line 32 \(i.e., because this.balance is greater than finalMileStone\).

### Preventative Techniques

This sort of vulnerability typically arises from the misuse of this.balance. Contract logic, when possible, should avoid being dependent on exact values of the balance of the contract, because it can be artificially manipulated. If applying logic based on this.balance, you have to cope with unexpected balances.

If exact values of deposited ether are required, a self-defined variable should be used that is incremented in payable functions, to safely track the deposited ether. This variable will not be influenced by the forced ether sent via a selfdestruct call.

With this in mind, a corrected version of the EtherGame contract could look like:

contract EtherGame {

    uint public payoutMileStone1 = 3 ether;  
    uint public mileStone1Reward = 2 ether;  
    uint public payoutMileStone2 = 5 ether;  
    uint public mileStone2Reward = 3 ether;  
    uint public finalMileStone = 10 ether;  
    uint public finalReward = 5 ether;  
    uint public depositedWei;

    mapping \(address =&gt; uint\) redeemableEther;

    function play\(\) external payable {  
        require\(msg.value == 0.5 ether\);  
        uint currentBalance = depositedWei + msg.value;  
        // ensure no players after the game has finished  
        require\(currentBalance &lt;= finalMileStone\);  
        if \(currentBalance == payoutMileStone1\) {  
            redeemableEther\[msg.sender\] += mileStone1Reward;  
        }  
        else if \(currentBalance == payoutMileStone2\) {  
            redeemableEther\[msg.sender\] += mileStone2Reward;  
        }  
        else if \(currentBalance == finalMileStone \) {  
            redeemableEther\[msg.sender\] += finalReward;  
        }  
        depositedWei += msg.value;  
        return;  
    }

    function claimReward\(\) public {  
        // ensure the game is complete  
        require\(depositedWei == finalMileStone\);  
        // ensure there is a reward to give  
        require\(redeemableEther\[msg.sender\] &gt; 0\);  
        redeemableEther\[msg.sender\] = 0;  
        msg.sender.transfer\(transferValue\);  
    }  
 }

Here, we have created a new variable, depositedWei, which keeps track of the known ether deposited, and it is this variable that we use for our tests. Note that we no longer have any reference to this.balance.

#### Further Examples

A few examples of exploitable contracts were given in the Underhanded Solidity Coding Contest, which also provides extended examples of a number of the pitfalls raised in this section.

