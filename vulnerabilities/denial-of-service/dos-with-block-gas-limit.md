# DoS with Block Gas Limit

[https://consensys.github.io/smart-contract-best-practices/known\_attacks/\#dos-with-block-gas-limit](https://consensys.github.io/smart-contract-best-practices/known_attacks/#dos-with-block-gas-limit)

## Example: Governmental - Gas limit

The Governmental was a game, deployed in April 2016, in which players could call a payout function that paid taxes to previous players Yes, it was a kind of a Ponzi scheme, with government-inspired dynamics.

The original contract can be inspected here. Below are the lines that caused the problem:  
[https://etherscan.io/address/0xf45717552f12ef7cb65e95476f217ea008167ae3\#code](https://etherscan.io/address/0xf45717552f12ef7cb65e95476f217ea008167ae3#code)

`creditorAddresses = new address[](0);  
creditorAmounts = new uint[](0);`

The code above clears two arrays of previous players. These operations' costs are unbounded, directly proportional to the number of users. In case of Governmental, the payout function ceased working when a large enough number of users signed up.

The best practice in this case would be to avoid unbounded loops as much as possible. If they are needed, account for the block gas limit with a method that ensures processing will be successful in any case.

Later on, someone was able to drain the contract. It’s unclear if that was due to an increased gas limit or fewer players after the incident.

This same contract was later found to be vulnerable to the deep callstack attack whereby one could manipulate the callstack by performing useless operations before calling the target contract, with the goal of halting the function at a determined point, allowing for manipulation of state. This attack is no longer applicable since implementation of EIP 150.  
[https://ethereum.stackexchange.com/questions/9398/how-does-eip-150-change-the-call-depth-attack](https://ethereum.stackexchange.com/questions/9398/how-does-eip-150-change-the-call-depth-attack)

Reddit thread about the vulnerability.

