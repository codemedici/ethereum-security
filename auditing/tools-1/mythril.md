# Mythril

Mythril was created by Bernhard Mueller, and is one of the earliest security tools for smart contracts, along with Oyente. It's actively maintained and provides very good, consistent results. They are now launching **MythX**, that includes other tools along with Mythril, but it's geared at development teams. MythX will optimize the time spent during analysis, good for continuous integration, but if you need to completely scrutinize a piece of code the classic Mythril is the way to go.

Mythril performs symbolic execution, SMT solving and taint analysis and requires no setup, a big plus in our book.

So we will use it on the [Useless Ethereum Token smart contract](https://etherscan.io/address/0x27f706edde3aD952EF647Dd67E24e38CD0803DD6).

Notice the warning in red there? This contract was exploited \(as you can see by the interesting looking transactions in the first page of Etherscan\), google it for info on the attack, it was widely communicated.

[Get the contract here](https://git.academy.b9lab.com/SOLIDIFIED-ETH-COURSE-course-repo/module-4-course-tools/blob/master/UselessEthereumToken.sol).  
Open the terminal, and install mythril: `pip3 install mythril==0.20.3`. On Windows you will need to install Python 3 before doing so, please use the version above, to ensure a consistent experience, but feel free to tinker with recent ones too after you successfully run this one.  
Now type **`myth a <path to file>/UselessEthereumToken.sol`**, and make sure you have `solc@0.4` installed. _\(It was 0.5.11 though\)_  
Now go out for a cup of coffee, and come back after 5 to 10 minutes.  
If you run into compiler version problems, take a look at the solc version you have installed, for instructions go to the [installation guidelines](https://solidity.readthedocs.io/en/v0.5.11/installing-solidity.html). Despite the numerous options, installing older versions is always a clunky experience. After you finish the chapter also take a look at `solc-select` \(`npm install --save-dev truffle-solc-select`\), as it will allow you to switch versions easily.

Take a look at the report, if you know the history, you will immediately notice that an integer overflow was exploited in transferFrom, and this is the first notification there. If you are feeling adventurous you can exploit it to this day, though you may have a hard time finding UETs for sale. It's fine exploiting this contract because it's useless, a wreckage sitting on mainnet. We will discuss later on the ethics involved in knowing this all, also refer to the responsible disclosure section in the previous chapter.

A simple tool analysis could've prevented this heist. A number of tools rely on Mythril for analysis, **Karl** \(by Daniel Luca\) being one of them: it monitors a network for vulnerable smart contracts. It's out of scope for this course but we encourage you to give it a go.

Take another look at the contract, and find out exactly where it could fail. Notice that the alerts flagged all instances of possible overflows, but only one could be exploited. There are other related bugs that also allow that to happen.

Another nice trick while using Mythril, is that it will flag any reachable assertion, so if the code you are auditing has assertions testing important invariants it will try all possible combinations and flag if it was ever reached.

We know, very few people use that feature of Solidity, so what you can do when auditing is include assert statements when in doubt if any situation could happen, include an assert with the condition you'd like to check, and run mythril on the file. It is very handy when looking at complex code bases, where assessing if a condition could happen manually can take a long time.

