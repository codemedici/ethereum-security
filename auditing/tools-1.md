# Tools

## Tools Lists

### Fuzzers

Fuzzing is the technique of providing random inputs to a program and check for unwanted behavior, like unexpected reverts. It's useful in order to find unhandled edge cases.

The most well know smart contract fuzzers are **Echindia** and **Harvey**.

### Vizualizers

These are the tools that help to comprehend a program by displaying its visual information, like a call graph that shows each of functions executed during a specific call.

A well-known tool for generating good visual representations is **Surya**.

### Static Analysis

These tools will inspect the source code for known vulnerabilities, much like you do manually. Examples are **Security** and **SmartCheck**.

* Securify - [https://securify.chainsecurity.com](https://securify.chainsecurity.com) - works on bytecode leve
* Oyente
* Maian
* Zeus
* MadMax
* MythX Security Tool: [https://mythx.io](https://mythx.io) \(Consensys\)

### Symbolic Execution \(a.k.a. Dynamic Analysis\)

These tools will run the code \(or application, they can be used in "black box" fashion\), with predefined inputs and check results. **Manticore** and **Mythril** are the most prominent ones.

A more complete list of tools can be found at [https://consensys.github.io/smart-contract-best-practices/security\_tools/.](https://consensys.github.io/smart-contract-best-practices/security_tools/)

With that many flavors to choose from, which ones should you pick first? There are no rules about this, you should use the tools that you are comfortable handling, that will help you understand or test the code you have at hand faster. In an audit the most important constraint is time, so you don't want to spend your time only running tools, over time you will develop a sense of how to use the tools in order to get faster to the point where you are comfortable with the code, while leaving time to analyse their reports, and also manually inspect the code.

If you are just starting this journey, there are some things to keep in mind.

Look for the most used tools, as they are usually well maintained, and you'll find better documentation and more places to get help if needed. Generally, visualizers and static analysis tools will require very little \(or no\) setup, fuzzers will require a little more, and symbolic execution can require extensive setup. Some tools will also use more than one method, saving you time and improving results. Keep this in mind, as tools that take long to setup will eat up your time during an audit, but can give you insights others can't.

Another important step will be to analyze the reports of the tools.

Keep in mind that generally security tools are setup in very strict ways, and will default to alerting you at the most remote possibility of there being a problem, which leads to lots of false positives. Usually a report from any of the tools above, except maybe for a well-set-up dynamic analysis tool, will bring you more false positives than bugs. This is also caused by the fact that some of the tools will break the code down into sections and will disregard the rest of it. This can cause an alert for possible overflow in an arithmetic operation, while disregarding a require\(\) just above that would prevent the overflow.

Some examples you've probably seen already are the uint overflow and "gas requirement high" in the static analyzer present in Remix.

If you want to get deeper into tools, [read this paper](https://publik.tuwien.ac.at/files/publik_278277.pdf). It includes a nice list comparing methods used, and vulnerabilities detected.

Now we will get our hands dirty with a couple of tools, to get you started.

Mythril  
Mythril was created by Bernhard Mueller, and is one of the earliest security tools for smart contracts, along with Oyente. It's actively maintained and provides very good, consistent results. They are now launching MythX, that includes other tools along with Mythril, but it's geared at development teams. MythX will optimize the time spent during analysis, good for continuous integration, but if you need to completely scrutinize a piece of code the classic Mythril is the way to go.

Mythril performs symbolic execution, SMT solving and taint analysis and requires no setup, a big plus in our book.

So we will use it on the Useless Ethereum Token smart contract.

Notice the warning in red there? This contract was exploited \(as you can see by the interesting looking transactions in the first page of Etherscan\), google it for info on the attack, it was widely communicated.

Get the contract here.  
Open the terminal, and install mythril: pip3 install mythril==0.20.3. On Windows you will need to install Python 3 before doing so, please use the version above, to ensure a consistent experience, but feel free to tinker with recent ones too after you successfully run this one.  
Now type myth a &lt;path to file&gt;/UselessEthereumToken.sol, and make sure you have solc@0.4 installed.  
Now go out for a cup of coffee, and come back after 5 to 10 minutes.  
If you run into compiler version problems, take a look at the solc version you have installed, for instructions go to the installation guidelines. Despite the numerous options, installing older versions is always a clunky experience. After you finish the chapter also take a look at solc-select, as it will allow you to switch versions easily.

Take a look at the report, if you know the history, you will immediately notice that an integer overflow was exploited in transferFrom, and this is the first notification there. If you are feeling adventurous you can exploit it to this day, though you may have a hard time finding UETs for sale. It's fine exploiting this contract because it's useless, a wreckage sitting on mainnet. We will discuss later on the ethics involved in knowing this all, also refer to the responsible disclosure section in the previous chapter.

A simple tool analysis could've prevented this heist. A number of tools rely on Mythril for analysis, Karl \(by Daniel Luca\) being one of them: it monitors a network for vulnerable smart contracts. It's out of scope for this course but we encourage you to give it a go.

Take another look at the contract, and find out exactly where it could fail. Notice that the alerts flagged all instances of possible overflows, but only one could be exploited. There are other related bugs that also allow that to happen.

Another nice trick while using Mythril, is that it will flag any reachable assertion, so if the code you are auditing has assertions testing important invariants it will try all possible combinations and flag if it was ever reached.

We know, very few people use that feature of Solidity, so what you can do when auditing is include assert statements when in doubt if any situation could happen, include an assert with the condition you'd like to check, and run mythril on the file. It is very handy when looking at complex code bases, where assessing if a condition could happen manually can take a long time.

Slither  
Slither, by Trail of Bits, is a static analyzer that performs amazingly. It's blazingly fast, static analyzers usually are, and will get all the low-hanging fruits for you in a second. Let's try it:

pip3 install slither-analyzer  
slither UselessEthereumToken.sol  
No time for coffee this time. You will notice it didn't bring the alert Mythril did, but it did bring lots of alerts, for older compiler versions, functions that could be external, coding style, and more.

Some of these issues are false positives, such as the strict equality one - it's checking for an uninitialized variable, strictly - so you have to check one by one, do not report an issue based on tool reports only.

Although screening through false positives can get tedious, tools are an effective way of harvesting all low-hanging fruit in the audit. They will help you not need to worry about reporting an older version of Solidity, and focus on business logic errors, and bugs that these tools simply can't find. Though both tools gave different results \(complementing each other nicely\) the more tools you add to your toolbox, the less value you get \(against the time you will spend\), so using a balanced set is advisable.

Use the tools you know and trust, but try a new one every now and then.

You can see the contracts and resulting analyses in this shared repository.

Other tools  
[https://github.com/crytic/slither/issues](https://github.com/crytic/slither/issues)  
[https://www.ethlint.com/](https://www.ethlint.com/)

## Mythril

Mythril  
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

## Slither

Slither  
Slither, by Trail of Bits, is a static analyzer that performs amazingly. It's blazingly fast, static analyzers usually are, and will get all the low-hanging fruits for you in a second. Let's try it:

pip3 install slither-analyzer  
slither UselessEthereumToken.sol  
No time for coffee this time. You will notice it didn't bring the alert Mythril did, but it did bring lots of alerts, for older compiler versions, functions that could be external, coding style, and more.

Some of these issues are false positives, such as the strict equality one - it's checking for an uninitialized variable, strictly - so you have to check one by one, do not report an issue based on tool reports only.

Although screening through false positives can get tedious, tools are an effective way of harvesting all low-hanging fruit in the audit. They will help you not need to worry about reporting an older version of Solidity, and focus on business logic errors, and bugs that these tools simply can't find. Though both tools gave different results \(complementing each other nicely\) the more tools you add to your toolbox, the less value you get \(against the time you will spend\), so using a balanced set is advisable.

Use the tools you know and trust, but try a new one every now and then.

You can see the contracts and resulting analyses in this shared repository.

## Formal Verification

* VerX - Presentation slides - [https://polybox.ethz.ch/index.php/s/Qpx3VZxAnmwyBWl](https://polybox.ethz.ch/index.php/s/Qpx3VZxAnmwyBWl)

