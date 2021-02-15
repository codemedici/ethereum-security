# specifying intended behaviour

Specifying Intended Behavior  
The auditor is tasked with ensuring the application behaves as specified. Where, exactly, is application behavior specified? This will vary greatly from project to project, but ideally there should exist a succinct document outlining the goal of the application, what is allowed and what is prevented. We call this a Statement of Intended Behavior. It should be precise and unambiguous so auditors can compare what the developers want to happen and the code that is intended to make it happen.

A Statement of Intended Behaviour will be presented as a separate document, sometimes as part of the repository's wiki or readme.md. Sometimes the document is simply non-existent. In such a case, request that the developer, along with the rest of his team create a document before the audit starts. Input from business-focused professionals is valuable. Sometimes, they will have a clearer view of how the system should behave.

Let's dive into some examples, starting with a concept familiar to all, a crowdsale. In a crowdsale Statement of Intended Behavior \(let us refer to this as “specification” in this context\), you will often see:

Dates \(opening, closing, discount periods\)  
Price  
Discounts  
KYC requirements \(whitelist\)  
Access controls \(may be static or editable\)  
Caps, refunds, etc.  
All very familiar, right? They will give you an insight into how the application is supposed to work, and what should be tested during the engagement.

The size of the specification will be proportional to the complexity of the application. To generalize for any application, the specs should include:

Goal of the application  
Main flows  
The actors / roles and what they do  
Access restrictions  
Failure states to be avoided  
One caveat: You will stumble upon specifications that seem to be wrong, and in fact are. If you notice that the owner of the contract can drain the contract of user's funds, it seems obvious that it needs to be reported. But what if the client has specified this as intended behavior?

This is always a tough call, and has been discussed many times such as in our auditor Adam Kolar's article and recently in the [unsolicited audit of Compound Finance's contracts.](https://medium.com/@ameensol/what-you-should-know-before-putting-half-a-million-dai-in-compound-fafdb2645f77)

This raises an interesting question to consider. Who is the audience of the report? Is it the users of the application, or the client who paid for the audit? Again, opinions vary in this formative industry. We, at Solidified feel that audience is the community. Naturally, clients hope for positive reports but the report itself is about the findings of a process. Compromising the process would ultimately reduce the integrity, and value, of the reports. Therefore, such a finding would be documented. If the client resists resolution \(more on remedial steps shortly\), the issue would remain in the report as an unresolved concern.

When in doubt, document the issue in the report. The whole purpose of our industry is to create systems where trust is not required, or its role is greatly minimized. Try to model the expectations of a user and the client's claims \(surprise is an anti-feature\). Scale them, and err on the side of caution. The report can and should list what is possibly a concern. That is, what has not been explicitly eliminated, by code, from the scope of possible concerns.

[Guiding Users Through The Dangerous Waters Of Smart Contract Security](https://medium.com/solidified/guiding-users-through-the-dangerous-waters-of-smart-contract-security-2f5bee033086)  
[What You Should Know Before Putting Half A Million DAI In Compound](https://medium.com/@ameensol/what-you-should-know-before-putting-half-a-million-dai-in-compound-fafdb2645f77)

