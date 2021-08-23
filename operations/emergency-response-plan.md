---
description: >-
  Prepare a plan for emergency response, and conduct live exercises at least
  twice a year to increase team preparedness.
---

# Emergency Response Plan

## Description

Even in the earliest stages of running a protocol or application in production, it is important to have a clear plan for emergency response. There is no worse time to figure out how to achieve fast and safe response as when an emergency is actually occurring. Lack of preparation leads to delays in response and sometimes leads to critical mistakes.

In order to have a clear plan that is understood by all participating team members, the recommended best practice is to document a plan with as many details as possible, and then to conduct live exercises at some frequency where the team reviews and practices the steps to ensure familiarity and readiness. Over time the plan can be refined and improved and manual steps can be automated.

## Example Plan for Start-Ups

Create a shared document for the details of your emergency response plan. The document might be stored in a shared file system, or GitHub, Google Docs, or a shared workspace editing system such as [Notion](https://www.notion.so/). Use a shared document with open permissions for team members to edit so that you can add to and improve the document over time.

The following is an example outline that you can use for your initial plan.

### 1. Potential Issues

| Category | Issue | Detection |
| :--- | :--- | :--- |
| Hacker Attack | Protocol DDoS | Spike in TXs or events |
| Hacker Attack | Stolen User Funds | Reports on social media |
| Network Problem | Spike in Gas Funds | Spike in Failed TXs |
| Protocol Problem | Unknown Bug | Spike in Failed TXs |

Brainstorm with your team on all the potential issues that could occur in production that would create an emergency. This might include unexpected system bugs, hackers stealing funds from users, DDoS attacks, failure of critical subsystems, hackers stealing administrative access credentials, or system overloads leading to sub-standard performance. Document each of the potential issues along with how each will be monitored and detected. Monitoring may be manual to start, however it is advisable to prioritize automated detection especially for the potential issues that might have the most serious impact. Issues may be categorized into types, for example you might have "Network Problems", "Protocol Problems", and "Hacker Attacks".

### 2. Response Procedures

| Issue | Response Steps | Details |
| :--- | :--- | :--- |
| Stolen User Funds | Pause contract | \(provide details on how\) |
|  | Identify exploit | \(specific tools to use\) |
|  | Create/test patch | \(suggest test methods\) |
|  | Emergency upgrade | \(provide details on how\) |
|  | Quantify losses | \(identify scripts/methods\) |
|  | Determine cleanup | \(identify approaches\) |

Identify all the response procedures that will be used, making sure to cover responses required for all of the identified potential issues. Response procedures should include all steps required to halt the issue, fix the issue, and fully restore services. The response procedures might include detailed steps that might be used in emergency situations such as:

* steps to pause your protocol
* steps to display a maintenance page to app users
* steps to release an emergency patch or upgrade
* instructions on key tools or scripts you rely on for maintenance or response

Response procedures should be documented in a step-by-step way so that any team member can follow the instructions. The procedures should also be mapped to potential issues, so in the case of an actual emergency any team member can find the recommended response procedures for the particular issue.

### 3. Escalation Process

| Category | Core Response Team | Extended Response Team |
| :--- | :--- | :--- |
| Hacker Attack | Upon detection | Upon validation |
| Protocol Problem | Upon validation | Upon validation |
| Network Problem | Upon validation | Upon resolution |

Document which groups of individuals should be notified when potential issues requiring emergency response are identified and who should be notified when emergency issues are confirmed. The escalation process may be different for different types of issues, for example in the case of Network Issues the escalation would go to the network administrators, whereas in the case of Application Issues the escalation would go to the application developers. In the case of system hacks it might also be necessary to escalate to people outside your organization. The document should make it easy for any reader to understand what escalation process to follow in the case of any specific emergency.

### 4. Core Response Team

| Name | Categories | Email | Chat | Phone |
| :--- | :--- | :--- | :--- | :--- |
| Janet R | Hacker, Protocol | [janet@example.com](mailto:janet@example.com) | @janet | +19995551212 |
| David L | Protocol, Network | [david@example.com](mailto:david@example.com) | @david | +19995551213 |

Create a list of all emergency response team members along with their contact information such as email addresses, chat handles, or phone numbers. If core response team members are responsible for specific categories of issues that should be noted in the list. Any reader should be able to use the list to find and contact a core response team member. Presumably this list will include administrators, operators, and key developers.

### 5. Extended Response Team

| Name | Role | Email | Chat | Phone |
| :--- | :--- | :--- | :--- | :--- |
| Eveline B | Security Consultant | [eveline@external.com](mailto:eveline@external.com) | @eveline | +19995551214 |
| Martin K | Community Liason | [martin@example.com](mailto:martin@example.com) | @marting | +19995551215 |
| Officer | Law Enforcement | [officer@fedlaw.com](mailto:officer@fedlaw.com) |  | +19995551216 |

Create a list of extended team members, individuals or organizations that might need to be notified in the case of or assist with emergency response along with their contact information such as email, chat handles, and phone numbers. Each extended team member's role should be noted in the list, for example if a developer has a particular area of knowledge, or if a member of the marketing or support team is responsible for customer communications. The extended team members might include people outside your organization, such as specialized security researchers, law enforcement, or responders. Any reader should be able to use the list to find and contact an extended team member.

### 6. Communications Process

| Communication | Timing | Channel | Responsibility |
| :--- | :--- | :--- | :--- |
| Responders | Open until completion | Telegram new channel | First responder |
| Internal updates | Hourly | Slack \#emergency channel | Responders |
| Users | Every 4 hours | Twitter, Telegram, Discord | Community Liason |
| Internal post-mortem | Resolution + 2 days | Slack \#operations channel | Responders |
| Public post-mortem | Resolution + 5 days | Company blog post | Head of support |

Document the details of how communications should occur during and immediately after any emergency. This should include:

* secure communications channel\(s\) to use for emergency responders
* internal communications channel\(s\) for emergency response activity updates
* frequency of internal updates
* responsibility for user communications during emergencies
* frequency of user updates
* timing to deliver internal and external post-mortems

Certain elements of communications may not be required for all emergencies, this should also be noted in the document.

### 7. Decision Process

| Decision | Required Approvers | Additional Approvers |
| :--- | :--- | :--- |
| Pause contract | CEO, COO | Administrators |
| Upgrade contract | Responders | Administrators |
| Access emergency fund | CEO, CFO |  |
| Notify law enforcement | CEO, COO |  |

Document all responses that might need approval before action, and identify who among the core or extended team members are the approvers. For example, if it is necessary to pause your protocol or issue an emergency patch or upgrade, you may want to ensure that certain individuals have approved. Approvals may be tracked in an automated fashion \(with key signing or some other automated tracking system\) but in some cases where that is not possible the approvals may be given via email or in a secure communications channel.

## Example Live Exercises

Once you have created an emergency response plan, run live excercises to practice the response procedures. This will provide training for the team members, ensure familiarity with the plan, and help identify potential flaws or missing steps in the plan before an emergency occurs.

For new production systems and newly developed emergency response plans, it is advisable to conduct live exercises more frequently until the team is comfortable with the plan. For example you might conduct 2-hour sessions once every few weeks or once a month until the team feels they have a thorough understanding of the plan and procedures.

After familiarity is firmly established, it is advisable to continue to conduct live exercises at least once every six months to ensure that team members maintain their awareness of the plan. In this case the exercise sessions might be longer, for example 4 to 6 hours with breaks, to cover a full set of potential emergencies.

The following is a description of how to run a live exercise for emergency response:

```text
1. Invite all members of the core response team
2. Assign one team member to act as the Moderator (also acts as exercise recorder)
3. The Moderator describes a scenario that indicates a potential emergency
4. Core response team members describe the steps to take out loud
5. When possible, team members execute the steps (possibly using a live environment)
6. If the steps include discovery, the Moderator declares what issues are found
7. The team proceeds through all steps of the plan until the Moderator declares completion
8. The Moderator reads back the steps taken and conclusion
9. The team discusses mistakes made or issues found in plan, the Moderator records findings
10. The Moderator chooses another scenario and steps 3-9 are repeated
```

Exercises can be recorded in notes by the moderator, they also might be recorded by video. After exercises are complete, the notes and videos can be published for those who could not attend and for potential future review by new team members.

## Example Plan for Larger Organizations

For larger organizations who have moved beyond the start-up or early company stage \(100+ team members\), where production systems have an increased number of components and operators, it will become increasingly useful and necessary to implement a more detailed emergency response plan. For this it is suggested you research available plan templates and sources, one recommended initial source is the [Framework for Incident Response](https://myresources.itrevolution.com/viewer/?Id=006657105) published by the DevOps Enterprise Forum.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/emergency-response-plan?](https://defender.openzeppelin.com/#/advisor/docs/emergency-response-plan?)

