# MEV



MEV is a measure of the profit a miner \(or validator, sequencer, etc.\) can make through their ability to arbitrarily include, exclude, or re-order transactions within the blocks they produce.



Let’s take a look at what kinds of actors are in an advantageous position in the blockspace marketplace and have access to more MEV opportunities.

* **Validators \(miners\)** who both \(a\) produce blocks by picking and ordering from a pool of pending transactions and \(b\) propose blocks to add to the Ethereum network. This gives them the ability to extract value by ordering transactions in a way that maximizes the profits that their bots are generating.
* **Arbitrage bot operators**, who devise MEV strategies for frontrunning, backrunning, and sandwich attacks, then making sure their transactions get included before the user they’re trading against by paying a higher gas fee or by sending the transaction to the miner themselves. The miner could be the one to operate a bot, but this is a game of know-how and expertise; some miners won’t be as sophisticated as the quantitative actors extracting MEV.

MEV is commonly misconceived as [Miner Extractable Value](https://arxiv.org/pdf/1904.05234.pdf), but as we discovered above, this is not accurate: MEV is value extracted by those taking advantage of some mechanism can dictate transaction ordering such as **Ethereum’s Priority Gas Auctions \(PGAs\)**. PGAs happen when arbitrage bots often compete against each other by bidding up transaction fees \(gas\) in what we call PGAs, starting a bidding war for the right to capture the arbitrage



### MEV in ETH 2.0

While there are changes in how consensus is reached in eth2, transaction ordering within each eth1 block is the same as it is today, both in the software that orders transactions \(eg. a Ethereum PoW client such as Geth\) and in the p2p network transactions are gossiped through.

This means a technology such as Flashbots’ [MEV-geth](https://docs.flashbots.net/flashbots-core/overview) \(a modified eth1 client software optimized for MEV extraction\) that allows eth1 transaction senders to tip the block proposer \(and transaction orderer\) for their preferred ordering could still exist.

