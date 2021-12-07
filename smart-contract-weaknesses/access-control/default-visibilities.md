# Default Visibilities

## Default Visibilities

Since pragma `0.5.0`, function visibility must be explicitly marked. In previous versions the visibility modifier in Solidity defaults to public for functions and internal for variables. Public access to functions intended for internal use only can be disastrous, so keep an eye on that when dealing with older compilers.\
Also note that gas cost varies with visibility. In some cases, public functions will spend considerably more gas than an external function would, so always favor external visibility, when possible.\
There are four visibility specifiers, which are described in detail in the [Solidity docs](http://bit.ly/2ABiv7j).\
Example:

contract HashForEther {

&#x20;   function withdrawWinnings() {\
&#x20;       // Winner if the last 8 hex characters of the address are 0\
&#x20;       require(uint32(msg.sender) == 0);\
&#x20;       \_sendWinnings();\
&#x20;    }

&#x20;    function \_sendWinnings() {\
&#x20;        msg.sender.transfer(this.balance);\
&#x20;    }\
}

Unfortunately, the visibility of the functions has not been specified. In particular, the `_sendWinnings` function is public (the default), and thus any address can call this function to steal the bounty.

### Preventative Techniques

It is good practice to always specify the visibility of all functions in a contract, even if they are intentionally `public`. Recent versions of solc show a warning for functions that have no explicit visibility set, to encourage this practice.

### Real-World Examples

#### Parity Multisig Wallet (First Hack)

In the first Parity multisig hack, about $31M worth of Ether was stolen, mostly from three wallets. A good recap of exactly how this was done is given by [Haseeb Qureshi](https://bit.ly/2vHiuJQ).\
Essentially, the multisig wallet is constructed from a base `Wallet` contract, which calls a library contract containing the core functionality (as described in [Real-World Example: Parity Multisig Wallet (Second Hack)](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#multisig\_secondhack)). The library contract contains the code to initialize the wallet, as can be seen from the following snippet:

contract WalletLibrary is WalletEvents {\
&#x20; ...\
&#x20; // METHODS\
&#x20; ...\
&#x20; // constructor is given number of sigs required to do protected\
&#x20; // "onlymanyowners" transactionsas well as the selection of addresses\
&#x20; // capable of confirming them\
&#x20; function initMultiowned(address\[] \_owners, uint \_required) {\
&#x20;   m\_numOwners = \_owners.length + 1;\
&#x20;   m\_owners\[1] = uint(msg.sender);\
&#x20;   m\_ownerIndex\[uint(msg.sender)] = 1;\
&#x20;   for (uint i = 0; i < \_owners.length; ++i)\
&#x20;   {\
&#x20;     m\_owners\[2 + i] = uint(\_owners\[i]);\
&#x20;     m\_ownerIndex\[uint(\_owners\[i])] = 2 + i;\
&#x20;   }\
&#x20;   m\_required = \_required;\
&#x20; }\
&#x20; ...\
&#x20; // constructor - just pass on the owner array to multiowned and\
&#x20; // the limit to daylimit\
&#x20; function initWallet(address\[] \_owners, uint \_required, uint \_daylimit) {\
&#x20;   initDaylimit(\_daylimit);\
&#x20;   initMultiowned(\_owners, \_required);\
&#x20; }\
}

Note that neither of the functions specifies their visibility, so both default to `public`. The `initWallet` function is called in the wallet’s constructor, and sets the owners for the multisig wallet as can be seen in the `initMultiowned` function. Because these functions were accidentally left `public`, an attacker was able to call these functions on deployed contracts, resetting the ownership to the attacker’s address. Being the owner, the attacker then drained the wallets of all their ether.

#### Resources

[https://www.parity.io/the-multi-sig-hack-a-postmortem/](https://www.parity.io/the-multi-sig-hack-a-postmortem/)

#### Postmortem

#### The Multi-sig Hack: A Postmortem

[https://www.parity.io/the-multi-sig-hack-a-postmortem/](https://www.parity.io/the-multi-sig-hack-a-postmortem/)

On Wednesday 19th July, 2017 a bug found in the multi-signature wallet ("multi-sig") code used as part of Parity Wallet software was exploited by parties unknown. As of the time of writing, three wallet accounts holding large balances of ETH have been compromised and the balances moved into accounts held by the attacker. The self-titled "White Hat Group" used the same exploit to secure the other compromised wallets within Ethereum, with the stated intention of returning control to the original owners.

All other functionality of Parity, including all normal, non multi-sig accounts have no known vulnerabilities. To give some perspective: 3 multi-sig wallets were exploited from of a total of 596 vulnerable multi-sig wallets (the rest were commandeered by the White Hat Group), which themselves are a tiny fraction of Parity accounts.

The compromised accounts can be viewed on Etherscan. At the time of writing, the thief is attempting to launder the stolen funds through exchanges.

We already disabled the use of the broken code (it requires use of an on-chain registered resource which we were able to quickly unregister), which means future multi-sig wallets created in all versions of Parity Wallet have no known exploits. There are phishing attempts afoot; please ensure you download Parity only via our official Github repository, https://github.com/paritytech/parity.

#### WHAT WAS THE NATURE OF THE BUG?

The bug was in a pair of extremely sensitive functions designed to allow the set-up of "multi-sig" wallets in the Parity Wallet software.\
The functions should have been protected in order that they be usable only in one specific circumstance, as the contract was being created. However, they were entirely unguarded, which allowed the attacker to reset the ownership and usage parameters of existing wallets arbitrarily.

#### WAS THE WALLET NOT AUDITED?

The original "Foundation" multi-sig wallet code was created and audited by the Ethereum Foundation's DEV team, Parity and others from the community. It is used extensively and underwent extensive peer review. This body of code continues to have no known security issues.

#### WHAT CODE IS USED IN PARITY WALLET?

The code used in Parity Wallet is a modified form of the original multi-sig wallet code. It was restructured into a lightweight "stub" contract which is deployed to the network every time a wallet is created, together with a much heavier "library" contract, containing the majority of the wallet's logic and which is deployed only once. (By splitting the code in this way, deploying a wallet is substantially cheaper in terms of gas costs.)

The bug was introduced during this restructuring.

HOW DID THE BUG ESCAPE NOTICE?\
In general, we treat security and consensus code extremely seriously at Parity.

All code changes undergo at least one peer review which we manage through a largely manual system of tracking and tagging, commonly found in open source projects. Alterations that are understood to involve core code require sign-off by core developers; whereas changes to UI code require sign-off by the UI team. Changes which are understood to be sensitive (such as with contracts or core consensus code) are tagged as such and undergo a greater level of review, generally two or more expert reviews.

The restructuring of the original multi-sig wallet contract happened as part of a much larger change, namely the introduction of the Wallet user-interface, a 4000-line, almost entirely Javascript, CSS and HTML alteration. The depth and nature of changes made (and thus the severity of the potential bug) was misunderstood by the rest of the team. It was erroneously tagged as a (purely) UI change and thus received only one review before merge. A later audit by a Solidity expert also missed the issue.

Though the code was open and public, and thus the bug could have been discovered, reported and fixed before any damage done, there was no incentivisation mechanism to ensure good-natured eyes from the community inspected it.

WHAT WILL PARITY DO TO ENSURE THIS DOESN'T HAPPEN AGAIN?\
While there is no fool-proof means of practically ensuring software contains no bugs, Parity Technologies is committed to minimising the chances that its software contains exploits. In response to the present exploit we will refine our development processes and CI system.

The first and biggest change will be to ensure that any alterations to the codebase that involve live contract code (which can be generally identified through .sol files) be reviewed by Solidity experts. At present the multi-sig wallet is the only Solidity code that is user-deployable and in wide use within Parity.

Secondly, we will be revisiting the design of Parity Wallet's Multi-sig at a high level. Part of the cause of this bug, and the reason it went unnoticed, was due to the large amount of complexity within the preexisting and well-audited multi-sig wallet. We will consider adding an additional, extremely simple, contract to sit between the more complex multi-sig and any assets it controls. We have already started work on this and a draft of the idea can be found in our contracts repository, the contract named CoreVeto. It is essentially a very simple veto-able time-lock contract, which gives a last defence against these kinds of black swan events where the unknown unknowns can occur. Had funds been sitting behind this contract rather than in the multi-sig directly, no theft would have been possible.

Thirdly, there is wisdom in the idea that the best disinfectant is direct sunlight. Had there been a greater degree of community ownership over the wallet code, then the greater number of eyes may have seen the bug before anyone with bad intentions had managed to exploit it. To this end we will be rearchitecting the UI into a more general and lightweight platform with the idea that modules are well-isolated and more inviting to the casual developer.

Fourth, some blame for this bug lies with the Solidity language and, in its current incarnation, the difficulty with which one can understand the execution permissions over functions. We will submit some suggestions to the Solidity team that would help to minimise bugs of this kind. We believe one or both of two ideas would help. One would be to change the default access mode of functions to "private", rather than the eminently insecure "public". Another, even safer, change would be to make it illegal to include functions that have neither an explicit access modifier nor a guard modifier. Almost all functions in production Solidity code have one or both. Either of these rules would have resulted in the buggy multi-sig library not compiling. Having written this, I see Emin Gün Sirer has had a similar appraisal.

Going forward, Parity will try to arrange a bug-bounty programme. Unfortunately, since Parity is a small, minimally-funded start-up, we have not the resources to do this alone. Outside of a few tips to the shoe fund, Parity has received no funding whatsoever from any organisations within the Ethereum ecosystem. So we appeal to the community and those that use our software in their well-funded businesses and projects: help us set up a fund to help ensure this doesn't happen again.

### Resources

[https://programtheblockchain.com/posts/2018/01/02/making-smart-contracts-with-public-variables/](https://programtheblockchain.com/posts/2018/01/02/making-smart-contracts-with-public-variables/)\
[https://medium.com/swlh/ethereum-aint-hiding-your-secrets-703e89088937](https://medium.com/swlh/ethereum-aint-hiding-your-secrets-703e89088937)\
[https://solidity.readthedocs.io/en/develop/miscellaneous.html#layout-of-state-variables-in-storage](https://solidity.readthedocs.io/en/develop/miscellaneous.html#layout-of-state-variables-in-storage)\
[https://solidity.readthedocs.io/en/develop/contracts.html#visibility-and-getters](https://solidity.readthedocs.io/en/develop/contracts.html#visibility-and-getters)\
[https://github.com/paritytech/parity-ethereum/issues/6995#issuecomment-342409816](https://github.com/paritytech/parity-ethereum/issues/6995#issuecomment-342409816)
