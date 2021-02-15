# Estimating and price qotes

Although we immediately think of price when we consider quotes, the most important part is estimating the effort it will take to properly secure a set of smart contracts. Even audits that are done for charity \(pro-bono\) should probably be the topic of some type of estimation process. Remember, even if the work is done pro-bono, reputation is at risk so you can’t afford to compromise the process.

The goal of an estimate is to efficiently assess the key factors that tend to affect actual effort / hours. In this context, “efficiently” means to limit oneself to a superficial perusal of the code that won’t take too long. The key is to know what to look for.

Many companies quote based on lines of code. In our experience, line count \(quantity\) is a very poor indicator. Complexity is a better indicator of the actual time required for the audit process. A very large, monolithic smart contract will often be easier to audit than a handful of very small smart contracts that interact in multiple ways.

In our experience, good indicators to note include:

The count of external calls: The number of external calls is a good indicator because they impact the code base complexity in a number of ways. When used within the codebase they increase the number of possible paths. When they are calling external, or untrusted contracts then they impact the time you'll spend to review them considerably. Instead of accounting for a known set of methods and being able to verify what are the possible return values, you will have to account for all possibilities.

Even simple implementations such as an ERC20 token can have an impact on a calling smart contract: USDT and OMG tokens do not return true for successful transfers, for example, and are known for not functioning well \(usually not being accepted at all\) in defi applications. Contracts can be maliciously altered too, so if you are calling untrusted contracts, this has to be accounted for. Recently SpankChain was hacked and the attacker used a rogue ERC20 token implementation. The rogue contract implemented the ERC20 standard interface, but when called for a transfer would re-enter SpankChain's contract.

The count of public / external functions: These are the points of entry. Execution starts here. They will determine the number of paths possible during execution.

Use of Solidity Assembly: Solidity Assembly takes a lot longer to audit. Code is harder to read, several opcodes that are not accessible via Solidity are at the developer’s disposal and none of Solidity’s usual safeguards apply.

Code Smell: \(More on this later\)

Other signs of cleverness, novel solutions: Anything not idiomatic

Your first estimates will probably be the most challenging for you. It will do you well to review estimated time commitments and actual time spent. This will help you improve your instincts and develop indicators of your own.

When working within a company, you will often not be the person to sell or quote the audits. Certain stakeholders may be tempted to price services below their true cost. Mispricing does not alter in any way the cost side of the transaction. Revenue and cost are separate concerns. Responsible parties should be aware of the risk of an audit compromised by unrealistic time constraints and should understand that your main duty is to safeguard the integrity of the process and everyone’s reputation. You can suggest ways to improve the estimation process if it is ineffective, and your helpful feedback to the stakeholders responsible for quoting can assist your organization’s ongoing improvements to its estimating process.

At Solidified, we request a quote from each of three auditors per engagement and then normalize their estimates. Multiple independent opinions at the outset helps with internal skills transfer around estimation itself and helps us reach informed consensus. Quotes include not only the overall price, but also the time each auditor estimates will be required to personally completely scrutinize the code. The quote is ready when each auditor is comfortable with the time allocated for them to produce uncompromised work.

When the Client Proposes the Scope  
We have described scope in the previous sections, including the necessity of a Code Freeze, the need to properly document Intended Behaviour and the actual commit you reviewed in your report.

Sometimes, the client will propose a project scope that doesn’t fit neatly into your process. For example:

To audit only certain files in the overall project  
To audit an amended version of something that was audited before, possibly by someone else.  
There are important considerations to keep in mind in these cases.

Treat all out-of-scope contracts as untrusted contracts. This may be counter-intuitive to the client, because they trust them. Again, your duty is to safeguard the integrity of the process and your audit team’s reputation. If you do not review them, treat them \(and most importantly, calls to them from the in-scope contracts\) as interactions with untrusted contracts.  
Treat all audits as full audits. It is not uncommon that clients request a follow-up audit on code that has previously been audited and changed just a bit. If you were not the first auditor, make sure to quote a full audit of the code. It's acceptable to charge a lower price If you did audit a previous version, but make sure to consider the time you'll need to get refamiliarized with the details.  
Lastly, if you notice important parts of the code base are out of scope, take time to guide your client to understand the risks involved. Remember, clients and readers of your report are depending on you to identify and raise concerns.

Missing Return Value Bug: At Least 130 Tokens Affected  
We Got Spanked: What We Know So Far

