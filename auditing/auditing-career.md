# Auditing Career

## Your career as an auditor <a href="#your-career-as-an-auditor" id="your-career-as-an-auditor"></a>

So, you completed the course, and are fully capable of auditing any code that comes your way, but how do you start? Where can you find opportunities?

We know, the first steps are the hardest. In information security in general, nothing speaks more about you than your experience. This is why it's really tough to convince people to hire you when you've never audited before. Our Ethereum niche market is a bit more open, but still, possible clients will always ask about your previous work (they will often accept extensive Solidity development experience, though). How do you get past this and land your first gig?

We are glad that security within Ethereum is maturing every day, and with this the gateways to entry are also being created every now and then, here we will discuss the current main ones.

### Solidified <a href="#solidified" id="solidified"></a>

Of course we will start with Solidified, as you will possibly participate in an "internship" with us.

We started in late 2017 and since then have performed over 100 audits and 50 bug bounties. What sets us apart is our community-based company. In fact, you might already be part of our community-based bug bounties which are very open to participation.

Our audits are also community based. For every audit, we select three members of our community to audit the code. The auditors are members of the community we know and are sure will deliver top notch quality work. The auditors provide quotes that are normalized, and a 20% Solidified fee is added. This is the quote sent to the client. If the client agrees, the audit starts.

Once the audit starts, each of the three auditors will perform a full audit, as if he or she was the only one in the engagement. In the final debrief meeting, we get together in a Slack channel, and open the reports simultaneously. This process excels for the following reasons:

* Layering the various experiences and backgrounds of individual auditors ensures great coverage.
* Layering the personal processes of each auditor provides further coverage. We do not constrain our auditors to a particular process, so they can work with the tools they know and like the best.
* Opening reports simultaneously creates a healthy competition, as nobody wants to find fewer bugs than their peers. This motivates everyone to perform at their best level. Auditor performance is clear.
* Separate reports enables further analysis. For example, a difference in valid bugs found by auditors is a great indicator of possible unfound bugs and possibly undesirable confusion or complexity in the code.

The merged report, produced in the debrief meeting is the final report which is published [here](https://github.com/solidified-platform/audits).

We also include the follow up of the fixes within the price of the audit, so the audit is only completed after all fixes are done, and verified by the auditors. All follow ups are also documented in the report. This process takes from one to two weeks, and usually two rounds.

#### How do you become part of this group? <a href="#how-do-you-become-part-of-this-group" id="how-do-you-become-part-of-this-group"></a>

The easiest way is to apply in [our website](http://solidified.io/). We will schedule a chat with you, and if everything goes well and you have the required experience we will invite you for an internship audit, much like the one from the course (at 50% rate). Keep in mind this depends on the demand for audits at a given time: only low complexity audits are used.

We also talk to colleagues and developers at conferences, and have drafted users from our bug bounty platform a number of times.

## Bug Bounties <a href="#bug-bounties" id="bug-bounties"></a>

Bug bounties are another great way to get your foot through the door of the smart contract security market. They are great for a number of reasons, number one being no previous demonstrated experience is required. As long as you post a valid bug, you can expect to receive the bounty.

Other reasons are not to be overlooked:

* It will give you experience in auditing and will help you sharpen your teeth and slowly develop your own techniques. So at least you gain is experience, while possibly also getting a hefty bounty.
* Other people, mostly from the security community, will see your posts. This will often lead to opportunities. Think about this every time you post a bug or a question in a bug bounty. Be polite, be gentle, be clear and be meticulous.
* They can ultimately become very profitable if you find critical bugs :)

### Places to find them <a href="#places-to-find-them" id="places-to-find-them"></a>

* Solidified [web.solidified.io](https://web.solidified.io/): On Solidified, all bug bounties are smart-contract specific, so all you'll find are smart contracts. We encourage you to join, you will be informed of new bug bounties posted, and keep an eye on our social media. While at it, join our Slack if you haven't yet.
* [Consensys Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/bug\_bounty\_list/): Maintained by our friends from Diligence, you will find a list of bug bounties. Always check if they are still active before digging in the source code.
* Legacy Bug Bounty platforms: Platforms like Bugcrowd and HackerOne also have Ethereum smart contracts every now and then. They are usually within larger scoped bug bounties, including web, infrastructure and other bits, so looking for the familiar names from the Ethereum community is a good first step.
* Continuous bug bounties: All larger companies in the space conduct one, though they are not as well advertised. A couple of good things about them are that you won't be facing as much competition as in the regular temporary bug bounty and also that their scope is often much larger than that of the average bug bounty. You also have to keep in mind that the code is usually in production, and we know well that a very good indicator of the existence of bugs is the time the contracts remained unhacked while holding significant value. But this does not mean all contracts are bug free, as several bugs are found and reported sometimes years after deployment of the contracts. Check out this [amazing catch](https://samczsun.com/the-0x-vulnerability-explained) by Samczsun.
* [Ethdev Reddit](https://www.reddit.com/r/ethdev): Once the place where all bug bounties were posted. Nowadays they will still show up every now and then, worth to take a look when hunting for bounties.
* Set web filters: Web filters like [IFTTT](https://ifttt.com/) are a great way of finding bug bounties. Though the channels above will cover most of the Ethereum bug bounties, they can be published anywhere, so looking elsewhere can be fruitful. Remember that your knowledge also applies for blockchains other than Ethereum (as long as they run the EVM, like Ethereum Classic, Hyperledger Burrow, Quorum and others).

Lastly, remember to always look for reputable companies behind the bug bounties. Any company with a good reputation in the space will honor their commitment to pay the bounty. Avoid shady and unknown players and unknown tokens unless they are run out of a platform that values its reputation. If the bug bounty has been ongoing for a while, check previously reported bugs, and how they were handled by the organization.

## Nexus Mutual <a href="#nexus-mutual" id="nexus-mutual"></a>

You can earn staking rewards on Nexus Mutual by applying your skills as an auditor and vouching for smart contracts you think are secure.

Nexus Mutual is a platform that allows users of smart contracts to protect themselves against hacks or bugs, like insurance. It uses a staking system to determine the price, or risk, of a smart contract failing. The more value staked the lower the perceived risk and the lower the price of cover. The system is designed to encourage those with smart contract security expertise to vouch for the security of smart contract systems and be rewarded by doing so.

Generally, staking works as follows:

1. You stake against a specific smart contract system you think is secure.
2. When someone else purchases cover you receive rewards for staking.
3. If there is a claim paid as a result of a hack or bug then your stake can be lost.

One interesting aspect is Nexus effectively allows you to earn rewards if you don’t find any bugs. So the time you spent diving deeply into a bug bounty or production smart contract that didn’t result in any findings can now be used to earn rewards.

To begin staking you need to join [Nexus Mutual](https://www.nexusmutual.io/) as a member, which involves KYC and a small membership fee of 0.002 ETH. After that, you will need to purchase some of the native token, NXM, for staking purposes and then choose which systems to stake against.

The main benefit of Nexus is that you don’t need to secure a client to get started. You may also use any tools or techniques you wish to gain comfort that the smart contract is secure before staking.

## Find bugs anywhere :) <a href="#find-bugs-anywhere" id="find-bugs-anywhere"></a>

Yes, if you happen to find a bug in a production smart contract, you can contact the company / organization that owns it to report the bug, and you will often get a bounty for it; this is not guaranteed by any means, and sometimes will take other forms, such as being invited for a subsequent audit, for example.

If you do find an unexploited critical bug out there, it is your duty as a security professional to perform what we call [**responsible disclosure**](https://en.wikipedia.org/wiki/Responsible\_disclosure). This means finding the responsible party, and disclosing the issue in private, to allow for a fix, or reduction of impact, before the bug is communicated to a wider audience. This is performed for the obvious reason that this information would enable any party with enough technical knowledge, usually a lot lower than the knowledge required to find the bug, to exploit the bug. Once the bug is patched you should be free to publicize it.

## Conclusion <a href="#conclusion" id="conclusion"></a>

We'll use the conclusion to say that the list above is not conclusive. As our niche market matures, other ways of using the knowledge you just earned from this course are continuously appearing. Nexus Mutual, for example, has been active for less than a year, we plan on launching a bug prediction market in the future, and the same skills apply for them.

