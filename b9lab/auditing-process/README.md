# Auditing process

The audit process  
The audit process varies greatly from company to company, and between individuals as there is, as yet, no generally-accepted industry standard process. We will walk through several activities that take place before, during and after an audit, beginning with foundational points that are generally agreed upon even as we recognize differences of opinion that exist at the practical level.

Let us begin with the definition of audit by the Merriam-Webster:

audit noun  
au·​dit \| \ ˈȯ-dət &lt;/span&gt;

au·dit \| \ ˈȯ-dət \

Definition of audit \(Entry 1 of 2\)

: a formal examination of an organization's or individual's accounts or financial situation

The audit showed that the company had misled investors.

The final report of an audit.

: a methodical examination and review

an energy audit of the house.

So, as you can see, a software audit fits the second definition, - a methodical examination and review.

Before we delve into what smart contract audits are, it might be good to look into the problem audits are meant to address. Smart contract auditing is a niche information security service. It arose out of necessity.

Smart contracts audits aim to prevent the pain entrepreneurs, developers and users experience when Ethereum contracts are hacked or otherwise fail. Such occurrences were fairly common in the early days of Ethereum, as briefly touched upon previously. Immutability implies that repair may be difficult and costly, or impossible. Immutability implies a requirement for debut production releases to be free of defects, but errors and oversights are likely to remain commonplace as new developers enter the space. The EVM is an unfamiliar platform, blockchain is, at first, an unfamiliar paradigm, and Solidity is, at first, an unfamiliar language. It is not reasonable to expect perfection from new developers.

As we saw in a previous chapter, it is not reasonable to expect perfection from the most seasoned and experienced developers, particularly when complex contracts are involved.

When applications actually started being deployed in the mainnet, circa 2016, only a handful of professionals were qualified to perform preventative code review and the practice was generally unknown. Also, knowledge of common vulnerabilities and attack vectors was largely undeveloped. Many serious problems surfaced in many contracts, often resulting in lost funds or malfunctioning systems. In hindsight, these can be understood as avoidable problems. It is fair to say that many defensive code patterns and best practices were learned post-mortem.

Observing projects getting killed by preventable problems increased general awareness of the importance of preventative quality-assurance. Two approaches shaped the formative Ethereum code security industry.

Bug Bounties  
The first of these is Bug Bounties. Bug bounties are a time-tested approach to reinforcing information security. Organizations such as HackerOne\[^1\], organize bug bounties for corporate clients. Bug Bounties are a way of reaching out to large numbers of qualified developers, to possibly discover critical issues. Importantly, Bug Bounties provide incentives to search and discover. After all, why should an accomplished expert devote valuable time and effort to fortify a piece of software if no incentive exists? Software defects in the open can go undetected for a considerable time if no one is incentivized to find them. As history has shown, the expert with the incentive to discover a vulnerability is often incentivized by an intention to exploit a weakness. Bug Bounties alter the balance of incentives. They offer honest experts a reason to search for problems and report findings before weaknesses devolve into incidents in production.

Soon after, companies began reaching out to successful bug hunters and community members known for dexterity around Solidity and the EVM. This is logical, because even if one intends to run a Bug Bounty campaign, one probably doesn’t want to offer bounty hunters low-hanging fruits. A rational actor will want to establish confidence in the code and a low probability of paying out the reward for a critical bug. Confidence enables the possibility of substantial rewards, and substantial rewards is what summons large contingents of talented developers.

Demand for expert code review gave rise to the present-day smart contract audit industry - a handful of specialized Information Security providers focused on Ethereum.

As mentioned earlier, opinions vary and many issues remain unsettled. We offer an opinion of what a smart contract auditing is:

Auditing a smart contract entails a methodical review of the in-scope source code, in order to provide reasonable assurance that the code behaves as expected, and contains no vulnerabilities.

Reasonable assurance is highlighted because it is impossible to ensure a piece of code contains no bugs. Beware of this when wording reports. Declaring a code base is bug free is irresponsible, and can lead to liability problems. Also keep an eye into how your clients will communicate the audit, and provide clear instructions to them, especially avoiding this type of statement.

It also serves you well to understand why clients engage smart contract auditors. The ostensive reason is to find and correct defects. Naturally, everyone can agree this is a desirable thing to do, even if they disagree on the value of engaging independent third parties to assist with the process.

There is, however, another reason to keep in mind. While largely spoken of in whispers, if at all, smart contract audits help de-risk project-related liabilities. Consider the following.

When a product or service causes accidents, losses, injuries or death it is seldom individual employees who are held accountable. This is because it falls on management to determine if their wares are suitable for purpose. Management may attempt to shift blame to an individual by showing that the individual failed to observe policy or acted incompetently. For example, the company may attempt to avoid blame if accident inspectors blame pilot error or equipment that was certified airworthy by a third party.

In the context of smart contracts, errors and omissions may lead to significant losses \(the unhappy trajectory\). How can management determine that a system is suitable for purpose?

Who understands smart contracts, other than their developers? Who develops quality-assurance processes? How, then, can management perform their duty to determine that a set of contracts is “ready” and “able?”

An emerging solution to this dilemma is one or more smart contract audits by respected specialists. As mentioned earlier, this does not \(cannot\) guarantee that no defects exist, but it can indicate that the company and its management acted responsibly by observing the best practices known at the time. In the abstract the trade can be understood as follows:

The company receives a defense against possible liability. The auditor accepts reputational risk.

For emphasis, auditors should apply care to all forms of communication to avoid a situation in which the auditor appears to take on, perhaps unwittingly, liability for the project.

We will explore bug bounties later in this chapter, and we will go a lot deeper in the auditor career chapter. Let us return to the audit process itself, starting with the first step.

