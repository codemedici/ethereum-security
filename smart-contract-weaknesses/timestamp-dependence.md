# Timestamp Dependence

## Timestamp Dependence¶

Be aware that the timestamp of the block can be manipulated by the miner, and all direct and indirect uses of the timestamp should be considered.

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the [Entropy Illusion](https://app.gitbook.com/s/-MT\_1xrE7o28ej0HwOCr/smart-contract-weaknesses/Vulnerabilities--Timestamp\_Dependence\_HTML/Vulnerabilities--Entropy\_Illusion.html) for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent.

From [Solidity docs](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#block-and-transaction-properties):\
&#x20;Do not rely on `block.timestamp`, `now` and `blockhash` as a source of randomness, unless you know what you are doing. \[..] The current block timestamp must be strictly larger than the timestamp of the last block, but the only guarantee is that it will be somewhere between the timestamps of two consecutive blocks in the canonical chain.

From [Ethereum Stack Exchange question](https://ethereum.stackexchange.com/questions/413/can-a-contract-safely-rely-on-block-timestamp):

&#x20;Block times are subject to the following constraints:

&#x20;• If you stamp your block with a time too far in the future, no one else will build on it (miners will not build on a block timestamped "from the future").\
&#x20;• Your block time cannot be stamped with an earlier time than its parent.\
&#x20;• Difficulty is kept lowest (best deal for miners) by not stamping blocks as earlier than they actually occur.\
&#x20;• using strict equality `==` block.timestamp would not be safe, since a block with that exact timestamp may never get mined. So use `>=` block.timestamp

&#x20;Taken together, these factors suggest that most blocks produced by small hashrate miners who are not trying to manipulate anything will be pretty close to accurate. However, it is trivial for a more powerful miner to manipulate timestamps over short periods, especially if there is something to be gained by doing this.

&#x20;Ultimate, there is no cryptographic way to verify the timestamp itself--only the [ordering of certain cryptographic structures](https://www.youtube.com/watch?v=phXohYF0xGo). Therefore block.timestamp needs to be supplemented with some other strategy in the case of high-value/risk applications.

`block.timestamp` and its alias `now` can be manipulated by miners if they have some incentive to do so. Let’s construct a simple game, shown in [roulette.sol](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#roulette\_security), that would be vulnerable to miner exploitation.

### Block Timestamp vs Block Number

`block.timestamp` can be manipulated by miners. The only constraint is that a timestamp has to be larger than the previous block's timestamp. Timestamps too far in the future are usually ignored by the network, but the miner usually has a 30-second window within which **to choose** a timestamp. Watch out for the use of block.timestamp for anything that is time-sensitive, like auctions or lottery ends. Contracts should proceed without assuming timestamp accuracy.

`block.number` can also lead to problems. There's no way to safely predict when a specific block will be mined.

As a rule of thumb, the `block.timestamp` is generally preferred, subject to accommodation of its inaccurate nature.

### Example

[roulette.sol](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#roulette\_security) from Mastering Ethereum book

contract Roulette {\
&#x20;   uint public pastBlockTime; // forces one bet per block

&#x20;   constructor() public payable {} // initially fund contract

&#x20;   // fallback function used to make a bet\
&#x20;   function () public payable {\
&#x20;       require(msg.value == 10 ether); // must send 10 ether to play\
&#x20;       require(now != pastBlockTime); // only 1 transaction per block\
&#x20;       pastBlockTime = now;\
&#x20;       if(now % 15 == 0) { // winner\
&#x20;           msg.sender.transfer(this.balance);\
&#x20;       }\
&#x20;   }\
}

This contract behaves like a simple lottery. One transaction per block can bet 10 ether for a chance to win the balance of the contract. The assumption here is that \`block.timestamp’s last two digits are uniformly distributed. If that were the case, there would be a 1 in 15 chance of winning this lottery.

However, as we know, miners can adjust the timestamp should they need to. In this particular case, if enough ether pools in the contract, a miner who solves a block is incentivized to choose a timestamp such that `block.timestamp` or `now` modulo 15 is 0. In doing so they may win the ether locked in this contract along with the block reward. As there is only one person allowed to bet per block, this is also vulnerable to front-running attacks (see [Race Conditions/Front Running](https://app.gitbook.com/s/-MT\_1xrE7o28ej0HwOCr/smart-contract-weaknesses/Vulnerabilities--Timestamp\_Dependence\_HTML/Vulnerabilities--Front-Running.html) for further details).

In practice, block timestamps are monotonically increasing and so miners cannot choose arbitrary block timestamps (they must be later than their predecessors). They are also limited to setting block times not too far in the future, as these blocks will likely be rejected by the network (nodes will not validate blocks whose timestamps are in the future).

### Example

From [Consensys recommendations](https://consensys.github.io/smart-contract-best-practices/recommendations/#timestamp-dependence)

Be aware that the timestamp of the block can be manipulated by a miner. Consider this [contract](https://etherscan.io/address/0xcac337492149bdb66b088bf5914bedfbf78ccc18#code):

uint256 constant private salt =  block.timestamp; // ! not random

function random(uint Max) constant private returns (uint256 result){\
&#x20;   //get the best seed for randomness\
&#x20;   uint256 x = salt \* 100/Max;\
&#x20;   uint256 y = salt \* block.number/(salt % 5) ;\
&#x20;   uint256 seed = block.number/3 + (salt % 300) + Last\_Payout + y;\
&#x20;   uint256 h = uint256(block.blockhash(seed));

&#x20;   return uint256((h / x)) % Max + 1; //random number between 1 and Max\
}

When the contract uses the timestamp to seed a random number, the miner can actually post a timestamp within 15 seconds of the block being validated, effectively allowing the miner to precompute an option more favorable to their chances in the lottery. Timestamps are not random and should not be used in that context.

#### The 15-second Rule

The [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) (Ethereum's reference specification) does not specify a constraint on how much blocks can drift in time, but it [does specify](https://ethereum.stackexchange.com/a/5926/46821) that each timestamp should be bigger than the timestamp of its parent. Popular Ethereum protocol implementations Geth and Parity both reject blocks with timestamp more than 15 seconds in future. Therefore, a good rule of thumb in evaluating timestamp usage is:

### Real-World Example: GovernMental

[GovernMental](http://governmental.github.io/GovernMental/), the old Ponzi scheme mentioned above, was also vulnerable to a timestamp-based attack. The contract paid out to the player who was the last player to join (for at least one minute) in a round. Thus, a miner who was a player could adjust the timestamp (to a future time, to make it look like a minute had elapsed) to make it appear that they were the last player to join for over a minute (even though this was not true in reality). More detail on this can be found in the [“History of Ethereum Security Vulnerabilities, Hacks and Their Fixes” post](https://applicature.com/blog/blockchain-technology/history-of-ethereum-security-vulnerabilities-hacks-and-their-fixes) by Tanya Bahrynovska.

### References

[https://](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#timestamp-dependence)[consensys](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#timestamp-dependence)[.github.io/smart-contract-best-practices/known\_attacks/#timestamp-dependence](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#timestamp-dependence)\
[https://github.com/](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#block-timestamp-manipulation)[ethereumbook](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#block-timestamp-manipulation)[/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#block-timestamp-manipulation](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#block-timestamp-manipulation)\
[https://](https://ethereum.stackexchange.com/questions/413/can-a-contract-safely-rely-on-block-timestamp)[ethereum.stackexchange.com](https://ethereum.stackexchange.com/questions/413/can-a-contract-safely-rely-on-block-timestamp)[/questions/413/can-a-contract-safely-rely-on-block-timestamp](https://ethereum.stackexchange.com/questions/413/can-a-contract-safely-rely-on-block-timestamp)

## Preventative Techniques

To avoid miner manipulation in random number generation, there are a few solutions:

* A commitment scheme such as RANDAO, a DAO where the random number is generated by all participants in the DAO.
* External sources via oracles, e.g. Oraclize.
* Using Bitcoin block hashes, as the network is more decentralized and blocks are more expensive to mine.

####

### Timestamp Dependence

There are three main considerations when using a timestamp to execute a critical function in a contract, especially when actions involve fund transfer.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use [block.number](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#block-and-transaction-properties) and an average block time to estimate times; with a `10 second` block time, `1 week` equates to approximately, `60480` blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable easily to manipulate the block number. The [BAT ICO](https://etherscan.io/address/0x0d8775f648430679a709e98d2b0cb6250d2887ef#code) contract employed this strategy.

It is possible to estimate a time delta using the `block.number` property and [average block time](https://etherscan.io/chart/blocktime), however this is not future proof as block times may change (such as [fork reorganisations](https://blog.ethereum.org/2015/08/08/chain-reorganisation-depth-expectations/) and the [difficulty bomb](https://github.com/ethereum/EIPs/issues/649)). In a sale spanning days, the [15-second rule](https://consensys.github.io/smart-contract-best-practices/recommendations/#the-15-second-rule) allows one to achieve a more reliable estimate of time.

**Note**\
If the scale of your time-dependent event can vary by 15 seconds and maintain integrity, it is safe to use a `block.timestamp`.

## Block Timestamp vs Block Number

block.timestamp can be manipulated by miners. The only constraint is that a timestamp has to be larger than the previous block's timestamp. Timestamps too far in the future are usually ignored by the network, but the miner usually has a 30-second window within which **to choose** a timestamp. Watch out for the use of `block.timestamp` for anything that is time-sensitive, like auctions or lottery ends. Contracts should proceed without assuming timestamp accuracy.

block.number can also lead to problems. There's no way to safely predict when a specific block will be mined.

As a rule of thumb, the block.timestamp is generally preferred, subject to accommodation of its inaccurate nature.

## Resources

* [https://blog.sigmaprime.io/solidity-security.html#block-timestamp](https://blog.sigmaprime.io/solidity-security.html#block-timestamp)
* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/timestamp-dependence.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/timestamp-dependence.md)
* [https://swcregistry.io/docs/SWC-120](https://swcregistry.io/docs/SWC-120)
* [https://ethereum.stackexchange.com/questions/419/when-can-blockhash-be-safely-used-for-a-random-number-when-would-it-be-unsafe](https://ethereum.stackexchange.com/questions/419/when-can-blockhash-be-safely-used-for-a-random-number-when-would-it-be-unsafe)
* [https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract)
* [https://github.com/randao/randao](https://github.com/randao/randao)
* [https://etherscan.io/address/0xcac337492149bdb66b088bf5914bedfbf78ccc18](https://etherscan.io/address/0xcac337492149bdb66b088bf5914bedfbf78ccc18)
