---
description: Run faster with security
---

# OZ Defender

Openzeppelin is mainly known in the Ethereum community for its set of utility smart contracts, that are used on a daily basis by Solidity engineers who want to develop faster with security \(DevSec\).

Openzeppelin Defender was born out of the idea to streamline the connection between DevOps and put security in every step. The goal is to develop and ship faster while following best practices, avoiding shorcuts.

Defender gives developers the tools and infrastructure to work faster and streamline various otherwise manual and error-prone tasks, in order to run faster with security \(SecOps\)

Defender is divided into four core components that are tightly integrated with each other yet modular enough to use those components independently of each other if needed, these are:

* Admin
* Relayer
* Autotask
* Sentinel

![](https://i.imgur.com/6nJ23Pk.png)

### Admin

> Interface for contract administration

Defender Admin is an interface for securely running administrative transactions on smart contracts, using Multisig wallets. Defender Admin acts as an interface for simplifying the process of running administrative functions, such as those implementing the "onlyOwner" modifier, so that the "owner" is in fact a Multisig wallet, rather than an individual person or key.

OZ recommends using either Gnosis Safe or the legacy Multisig Gnosis wallet, Admin just acts as an interface, it doesn't require granting any kind of rights to Defender for managing your contracts, so you can more easily execute, review and monitor the transactions.

Besides calling generic functions, Admin comes with a couple of pre-packaged actions, such as pausing contracts with a single click that will create a pause proposal \(the contract must be pausable in the first place\). Similarly, if your contract is upgradeable, there is a shortcut button for creating an upgrade proposal, provided an implementation address of the upgraded contract, defender will take care of packaging the transaction that goes through a Multisig of the proxy administrator and ultimately upgrade the contract.

#### Upgrades

Upgrades allow developers to replace the original implementation of a contract while preserving their address and state. Upgrades deserve a paragraph of their own because it is critical that contracts are upgraded securely and with minimal service disruption.

From the users' perspective, an upgradeable smart contract means having to increase their level of trust in external actors, because admins can now "pull the rug" from under their feet at any pint in time. That said, users of the contract probably wouldn't want to be in the situation where a critical bug is found and no one can pause or upgrade the contract in order to fix it. If the contract was not upgradeable, the developers would have to convince everyone to migrate to the new contract address \(both users and dApps\) and move their balances; this transition would be slow and not everyone will always get the message, putting their funds at risk.

Before beginning, it is worth pointing out the technical distinction between a proxy contract and an implementation contract:

* **Proxy contract**: this is what the world \(users, contracts, dapps\) see as our contract and interacts with, its address never changes. This is also where the balances and internal state lives, including a pointer to the implementation contract.
* **Implementation contract**: this is where the contract behavior/logic \(functions, events, etc\) live. When an upgrade happens, the proxy contract will change the pointer to a new implementation address. The only account that can cause this upgrade is called an "**upgrade admin**".

The rest of this paragraph only deals with transferring upgrade admin rights to a Multisig wallet, there are links in the resources should readers wish to learn about writing upgradeable contracts. The reason for having a Multisig wallet for the upgrade admin are mainly mitigating the risk of lost or compromised keys \(reducing centralization of trust\) and representing all stakeholders. For more practical advice, there is a great step-by-step article from Openzeppelin about [using multiple signatures for critical administrative tasks](https://defender.openzeppelin.com/#/advisor/docs/use-multiple-signatures-for-critical-administrative-tasks) and various best-practice considerations.

In order to transfer admin rights to a Multisig \(Gnosis Safe\), there is a function called `transferProxyAdminOwnership` that is exposed via the upgrades plugin. The Multisig can be set up with a number of private keys and it is also possible to define the number n-of-m signatures required directly from the Gnosis Safe UI, to get started on the Rinkeby testnet visit [rinkeby.gnosis-safe.io](https://rinkeby.gnosis-safe.io/app/#/welcome). The following snippet shows a simple script used to transfer ownership:

```javascript
async function main () {
  const gnosisSafe = 'replace with your Gnosis Safe multisig address';
  console.log('Transferring ownership of ProxyAdmin...');
  // The owner of the ProxyAdmin can upgrade our contracts
  await upgrades.admin.transferProxyAdminOwnership(gnosisSafe);
  console.log('Transferred ownership of ProxyAdmin to:', gnosisSafe);
}
```

The above script will call the `transferProxyAdminOwnership` on the admin object of the upgrades plugin, it can be run using hardhat as follows:

```javascript
npx hardhat run --network rinkeby scripts/transfer-ownership.js
```

Now that ownership has been transferred to Gnosis Safe, it will not be possible for a single account to do any sort or privileged operation by itself, what any of the Multisig signers can do is sign and propose or approve new proposals, which can be easily done using the Defender Admin UI. The proposal will only go through once n-of-m signatures have approved it.

Defender also provides a plugin for Hardhat, called `hardhat-defender`, which can be used for requesting to deploy a new implementation contract in a single line, as shown in the example below.

```javascript
async function main() {
  const proxyAddress = 'replace with your proxy address';

  const factory = ethers.getContractFactory('MyContractV2');
  console.log("Preparing proposal...");
  const newProposal = await defender.proposeUpgrade(pro, factory, { title: 'Propose Upgrade to V2' });
  console.log("Upgrade proposal created at:", newProposal.url);
}
```

This will validate that the contract is upgradeable and can be safely upgraded, then will deploy the implementation contract and instantly create an upgrade proposal within Defender Admin.

Finally, the upgrade proposal can be run like any other script:

```javascript
hardhat run scripts/propose-upgrade.js
```

behind the scenes the script can use a private key used for local development, since the upgrade will still need to go through a proposal for all the Multisig signers.

### Relayer

> Manages private keys and tx delivery

Relayers provide infrastructure for securely hosting private keys and simplifying the process of programmatically calling sensitive functions protected by access control.

When a new Relayer is created, behind the scenes Defender is going to create a private key in a secure keyboard that is assigned to every team member and that can only be used for a particular Relayer. Defender generates an API key and a secret that can be used for authenticating to Defender and sending the request for a transaction to be signed.

The defender UI will let you chose the contract and the function to call, so that the transaction can be sent directly from the address of the Relayer. The same can be done with a custom script calling Relayer's API, using an `npm` package called `defender-relay-client` either as a standalone or with ethers.js or web3.js.

In order to integrate it in a script, a default provider and Defender Relayer signer must be initialized using the credentials generated by Defender, i.e. the API key and secret:

```javascript
exports.handler = async function(credentials) {
    const provider = new DefenderRelayProvider(credentials);
    const signer = new DefenderRelaySigner(credentials, provider, {speed: 'fast'});

    const contract = new ethers.Contract(contractAddr, contractABI, signer);
}
```

After the contract's instance is creator using the signer, every time a transaction needs to be signed it will go to Defender Relayer to sign the transaction and broadcast it. Defender will also take care of nonces management so that they are increased whenever transactions are sent from any Defender account. Furthermore it will dynamically calculate the best gas prices for each transaction by checking multiple gas price oracles; the reader might have already noticed in the example above that the signer is generated with the option `{speed: 'fast'}` which will tell Defender to use a competitive gas price. Defender will also broadcast the transaction via multiple network providers for best availability. Finally, once the transaction has been broadcast, Defender will keep monitoring the transaction to make sure it gets mined and if it doesn't it will resend it with increased gas price.

To sum ip, relayer saves developers the trouble of setting up infrastructure for submitting transactions to the network and securely host private keys, additionally the `npm` package takes away the pain of writing the low-level code for building and signing transactions and keeping track of nonces.

### Autotask

> Executes custom code

Autotask helps managing recurrent administrative transactions thanks to serverless functions, think of them as small javascript snippets that are tightly integrated with Relayers and can run arbitrary logic that is triggered on a scheduled basis or by a public webhook.

Imagine a DeFi contract that has a function for withrawing all the transaction fees to a new location, this function might need to be called at predictable intervals or in response to certain events in order to sweep the fees from the contract. For that we should use a hot wallet \(Relayer\) for sending transactions on a regular basis; it would be inefficient and error-prone to do so manually with Metamask each time.

The code submitted to Autotask is like the one shown in preceding paragraph about Relayer, the only difference is that in Autotask you don't need to paste any king of API key or secret, as each task can be assigned to a particular Relayer through the UI. The UI also lets you choose whether to run the autotask on a scheduled basis, or via an HTTP post to a webhook. Using the webhook, a public URL is created that can be triggered, for example, by third party services for implementing gasless transactions by exposing that URL. Tasks can also be run manually at any time so that they can easily debugged.

Not only can autotasks run custom logic, they can also access external APIs, for that you can define secrets in your autotasks that can be accessed from within your autotask code without having to hardcode them in the autotask's script, as these secrets are managed via the UI and kept in a secure vault and made available at runtime when the tests are run.

### Sentinels

> Monitor function call and events

Sentinels are a component of defender for monitoring contracts so that conditions can be defined within contracts that need to be monitored and get real time alerts via email, slack/discord/telegram and even send their data to the Datadog cloud monitoring service for analysis. It is even possible to invoke Autotask in response to that, by defining any kind of transaction that will not only trigger an alert, but also run Autotasks in order to react to them and send a transaction to Relayers to immediately respond to whatever happened on the contract being monitored.

As an example, let's say we needed to monitor large "whale" deposits to a DeFi contract, in which case a Sentinel would be triggered, which in turn would call a "sweep" function for draining the contract from all of the transaction fees accrued thus far by the contract, this transaction is passed to the Relayer which actually moves the funds out of the contract by signing and broadcasting the transaction.

In sentinel there are two ways to define what triggers an alert. First by monitoring transaction properties, filtering if certain conditions are met, the conditions can be applied on the following transaction parameters: value, gasPrice, gasUsed, to, from, nonce, status \('success' or 'failed'\) or input. For example, the following filter can be applied in order to trigger alerts for transaction with a price greater than 10 gwei and only if they are successful:

```bash
gasPrice > 10e9 and status = 'success'
```

The other way to trigger alerts is by reacting to events through contract conditions, these specify which events and function calls should be monitored, by monitoring specific conditions within events and functions. The Defender UI lets users select from all the available events and access each event's variables programmatically using conditional statements.

For example, let's say that a `deposit` function emits a `Deposited(address from, uint256 value)` event; if the goal was to monitor whale transactions, the conditional statement to check for transfers higher than 1000 ETH would be:

```bash
value > 1000000000000000000000
```

Basically, Sentinels allow to set conditions based on arguments of the functions that were called on the contract as well as the arguments of the events that were fired.

After creating a sentinel it is possible to configure what happens when a transaction or an event that matches the predefined conditions is mined, such as sending an email or triggering a Slack webhook. Most importantly, it is possible to trigger Autotasks. Very commonly, Sentinels are also used for monitoring high failure rates, or monitoring for unusual activity that could indicate a hack and therefore immediately pause a contract.

### Advisor

Advisor is a knowledge-base with security best practices that have been collected throught the years, these best practices are grouped under the categories of development, testing, monitoring and operations, each best practice carries a security rating score and an "effort" rating for implementing such best practice.

It is possible to query this knowledge base for any security related topic, such as reentrancy, and it will return a page containing a description of the attack, links to previous instances of these attacks, what patterns can be implemented to protect against it, some code examples and various other resources.

Although it won't remove the need for an audit, it can certainly help in guiding developers about best practices they should be aware of. This knowledge-base is being continuously updated.

### Resources

**Defender**

{% embed url="https://defender.openzeppelin.com" caption="" %}

**Community Forum**

{% embed url="https://forum.openzeppelin.com" caption="" %}

**Documentation**

{% embed url="https://docs.openzeppelin.com/defender/" caption="" %}

**OZ Upgradeability Workshop**

{% embed url="https://github.com/OpenZeppelin/workshops/tree/master/05-upgrades-management/code" caption="" %}

**Writing Upgradeable Contracts**

{% embed url="https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable" caption="" %}

**Use Multiple Signatures for Critical Administrative Tasks**

{% embed url="https://defender.openzeppelin.com/\#/advisor/docs/use-multiple-signatures-for-critical-administrative-tasks?query=signatures" caption="" %}

