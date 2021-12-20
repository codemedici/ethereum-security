---
description: >-
  Ensure that all critical administrative tasks require multiple signatures to
  be executed.
---

# Use multiple signatures for critical administrative tasks

### Description

Critical administrative tasks should not be controlled by a single signer. Requiring multiple signatures to execute these sensitive tasks reduces the impact of private keys being lost or compromised.

Transactions that require multiple signatures can be implemented with a [Multisig Wallet](https://academy.binance.com/en/articles/what-is-a-multisig-wallet) or a [Threshold Signature](https://academy.binance.com/en/articles/threshold-signatures-explained).

As with many parts of this young ecosystem, there are not many documented best practices or experiences for using multiple signatures. Take into account that while they can increase the security of your system when carefully implemented, they are also an extra layer of complexity that can introduce new risks and points of failure.

Following are a set of guidelines and recommendations to take into account when implementing your multisignatures.

* Assign different roles for different types of tasks. [OpenZeppelin Contracts provides the Roles library](https://docs.openzeppelin.com/contracts/2.x/access-control#role-based-access-control) for implementing role-based access control.
* Define a hierarchy of permission levels linked to the roles.
* Require a higher number of signatures for roles higher in the hierarchy.
* Use independent multisignatures for every role.
* Give ownership of every private key to a different individual.
* Train every key-holder on the secure handling of their devices and communications. The [Electronic Frontier Foundation published good resources for safe online communications](https://ssd.eff.org).
* Distribute the keys to individuals that guarantee diversity of geographic locations, hardware manufacturers, and software applications.
* Do not use a 1 of N multisignature because losing control of a single private key means the system is immediately compromised.
* Do not use an N of N multisignature because it does not provide any redundancy in case some of the private keys are lost.
* Consider using an M of N multisignature, with `M = (N/2) + 1` to balance concerns. A lower M allows for quicker response. A higher M requires a stronger majority support.
* For configuration tasks, like changing parameters, a 2 of 3 multisignature might be enough. Consider using other kinds of fail-safes, like allowing the parameters to vary only by a small delta and with some time window before they are applied, so the functioning of system does not completely change with a single transaction. Consider separating the tasks that are adjustments from the ones that disable or enable functionality. For example, do not allow a parameter to be adjusted to 0 if it means a functionality will stop working; define a separate function with a separate role to disable it.
* For more critical and less common tasks, like upgrading the implementation of a contract, a 3 of 5 multisignature might be better. If the tasks are time-sensitive, the key-holders must agree to be on-call to react immediately after being notified.
* For tasks that require representation from different stakeholders, consider giving ownership of private keys to two representatives of each stakeholder category.
* Document the roles, corresponding tasks, and key-holder contacts in a private repository.
* Document the threat model for every role and task.
* Automate the monitoring, administrative triggers, and notification of key-holders.
* Test and run simulations of all the situations that will require multiple signatures.

Note that multisignatures are part of permissioned systems. While they can be implemented with a little decentralization, they are far from the ideal of permission-less systems that are maintained stable by the interest of all their participants. Consider them as a fail-safe during the early stages of your system, or as a temporary solution until safer governance mechanisms evolve.

See the [Defender Advisor](https://docs.openzeppelin.com/defender/advisor) article "Secure All Administrative Keys" for best practices related to the individual private keys.

[Gnosis Safe](https://gnosis-safe.io) provides a graphical user interface to set up multisignature wallets for the Ethereum blockchain.

The [Defender Admin](https://docs.openzeppelin.com/defender/admin) service acts as an interface to manage your smart contract project through one or more secure multi-signature contracts.

### Example

Create a directory in a private repository only accessible to the people in your team in charge of operations and security.

For every task that requires a multisignature to be executed, create a document in that folder following this example.

#### Task: Pause the trading functions

**Description**

During a security incident all the trading functions should be paused while the teams investigate further and define a solution.

**Role**

* Name: `Pauser`
* Contract: `Trading.sol` deployed in the Ethereum mainnet at address `0x...`
* Function: `pause`
* Multisignature contract: [`GnosisSafe.sol` version 1.2.0](https://github.com/gnosis/safe-contracts/blob/v1.2.0/contracts/GnosisSafe.sol) deployed at address `0x...`
* Threshold: 2 of 3
* Key holders: 1 member of the security team, 1 developer, the CTO

**Keyholders**

| Name             | E-mail address                                    | Signal phone number | Alarm phone number |
| ---------------- | ------------------------------------------------- | ------------------- | ------------------ |
| Satoshi Nakamoto | [satoshi@example.com](mailto:satoshi@example.com) | (+###) ########     | (+###) ########    |
| ...              | ...                                               | ...                 | ...                |
| ...              | ...                                               | ...                 | ...                |

**Alert triggers**

* A security vulnerability is disclosed that can result in stolen funds from traders' accounts
* Monitoring flags suspicious activity that seems to have resulted in stolen funds from traders' accounts
* Credentials to deploy the user interface are compromised
* The Chainlink oracle is compromised
* DAI loses its peg by 1%
* ...

**Procedure**

1. In the #incident-response channel of the team private Slack, the security team member on call notifies about the incident and about the start of this procedure.
2. A security team member makes a phone call to the alarm number of every key holder to get their immediate attention.
3. A security team member sends a GPG encrypted email to all the key holders, notifying them about the incident and requesting them to confirm in Signal and to join the #incident-response channel in Slack.
4. A security team member makes a video call confirmation in Signal to all the key holders.
5. A security team member makes the [Pause action proposal in OpenZeppelin Defender](https://docs.openzeppelin.com/defender/admin#pauseunpause)
6. Every key holder joins the #incident-response channel and verifies the identity of the GPG signed email and verifies the `pause` transaction
7. Every key holder reviews and approves the [Pause action proposal in OpenZeppelin Defender](https://docs.openzeppelin.com/defender/admin#pauseunpause)
8. A security team member notifies the owners of the resume task so they are ready to follow that procedure.
9. A security team member notifies the communication team to prepare an announcement about the incident.

**Simulation exercise**

Every 2 months, a member of the security team will trigger a simulation of this procedure in a test net.

**Threat Model**

**The keys of the holders of the pause task can be compromised**

Mitigations:

* Every key holder will be trained on secure communications and safe storage of digital assets
* In the case of a key being compromised, the holder will immediately notify the security team to start the procedure to replace the key
* 2 signatures will be required to execute the action
* The 3 key holders should be geographically distributed most of the time
* When 2 of the key holders will be in the same geographic location, the security team will be notified and a protocol with extra precautions will be implemented
* The 3 key holders should use different technology stacks to sign their transactions

**The pause procedure can be started by compromised team credentials**

Mitigations:

* The notification will be a GPG signed email
* The notification will be confirmed with a Signal video call
* Before signing the transaction, the key holders will check the Slack thread to get informed about the situation

**The key holders refuse to execute the Pause action**

Mitigations:

* 2 of 3 signatures will be required to execute the action
* The key holders should have demonstrated their good intentions for the success of the project
* The key holders will be under contractual obligation to perform this task
* The sytem could also be paused through an upgrade executed by a multisignature with higher privileges

**The information about the incident can leak**

Mitigations:

* At any time, at least 1 member of the security team will be on-call to diagnose incident severity as soon as possible
* Communication about the incident should only occur on trusted platforms
* While the incident is ongoing, the communication channel should be restricted to the participating individuals
* No information will be communicated out of the team before an appropriate response has been prepared
* The communications team should join the incident response team to help prepare the public disclosure

**The system can be paused without good reason**

Mitigations:

* At any time, at least 1 member of the security team will be on-call to diagnose incident severity
* 2 signatures will be required to execute the action
* Before signing the transaction, the key holders will check the Slack thread to get informed about the situation

**The incident could be a distraction to prepare a bigger attack**

Mitigations:

* The only action that can be executed by the Pauser role is to pause trading
* The key holders of the Pauser role will not be holders of keys for roles with higher privileges
* The [Pause action proposal and signatures will be executed in OpenZeppelin Defender](https://docs.openzeppelin.com/defender/admin#pauseunpause)
* The addresses of the contract to pause and the key holder accounts will be added to the [OpenZeppelin Defender address book](https://docs.openzeppelin.com/defender/admin#address-book) with a descriptive alias.
* The key holders will be trained on how to review a transaction in OpenZeppelin Defender before signing it
* While a part of the team is investigating the incident, the rest will be monitoring other sensitive areas of the system

## Resources

* [https://defender.openzeppelin.com/#/advisor/docs/use-multiple-signatures-for-critical-administrative-tasks?](https://defender.openzeppelin.com/#/advisor/docs/use-multiple-signatures-for-critical-administrative-tasks?)
