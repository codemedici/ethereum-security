# DeFi Best Practices



The DeFi space has had a tumultuous couple months, with a number of attacks as well as unexploited vulnerabilities being reported.

Bugs are unavoidable, but there are many things that can be done to reduce their frequency, and mitigate their negative effects.

As auditors, we want to help, but in order to really get developers to truly prioritize security, users need to start asking tough questions, and putting their money into the protocols that can answer them thoughtfully.

The following list of questions is useful for understanding a development team’s security stance. There isn’t always a right or wrong answer, and some teams \(or solo developers\) don’t have the resources to cover all the bases, regardless users deserve to have the information to decide what level of risk they are comfortable with.

We hope that asking these question will push the conversation in the right direction.

## Admin permissions

The majority of popular defi protocols have some form of centralized control that enables specific ‘administrator’ addresses to intervene in powerful ways.

This has some security benefits, but it means that you have to trust the administrator\(s\) not to abuse their privileges. It also adds the risk of an attacker gaining access to an administrator’s private keys, and all the privileges that come with them.

An administrator account can take several possible forms, including a single address, a multisig wallet, or even be a DAO controlled by a voting process.

* What special actions can administrators take?
  * Pausing the system?
  * Modifying balances?
  * Whitelisting/blacklisting of tokens and/or users?
  * Upgrading a subset of the system.
  * Upgrading all of the system \(which is equivalent to omnipotence\).
  * Anything else?
* Which of these actions ones do and do not have a time delay on them?
* If there is a time delay, how long is that time delay?
* How many people have administrator privileges?
* How many of those admins must approve before some action is taken?
* Are any administrative actions controlled by on-chain governance \(ie. a DAO\)?
* Where can I stay up to date about proposed changes to the protocol?

Some of this information is already being tracked at [DefiWatch](https://defiwatch.net/admin-key-config-and-opsec/project-reviews).

## External Dependencies

The Ethereum chain is full of adversarial actors, so in general developers should avoid making any assumptions about how contracts from other systems will behave. In many DeFi applications this is not possible, as services are built on top of existing contracts.

These questions should help expose risks related to external dependencies.

* What oracles does your system depend on?
* What exchanges does your system depend on?
* What 3rd party smart contracts did you use to build your system \(ie. OpenZeppelin\)?
* What tokens does your system support, and what assumptions do you make about their functionality?

## Responsible disclosure and bounty programs

For talented hackers, there are strong financial incentives to attack DeFi protocols. Having a bounty program in place creates a financial incentive to report vulnerabilities rather than exploit them. Reporting a vulnerability through a bounty program is also good for a hacker’s reputation, and the added benefit of not being illegal.

Any company running a DeFi protocol, with people’s money on the line, should have a bounty program. Here are some good questions you can ask about their program and disclosure process:

* Is the source code of your contracts publicly available?
* Is it easy to find the security contact information on your website and git repos?
* Do you have a bounty program on your contracts?
* Which contracts are in scope?
* What is the range of bounty payments?
* Have you ever made a bounty payment?
* Have you ever denied payment on a bug report?
* Is it easy to find details of the bounty program on your website and git repos?

Ideally this information would all be found at “website.com/security” and make use of GitHub’s SECURITY.md feature.

## Incident response planning

It’s difficult for developers to think clearly during a security incident, while new information keeps coming in, and users are asking tough questions on Twitter, Telegram, Discord etc…

Having a plan is evidence of a healthy orientation towards security. It may not make sense for a team to make their full plan public, but they should be able to answer some basic questions about it:

* Do you have a written plan outlining how to handle a security incident?
* Which scenarios does your plan take into consideration?
* If your system is upgradeable, are the steps to do so well documented?
* If you discover a vulnerability which places funds at risk, would you preemptively exploit it to protect the funds?

## Audits and secure development

Auditing is not a silver bullet, and not all audits are equal, but it is a crucial step for any DeFi contracts before deployment.

There isn’t necessarily a ‘right answer’ for each of these questions, but the answers should enable knowledgeable community members to get a feel for the developer teams’ security stance.

* When was your last audit?
* How much effort \(in person-hours\) went into the audit?
* Which firm\(s\) performed it?
* Is the report public?
* Was any portion of your system excluded from the scope?
* Have your contracts been upgraded since your last audit? If so, what changed?
* Do you have an ongoing relationship with any security firms?
* Do developers review each other’s PRs \(at least in Solidity files\) before merging?
* What portion of your contract code is covered by unit tests?
* Do you use any other security analysis tools in your process?

