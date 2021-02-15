# Audit report

The Audit Report  
The Audit Report is the deliverable of the engagement. As such, it's important that it includes defined sections and communicates the project completely. These are the normalized section headings of a Solidified Audit Report:

Identification of the client  
Date  
Scope \(list all files\)  
Commit hash and repository address  
Bugs  
Conclusion

Document who requested the audit. It’s acceptable if the client requested anonymity. Your report should indicate this explicitly. Also document the date the audit was published, the files reviewed, bugs and concerns discovered and an overall conclusion about the health of the application.

Your writing style should address diverse audiences. For example, the client might be a Venture Capitalist with limited understanding of the technical details. The report may also appear in the public domain \(as do all Solidified Audit Reports\) and it might be read by end-users considering participation. Finally, it is a foregone conclusion that developers will be interested in technical explanations of bug reports. It is therefore important to make the issues and the overall sentiment clear for a non-technical audience, supported by unambiguous technical explanations of the issues and sentiment described.

Not all Audit Reports are prepared for such diverse audiences. If delivering to developers on a confidential basis, it may be acceptable to be less didactic while ensuring that bugs are clearly and concisely described and that the introduction and conclusion can be understood by the average ethereum user \(a technically literate user\).

In particular, be sure to describe the potential impact \(why it matters\) in terms that are understandable by the widest possible audience, and explanations in terms a developer can parse to comprehend the precise nature of the bug without further explanation.

Reporting Bugs  
Bug reports are the main product of both audits and bug hunts. A bug report is only as good as the understanding it provokes in the mind of the receiver. The central task of a bug report is to make the issue crystal clear to other people.

Also keep in mind that the audience of a bug report is often the very people who either wrote the code or audited it. They have looked at it from many angles \(perhaps not all angles!\) and your task is to change their minds about something they thought was correct. Explain in a matter-of-fact, non-accusatory tone and include sufficient information to support your claims.

Clearly describe the problem, the consequences, steps to get there, impact, severity and optionally a suggested direction for the fix.

Keep in mind that bugs are subjective. Indeed, considerable controversy can swirl around exactly what is and what is not a bug. For example, under certain conditions that probably cannot possibly exist, something terrible could happen. Or, under everyday conditions, something odd can happen but it is of no serious consequence.

We will dive deep into bug categories and severity classifications shortly. For now, let's focus on the central point, which is that a bug report should describe the discovery with completeness and precision.

Some good examples:

The ERC20 Approval Attack:

This is how the original ERC20 approval attack was described. Although the format is unusual, it has everything that a well-described bug should have: the context, the steps to the exploit, a brief analysis and a possible workaround.

CryptoKitties empty fallback:

This was found by Nick Johnson during the initial crypto kitties bug bounty. It follows a format more likely to be found in bug bounty and audit reports with concise explanation and consequences.

[ERC20 Approval Attack](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)  
[CryptoKitties Bounty Issue 3](https://github.com/dapperlabs/cryptokitties-bounty/issues/3)

