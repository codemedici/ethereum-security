# Engagement

The Engagement  
We will discuss the technical process in the following modules. Let's first explore high-level processes we know companies utilize so you are prepared to participate in processes used anywhere.

The Solidified Process

Solidified uses a very particular process, that we feel is ideal for auditing smart contracts. All audits include three auditors in the team, with the exception of some very low complexity audits, in which case we allow teams of two.

The auditors are from our community \(we hope to see you there soon!\), and each provides a quote for the engagement, as previously described. When the client accepts the combined quote and engages the audit team, work commences immediately. The three auditors are tasked with independently completing a complete review of the code - as if three independent auditors were hired to perform the same audit.

We schedule a debrief meeting on delivery day. When the meeting starts everyone opens their reports simultaneously. This process creates healthy competition between the team members, which encourages them to present their best performance. Even so, it's not uncommon that a vulnerability will be found by only two out of three. On occasion, a single auditor finds a vulnerability that was missed by the other two. This is, itself, an advantage of layering independent audits, diverse sets of experience, and uniquely personal work processes.

We do not require the auditors to follow a prescribed process. Auditors are encouraged to audit using the tools they know and trust, inspecting code in the ways that best suit them. This process is an expression of the usual layering process used in risk management and information security. When something is important, you create multiple controls around it. But, you do not create unreasonable layers of control over the same asset. You look for variety, ensuring the sum of the mitigated risk is greater than that of the individual controls.

In that debrief meeting, the reports are merged into the final Solidified report that is delivered to the client.

Remediation Period

After the report is delivered the project enters a phase in which the client can report fixes that will be verified and documented by the team. The effectiveness of the fixes is verified by the audit team. This is to confirm that the fixes actually work and, importantly, do not create new issues.

The commits in which each issue was fixed are included, as well as a last-reviewed version both in the summary and in the conclusion of the report. Take a look at our reports.

All of our reports are public, to be used by the community. By publishing the reports we also serve users, empower processes such as Nexus Mutual's smart contract cover, and educate both developers and aspiring security professionals. Everyone wins. Admittedly, clients insist on confidential reports. We don’t work on that basis, but we understand our colleagues who do.

Other Processes  
Each company has their own process. As mentioned earlier, processes vary in this formative specialty. The value of an audit is closely linked to the thoroughness of the process and unwavering integrity. Audit reports are meaningless without an understanding of the process that resulted in the report.

We can describe, in general terms, other processes that are known to be practiced.

Most auditors set up a team, including Senior level members \(and sometimes Junior\) and will have them test part or the whole. Usually, the more experienced team lead will review everything, and will be in charge to make sure nothing is overlooked. A junior auditor position in a reputable company can be a great career path and an opportunity to learn deeply and quickly.

Some companies include automated analysis in their process. And, some companies sell only automated reports \(lol\). Tools are awesome, but your name is on the line so make sure to use the tools for their intended purpose \(assisting you to accomplish the task\) and don't rely on them inappropriately. Remember, an automated tool can raise issues it is programmed to detect but they will, generally, fail to grasp Intended Behavior. When false positives are raised it falls on an expert human whose reputation is on the line to determine what is potentially serious and what can be safely dismissed.

Lastly, there are several individuals who provide audit services in the larger Ethereum community, some of them very well regarded.

After the Audit  
We encourage clients to proceed to a bug bounty with significant rewards, as another way to layer mitigation of the risk of bugs and their impact. Some in our industry do not believe bug bounties are effective in securing smart contracts. Our view is a little more nuanced.

A bug bounty is not a replacement for a meticulous process backed by individuals and an organization with “skin in the game.”  
The effort the community invests in a bug search is driven largely by the size of the bounties. Trivial rewards are unlikely to solicit concerted efforts from strong developers.  
Every live contract is a de facto bug bounty. The main participants operate with inhibited ethical controls, but their skill level should not be underestimated. The incentives correlate with the value entrusted to the contracts.  
It should not be worrisome to post large bounties that will incentivize the ethical community to invest considerable effort in a further search for possible problems. After all, unclaimed bounties cost nothing. Any worry that remains belies uncertainty about the code and the audit process\(es\) performed. If such worries are significant, that is the conscience’s way of saying release into the real world where other people’s money will be at risk may be an irresponsible thing to do.  
In bug bounties, the hunters tend to look for critical bugs, but report whatever they see along the way. They tend to not look over the whole codebase, but they spend time in areas that appear to be high-risk. In combination with an audit, the entire code base is secured by an audit, and the high risk areas are further secured by more eyes and more imagination focused on the code. That’s more experts applying their experience, their imagination and their skills to mitigate the risk that something subtle has gone unnoticed.

[Solidified Public Audit Reports](https://github.com/solidified-platform/audits)

