# DID Protocols

## uPort

[https://**github.com/uport-project**](https://github.com/uport-project)

Reading

[Uport: Meta Transactions](https://medium.com/uport/making-uport-smart-contracts-smarter-part-3-fixing-user-experience-with-meta-transactions-105209ed43e0), i.e. users can interact with the Ethereum blockchain without holding any Ether.

[https://github.com/uport-project/uport-identity/blob/develop/contracts/TxRelay.sol](https://github.com/uport-project/uport-identity/blob/develop/contracts/TxRelay.sol)

News

[https://medium.com/uport/first-official-registration-of-a-zug-citizen-on-](https://medium.com/uport/first-official-registration-of-a-zug-citizen-on-ethereum-3554b5c2c238) [ethereum-3554b5c2c238](https://medium.com/uport/first-official-registration-of-a-zug-citizen-on-ethereum-3554b5c2c238)

uPort Connect

[https://developer.uport.me/categories/uport-connect](https://developer.uport.me/categories/uport-connect)

Single sign-on and transaction signing for your client-side app

uPort Credentials

[https://developer.uport.me/categories/uport-credentials](https://developer.uport.me/categories/uport-credentials)

Request, sign, and issue credentials from your app server

uPort Transports

[https://developer.uport.me/categories/uport-transports](https://developer.uport.me/categories/uport-transports)

Set up communication channels between your app and uPort clients.

Tools

EthrDID

[https://developer.uport.me/categories/ethr-did](https://developer.uport.me/categories/ethr-did)

Create Decentralized Identifiers and manage their interactions in your app.

Using the Ethr-DID library, you can:

* Create and manage keys for DID identities
* Sign JSON Web Tokens
* Authorize third parties to sign on a DID's behalf
* Enable discovery of service endpoints \(e.g. decentralized identity management

services\)

The Ethr-DID library conforms to ERC-1056 and supports the proposed Decentralized Identifiers spec from the W3C Credentials Community Group.

EthrDID Registry

[https://developer.uport.me/categories/ethr-did-registry](https://developer.uport.me/categories/ethr-did-registry)

Smart contract for the resolution and management of decentralized identifiers \(DIDs\)

## 3Box

3Box\`

**3Box** is a distributed database for Ethereum accounts that allows users to easily store and share public and encrypted information without needing to store any information on the blockchain. The 3box.io application allows Ethereum users to create a social profile, which they can use to collect data, log into Ethereum applications, and build connections.

Once we begin replacing hex identifiers with human-readable profiles consisting of names and images, we can generate social context and welcome a mainstream audience to Web3.

* **3Box-js** - [https://github.com/3box/3box-js/](https://github.com/3box/3box-js/)
* 3box-js is a JavaScript API that allows applications to integrate with 3Box, allowing them to onboard users and set/get data into 3Box profiles.

Blogs

3Box.js: 7 Use Cases for Social Dapps

[https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44](https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44)

Towards a More Human Web3

[https://medium.com/3box/towards-a-more-human-web3-553d6145a5b](https://medium.com/3box/towards-a-more-human-web3-553d6145a5b)

Announcing Ethereum ProfileS

[https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23) [f0316e15ce23](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23)

3Box.js

3Box.js: 7 Use Cases for Social Dapps

[https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44](https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44)

This page describes some of the ways that your application can use the 3Box.js library. For more detailed documentation see the [Github repo](https://github.com/3box/3box-js/).

1. Public User Profiles

Wouldn’t it be great if we could replace hex addresses with social user profiles in our dapps? 3Box is a system for social user profiles, and we make it easy to

discover other users.

The getProfile method allows you to get the public profile for one or more ethereum accounts at any time. This allows you to replace hex addresses with user names and images throughout your application , which provides users with a huge amount of social context. getProfile is also useful for onboarding new users, which we will describe in more detail in \#3.

The box.public.set\(\) and box.public.get\(\) methods allow you to set and get data in the public profile. For example, you might store community or group affiliations in the public profile .

1. Private Encrypted Storage

3Box provides encryption to Ethereum users. This powerful functionality e nables users and apps to store and exchange private information . This information will only ever be able to be read by those approved by the user. Encryption is not not currently supported by standard Ethereum wallets, but we can enable it which brings on a whole new set of Ethereum use cases.

The box.private.set\(\) and box.private.get\(\) methods allow you to set and get private encrypted data in 3Box. For example, you might store a private image in the user’s private profile .

1. User Onboarding

As an outcome of having \#1 and \#2 above, apps can easily onboard new users to their dapp by calling getProfile, box.public.get\(\), and box.private.get\(\) for public and private information. One nice thing about this system is that users can share all of their information with an application that requests it with one click, making filling out forms and sharing information a simplified experience.

1. Decentralized Messaging or Public Key Infrastructure \(PKI\) Applications can store public user communication keys and other verifiable information in 3Box and other users and applications can verify messages or claims against the user’s 3Box profile . This allows 3Box to act as a user’s very own decentralized public key infrastructure , making it trivial to implement encrypted messaging systems or verifiable identity credentials using 3Box.
2. Distributed User Content Storage and Management

You might also think about using 3Box as a generalized way to store and keep track of user generated content without needing to store the content hashes on the blockchain; these hashes would instead be kept with the user in either the public or private profile. These bits of content can represent everything from tweets and peeps, to videos and likes. All you need is the hash of an IPFS object and you can store it in 3Box: be creative!

1. Sharing Data Between Apps

One of the benefits of storing data with the user \( free of silos\) is that the data becomes more portable and interoperable wherever the user goes. This data can be shared with anyone or any application that calls getProfile, box.public.get\(\), or box.private.get\(\).

1. Shared Reputation Systems

One of the most obvious use cases for sharing data between applications is building shared reputation systems. We are certain that these will emerge as the decentralized application ecosystem matures and we need better and more sophisticated ways to provide context to our users. 3Box is the obvious place to store most of this data in an off-chain, distributed manner, with the user.

TODO Reading

Towards a More Human Web3

[https://medium.com/3box/towards-a-more-human-web3-553d6145a5b](https://medium.com/3box/towards-a-more-human-web3-553d6145a5b)

Announcing Ethereum ProfileS

[https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23) [f0316e15ce23](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23)

## Origin Protocol





At Origin, we’re building a platform for decentralized, peer-to-peer marketplaces. You can imagine a future Airbnb-like DApp allowing users to rent out their homes, accept payment in crypto currencies and save up to 25% in fees. But you don’t want to rent your home out to just anyone… you want to be sure the person is reputable and won’t trash your house.

This is no different in the ERC 725 world… we need to trust a third party to issue verifiable claims about someone, and we need a way to verify those claims on- chain. If we can do that, we can protect a contract by having it verify that a claim exists on an identity before allowing execution to continue. 

Like everything we build at Origin, the smart contracts and Identity Playground DApp are open-source and available to all under the MIT license.

### Example

Let’s walk through an example. Imagine there’s a DApp built on Origin called BlockBnb, which allows buyers to rent properties from sellers. For a buyer to interact with a BlockBnb listing contract, they must do so via an identity contract containing a valid claim issued by BlockBnb. The whole process would look like this:

* * Buyer deploys a new identity contract \(or reuses one they deployed earlier\)
  * Buyer visits blockbnb.com/verify and obtains a cryptographic signature proving that they control a particular email and phone number
  * Buyer adds this claim to their identity contract
  * Buyer tries to rent a property via a BlockBnb listing contract
  * Listing contract looks at buyer’s identity for a claim issued by BlockBnb
  * Listing contract recovers the public key from the claim signature and verifies it is still valid on the Blockbnb issuer contract
  * Transaction is allowed to proceed

Now that the buyer has a verified claim on their identity from BlockBnb, they can

interact with any other contracts also accepting claims issued by BlockBnb. Taking this a step further, imagine the US Postal Service deploy their own claim issuer contract, offering verifiable claims about an identity’s postal address. Any contract requiring a verified address can simply look for a claim issued by the US Postal Service contract before allowing interaction… and do so all on-chain.

DApp Demo

From a technical perspective, it’s nonintuitive but amazing to realize that there isn’t a traditional backend powering our DApp. The DApp runs entirely on client- side code \(HTML and JavaScript\) and doesn’t rely on centralized servers or private databases controlled by Origin or anyone. Thanks to the Ethereum blockchain and IPFS we can create marketplace experiences that are truly decentralized and available everywhere. As a result, our DApp can be run from any machine or directly accessed from any gateway on the IPFS network. Try it on [Infura](https://ipfs.infura.io/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/), [Siderus](https://siderus.io/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/), [Jes](https://ipfs.jes.xxx/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby), or [Origin’s own IPFS gateway](https://gateway.originprotocol.com/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/) .

* You can try the DApp at [**demo.originprotocol.com**](https://demo.originprotocol.com/)
* Usage instructions [https://medium.com/originprotocol/origin-demo-dapp-is-now-](https://medium.com/originprotocol/origin-demo-dapp-is-now-live-on-testnet-835ae201c58) [live-on-testnet-835ae201c58](https://medium.com/originprotocol/origin-demo-dapp-is-now-live-on-testnet-835ae201c58)

First, the foundation of our platform is a series of smart contracts running on the Ethereum Blockchain. These smart contracts enable functionality that replaces the need for trusted intermediaries to manage marketplaces, and allows us to instead create marketplaces that are governed by a set of open, fair and transparent rules.

Second, our Origin.js library gives developers a way to interact with our smart contracts without having to worry about the challenges of developing software on the blockchain. Origin.js is under active and rapid development, but there are already plenty of examples of how to use the library in [our documentation](https://docs.originprotocol.com/) that you can try for yourself.

Finally, our demo DApp is the first application that is built using Origin.js. This DApp is meant to illustrate the types of marketplaces that are possible and to give both developers and end users a chance to experiment with this new type of distributed marketplace.

We’ve crammed a lot of new features into this latest release, including support for user identity \(supporting the ERC 725 identity standard\), transaction steps, escrow, ratings, and reviews.

**Identity**

Origin \(the company, not the open-source platform\) is running an ERC 725 attestation service as part of our open-source bridge server. This means we can  

verify pieces of your identity like your email address, phone number, and social media accounts, and then sign attestations to be published on the blockchain. It’s a bit like using a notary service where you can choose which third parties you trust to verify your identity and/or the identities of others. If you add your name, profile picture, or bio we will publish that information to IPFS and store the resulting content hash on the Ethereum blockchain. We believe it would be irresponsible for us to encourage people to publish their phone numbers to an immutable database, so for more sensitive information like your phone number, email address, or social media usernames, we’ll only store an attestation to the existence of those accounts on the blockchain instead of the data itself.

Ultimately, the best privacy conscious solution will probably involve zero- knowledge proofs and Merkle trees.

**Transition progress &** **escrow**

Another big improvement to our platform is tracking transaction progress with state transitions. Our smart contracts keep funds escrowed until the buyer confirms receipt of the goods or services. Our DApp has been updated to show both the buyer and seller where they are in the purchase flow and the next steps that need to be taken .

**Ratings** **and** **reviews**

This feedback is now included as a core part of   Origin.js. On Origin, both buyers and sellers are required to leave reviews as part of the transaction flow. Buyers can be confident that reviews came from verified purchasers, and sellers can trust that they’re dealing with reputable buyers. We support 5-star ratings as well as an optional text-based reviews. Reviews are solicited for each completed transaction and can be aggregated for a specific user or product.

Resources

* live [ERC 725 identity playground](https://erc725.originprotocol.com/) that’s currently running on the Rinkeby test net.
* [GitHub repo](https://github.com/OriginProtocol/identity-playground)
* [Origin Demo Dapp](http://www.github.com/originprotocol/demo-dapp) \(github\)
* You can try the DApp at [**demo.originprotocol.com**](https://demo.originprotocol.com/)**.**
* [https://erc725.originprotocol.com/\#/](https://erc725.originprotocol.com/%23/)

Learn more about Origin:

* Buy and sell on our DApp: [dapp.originprotocol.com](http://dapp.originprotocol.com/)
* Create your own marketplace: [originprotocol.com/creator](https://www.originprotocol.com/creator)
* Connect with our team on Discord: [originprotocol.com/discord](http://originprotocol.com/discord)
* Learn more on our website: [originprotocol.com](http://originprotocol.com/)

