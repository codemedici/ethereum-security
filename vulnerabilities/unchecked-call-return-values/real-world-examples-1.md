# Real-World Examples

## Real-World Examples

## Real-World Example: Etherpot and King of the Ether

[Etherpot](http://bit.ly/2OfHalK) was a smart contract lottery, not too dissimilar to the example contract mentioned earlier. The downfall of this contract was primarily due to incorrect use of block hashes \(only the last 256 block hashes are usable; see Aakil Fernandes’s [post](http://bit.ly/2Jpzf4x) about how Etherpot failed to take account of this correctly\). However, this contract also suffered from an unchecked call value.

Consider the function cash in [lotto.sol: Code snippet](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#lotto_security).

...  
  function cash\(uint roundIndex, uint subpotIndex\){

        var subpotsCount = getSubpotsCount\(roundIndex\);

        if\(subpotIndex&gt;=subpotsCount\)  
            return;

        var decisionBlockNumber = getDecisionBlockNumber\(roundIndex,subpotIndex\);

        if\(decisionBlockNumber&gt;block.number\)  
            return;

        if\(rounds\[roundIndex\].isCashed\[subpotIndex\]\)  
            return;  
        //Subpots can only be cashed once. This is to prevent double payouts

        var winner = calculateWinner\(roundIndex,subpotIndex\);  
        var subpot = getSubpot\(roundIndex\);

        winner.send\(subpot\);

        rounds\[roundIndex\].isCashed\[subpotIndex\] = true;  
        //Mark the round as cashed  
}  
...

Notice that on line 21 the send function’s return value is not checked, and the following line then sets a Boolean indicating that the winner has been sent their funds. This bug can allow a state where the winner does not receive their ether, but the state of the contract can indicate that the winner has already been paid.

A more serious version of this bug occurred in the King of the Ether. An excellent [post-mortem](https://www.kingoftheether.com/postmortem.html) of this contract has been written that details how an unchecked failed send could be used to attack the contract.

### Resources

[https://www.kingoftheether.com/postmortem.html](https://www.kingoftheether.com/postmortem.html)  
[https://aakilfernandes.github.io/blockhashes-are-only-good-for-256-blocks](https://aakilfernandes.github.io/blockhashes-are-only-good-for-256-blocks)

