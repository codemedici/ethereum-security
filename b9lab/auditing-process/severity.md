# Severity

Categorization of Severity  
Risk ratings, as well as the processes we've just seen, vary greatly between organizations. Each company will have its own way to classify bugs. Even when the familiar categories of critical, major, and minor are used, the definitions of what’s included are inconsistent between firms.

This is another example of organizations in the space working independently on their own processes in a standards-free setting. When one finds a bug, it’s important to categorize it properly, according to the local customs of the audit team or bug bounty program.

While it may be tempting to subjectively stamp a discovered bug as Critical \(because it’s serious\), always review the bug bounty or company procedure / standard to ensure you do not categorize incorrectly. Remember, your bug can and will be refuted if it’s classified the wrong way so you need to be prepared to defend your classification of the bug as well as your description of the bug.

You need to be prepared to defend your classification of the bug as well as your description of the bug.

Several industry standards \(in the wider security industry, not smart contract audits\) address the topic. The most prominent of these is the OWASP \(Open Web Application Security Project\) risk classification standard, which is used by the Ethereum Foundation and many others. Another is the CVSS \(Common Vulnerability Scoring System\), created and mostly maintained by NIAC, a United States council that includes members from the American industry, infamous now after several members resigned once the US Government recently revoked funding to the council.

OWASP  
Let's take a closer at the OWASP risk classification standard, because it's the most widely used in open source projects, and it's also the one you'll most likely see when participating in bug bounties.

When one classifies a bug as “critical”, one is saying the risk related to the bug is very high, but what, exactly, is risk?

Risk  
Risk is a familiar term to everyone. Humans weigh risks in our daily lives with almost every action we take \(or do not take\). Think, for a moment, of how you quickly assess the situation in order to cross a street. These assessments are made almost instinctively, and risk-assessment of any given situation varies from one individual to the next. We have all met someone who takes risks we would not take \(for their betterment or to their detriment\). We have all met someone who regularly shies away from actions we perceive to be low-risk.

We can approximate these differences of opinion as different estimates of probability and different estimates of consequences.

But, if everyone's assessments are different \(the term for this is risk appetite - the amount of risk one accepts in order to achieve a goal\), doesn’t that mean that perceived risk is subjective? How can we objectively describe the risk of a given vulnerability?

Let's first properly define risk.

Risk = Likelihood \* Impact  
Risk equals the Likelihood of something materializing \(or, in our case, the likelihood of the bug being exploited\) times the Impact caused when it happens. This formulation is pervasive. It applies to everyone assessing risk across all domains. It applies to investment portfolios, regulatory compliance, smoke detectors and yes, even crossing the street. As you might have guessed, it applies to bugs in smart contracts.

With this understanding in mind, let's look at how OWASP breaks down likelihood and impact, making a previously purely interpretative assessment more objective.

Likelihood  
OWASP breaks likelihood into two sub-dimensions, with four items each. The final score is usually a simple average of all the values. The factors are:

Threat Agent Factors  
Threat agent is the possible attacker. The goal here is to estimate the likelihood of a successful attack by this group of threat agents. Use the worst-case threat agent.

Skill level

Skills: How technically skilled is this group of threat agents? No technical skills \(1\), some technical skills \(3\), advanced computer user \(5\), network and programming skills \(6\), security penetration skills \(9\)  
Motive: How motivated is this group of threat agents to find and exploit this vulnerability? Low or no reward \(1\), possible reward \(4\), high reward \(9\)  
Opportunity: What resources and opportunities are required for this group of threat agents to find and exploit this vulnerability? Full access or expensive resources required \(0\), special access or resources required \(4\), some access or resources required \(7\), no access or resources required \(9\)  
Size: How large is this group of threat agents? Developers \(2\), system administrators \(2\), intranet users \(4\), partners \(5\), authenticated users \(6\), anonymous Internet users \(9\)  
Vulnerability Factors  
The next set of factors are related to the vulnerability involved. The goal here is to estimate the likelihood of the particular vulnerability involved being discovered and exploited. Assume the threat agent selected above.

Ease of discovery: How easy is it for this group of threat agents to discover this vulnerability? Practically impossible \(1\), difficult \(3\), easy \(7\), automated tools available \(9\)  
Ease of exploit: How easy is it for this group of threat agents to actually exploit this vulnerability? Theoretical \(1\), difficult \(3\), easy \(5\), automated tools available \(9\)  
Awareness: How well-known is this vulnerability to this group of threat agents? Unknown \(1\), hidden \(4\), obvious \(6\), public knowledge \(9\)  
Intrusion detection: How likely is an exploit to be detected? Active detection in application \(1\), logged and reviewed \(3\), logged without review \(8\), not logged \(9\)  
Impact  
Impact is usually measured in financial terms, in OWASP's case it also derives from a number of factors:

Technical Impact Factors  
Technical impact can be broken down into factors aligned with the traditional security areas of concern: confidentiality, integrity, availability, and accountability. The goal is to estimate the magnitude of the impact on the system if the vulnerability were to be exploited.

Loss of confidentiality: How much data could be disclosed and how sensitive is it? Minimal non-sensitive data disclosed \(2\), minimal critical data disclosed \(6\), extensive non-sensitive data disclosed \(6\), extensive critical data disclosed \(7\), all data disclosed \(9\)  
Loss of integrity: How much data could be corrupted and how damaged would it be? Minimal slightly corrupt data \(1\), minimal seriously corrupt data \(3\), extensive slightly corrupt data \(5\), extensive seriously corrupt data \(7\), all data totally corrupt \(9\)  
Loss of availability: How much service could be lost and how vital is it? Minimal secondary services interrupted \(1\), minimal primary services interrupted \(5\), extensive secondary services interrupted \(5\), extensive primary services interrupted \(7\), all services completely lost \(9\)  
Loss of accountability: Are the threat agents' actions traceable to an individual? Fully traceable \(1\), possibly traceable \(7\), completely anonymous \(9\)  
Example:

Business Impact Factors  
The business impact stems from the technical impact, but requires a deep understanding of what is important to the company running the application. In general, you should be aiming to support your risks with business impact. The business risk is what justifies investment in fixing security problems.

Many companies have an asset classification guide and / or a business impact reference to help formalize what is important to their business. These standards can help you focus on what's truly important for security. If these aren't available, then it is necessary to consult with people who understand the business to get their take on what's important.

The factors below are common areas for many businesses, but this area is even more unique to a company than the factors related to threat agent, vulnerability, and technical impact.

Financial damage: How much financial damage will result from an exploit? Less than the cost to fix the vulnerability \(1\), minor effect on annual profit \(3\), significant effect on annual profit \(7\), bankruptcy \(9\)  
Reputation damage: Would an exploit result in reputation damage that would harm the business? Minimal damage \(1\), Loss of major accounts \(4\), loss of goodwill \(5\), brand damage \(9\)  
Non-compliance: How much exposure does non-compliance introduce? Minor violation \(2\), clear violation \(5\), high profile violation \(7\)  
Privacy violation: How much personally identifiable information could be disclosed? One individual \(3\), hundreds of people \(5\), thousands of people \(7\), millions of people \(9\)

We combine Severity and Impact to get to the severity of the Risk:

Using the example tables above, the issue would fall into High risk bug. Keep in mind that business impact is hard to measure without knowing details about the client, so technical factors usually prevail. OWASP provides a nice spreadsheet so we don't have to reinvent the wheel.

Although OWASP's model is the industry standard, when we look at our niche \(Ethereum smart contracts\), we'll find simpler models. At Solidified we encourage that, as simpler models make everyone's lives easier. Removing information that is less relevant when dealing with smart contracts creates focus on what is especially important.

With simpler models, it is still important to clearly outline what is included in each severity category, in order to remove doubt and subjective interpretation. The model below is very simple, but can be applied to most smart-contract-only bug bounties:

Critical: Stealing user funds, freezing funds in the smart contracts.  
Major: A user obtains advantage over others in an unintended way.  
Minor: Bugs that can cause friction to users, but put no funds at risk and create no unfair advantages for particular users.  
Solidified’s bug bounty platform categorizes the most common known vulnerabilities and anti-patterns, e.g. underflow, so that “other” is seldom required.

It is important to observe the classification standards of the organization or platform you are working with. Always review the rules before setting the category assessment because incorrect classification may lead to your report \(and your bounty claim\) being rejected. Remember to gather evidence supporting your report.

[OWASP Risk Rating Template](https://www.owasp.org/index.php/File:OWASP_Risk_Rating_Template_Example.xlsx)

