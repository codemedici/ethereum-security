# Block Timestamp vs Block Number

block.timestamp can be manipulated by miners. The only constraint is that a timestamp has to be larger than the previous block's timestamp. Timestamps too far in the future are usually ignored by the network, but the miner usually has a 30-second window within which **to choose** a timestamp. Watch out for the use of `block.timestamp` for anything that is time-sensitive, like auctions or lottery ends. Contracts should proceed without assuming timestamp accuracy.

block.number can also lead to problems. There's no way to safely predict when a specific block will be mined.

As a rule of thumb, the block.timestamp is generally preferred, subject to accommodation of its inaccurate nature.

