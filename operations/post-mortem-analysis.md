---
description: >-
  For every production issue that requires emergency response, conduct a
  post-mortem analysis, document follow-up actions, and track actions to
  completion.
---

# Post-Mortem Analysis

## Description

If a production issue occurs that requires emergency response, whatever the root cause, once the issue is resolved then the most important next step is to identify what actions should be taken to prevent the same or similar issues from occuring in the future. Actions might include:

* improve monitoring and detection
* enhance emergency response processes
* add new security tools or code protections
* improve system infrastructure
* modify smart contracts

In order to ensure that issues are not forgotten, it is recommended that you conduct an in-person post-mortem analysis meeting within 48 hours of the completion of response to any production issue. All key responders should be invited, with one person designated to take notes and document the post-mortem meeting.

The discussion at the post-mortem meeting should be blameless and factual. The goal of the meeting is to review the timeline of exactly what occurred and use that to identify specific actions the team can take to fix identified issues, prevent similar issues, or improve response capabilities in the future. If the root cause of the issue is not yet identified, the first action should be to identify and document the root cause, which may in turn trigger the need for further follow-up actions. For more information on how to conduct post-mortem meetings see the [Atlassian Incident Management Handbook](https://www.atlassian.com/incident-management/handbook/postmortems).

After completion of the post-mortem analysis meeting, the meeting notes and actions should be published internally. Response team members, or others who might have involvement in the follow-up actions, can add edits and comments to the document.

All follow-up improvement actions identified in the post-mortem analysis should have an assignee, a target completion date, and should be tracked to completion. In order to ensure completion you may use an automated issue-tracking system the team should review the open post-mortem issue list monthly.

## Example

Create a shared document to track the post-mortem summary and actions. The document might be stored in a shared file system, or GitHub, Google Docs, or a shared workspace editing system such as [Notion](https://www.notion.so/). Use a shared document with open permissions for team members to edit so that individuals not attending the post-mortem analysis meeting can add information.

The following is an example outline that you can use for the post-mortem analysis.

#### 1. Incident Timeline

| Time \(UTC\) | Description |
| :--- | :--- |
| 04:18 | Failed TXs alarm fired |
| 04:20 | First responder begin reviewing TXs |
| 04:25 | Received first user notice of failures |
| 04:30 | First public notice on social media |
| 04:45 | First responder notified core response team |
| 05:00 | Emergency channel open |
| 05:23 | Team identified issue in account |
| 05:55 | Script to fix issue completed and tested |
| 06:10 | Code review completed |
| 06:19 | Upgrade deployed and TX submitted |
| 06:30 | Confirmed that issue resolved |

Work with the team to document the timeline of all relevant events and actions during the incident under review. It is not necessary to list names of participants in the timeline, the critical aspect is to identify what occurred and what actions were taken.

#### 2. Root Cause

_Root cause: due to a rapid price drop a user account became under-collateralized, an attempt to liquidate the account failed due to an error in the smart contract which in turn caused all liquidation calls to fail. The code involved was not audited._

Provide a clear description of the root cause. If the root cause is unknown, assign an action to find and publish the root cause. Knowing the root cause is critical to ensure that you have defined all important follow-up actions to prevent future issues, and the root cause description may be required to share with users.

#### 3. Worked Well

| Worked Well |
| :--- |
| Failed TX detection |
| Responders notified and online fast |
| Internal communications |
| Root cause identification by examining TXs |
| Patch development, testing, code review |

List out all aspects of system protections and response that worked well during the incident. Include mention of communications and processes as well as technical actions and use of tools.

#### 4. Could Be Better

| Could Be Better |
| :--- |
| Response to users and on social media |
| Upgrade deployment tooling |
| Audit all smart contract modifications |

Based on the incident timeline and understanding of the root cause, discuss and document anything that could have been better in avoiding or handling the production incident.

#### 5. Follow-Up Actions

| Action | Assigned To | Target Complete |
| :--- | :--- | :--- |
| Add step to involve comms early in emergency | Susan J | March 15 |
| Implement tool for faster smart contract upgrade | Adam K | April 15 |
| Establish plan to audit incremental changes | Tony L | March 22 |

Using the list of items that the team identified as Could Be Better, create a list of specific follow-up actions that the team will commit to complete within a specified timeframe. Each action should be assigned with the understanding that it will be tracked to completion. The goal of the follow-up actions is to ensure that the specific issue, and as far as possible similar issues, will not occur or will be resolved much more quickly in the future.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/post-mortem-analysis?](https://defender.openzeppelin.com/#/advisor/docs/post-mortem-analysis?)

