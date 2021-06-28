# DIDs

## Introduction

Decentralized Identities\(DIDs\): DIDs are fully under the control of the DID subject, independent from any centralized registry, identity provider, or certiﬁcate authority.

"Decentralized identities are anchored by blockchain IDs linked to zero-trust datastores that are universally discoverable”.

Blockchain IDs

Gaining trust not from central authorities, but rather from inter-connected, yet fully decentralized networks of people, all choosing to participate in a Web of Trust.

Zero-trust datastores

A zero-trust datastore encompasses the ability to store private information locally, while maintaining trust and authenticity globally.

Using decentralized constructs, like an Ethereum Claims Registries, alongside the uPort SimpleSigner libraries, it’s possible to store encrypted data on a smartphone and pass that information back to servers and applications without losing trust in its authenticity. Save information locally. Share data globally. No middleman required.

Ethereum Claims Registry — Public Keys

The uPort team recently published \(2018\): • ERC780 EIP - ERC: Ethereum Claims Registry \#780 ◇ The Ethereum Claims Registry serve as the public registry for trust anchors.

Allowing distributed applications to publicly save/register a public key. • ERC1056 EIP - ERC: Lightweight Identity \#1056

Why is a public claims registry important?

Because distributed applications use the complimentary private signing key to generate attestations. When another application requests the privately signed information, from a private environment \(smartphone/userspace\) they need authenticate the information.

To verify the source of the signed information, it’s essential anyone can lookup the corresponding public key i.e. via the Ethereum Claims Registry using a decentralized identity resolver.

## uPort SimpleSigner — Private Keys

The uPort SimpleSigner allows trust anchors to privately sign attestations using the JSON Web Token \(JWT\) speciﬁcation. By using JWTs, instead of saving information on a public blockchain \(ERC725\) user’s private information can be kept conﬁdential… and private.

* ERC725 EIP -ERC Ethereum Identity Standard \(Proxy Account\) ◇ ERC 725 is a proposed standard for blockchain-based identity authored by Fabian Vogelsteller, creator of ERC 20 and Web3.js. ERC 725 describes proxy smart contracts that can be controlled by multiple keys and other smart contracts. ERC 735 is an associated standard to add and remove claims to an ERC 725 identity smart contract. These identity smart contracts can describe humans, groups, objects, and machines. ERC 725 lives on the Ethereum blockchain. ◇ ERC 725 allows for self-sovereign identity. An open, portable standard for identities will enable decentralized reputation, governance, and more. Users will be able to take their identity across different Dapps and platforms that support this standard.

### Universally Discoverable

As deﬁned in the Decentralized Identity speciﬁcation, the identities must be discoverable using traditional systems protocol speciﬁcations like URL or URI.

[Decentralized Identiﬁers \(DIDs\) -](https://w3c-ccg.github.io/did-spec/) [https://w3c-ccg.github.io/did-spec/](https://w3c-ccg.github.io/did-spec/) - Data Model and Syntaxes for Decentralized Identiﬁers \(DIDs\)

[3Box -](https://github.com/3box/3box) [https://github.com/3box/3box](https://github.com/3box/3box) [-](https://3box.io/) [https://3box.io/](https://3box.io/) - A Distributed DB for Ethereum users

## Self-sovrreign ID vs ID on blockchain

Identity unlocks formal services as diverse as voting, ﬁnancial account ownership, loan applications, business registration, land titling, social protection payments, and school enrollment.

### Identity problems

First, despite progress in some areas of the world, many people still lack formal identiﬁcation.

Functional IDs: any given person may have a variety of functional IDs \(e.g., driver’s license, health insurance card, voter registration card\).

The centralization of personal data in DID databases can increase privacy threats.

### Trends

Advances in biometrics Blockchain ID

Many proposed blockchain-backed ID systems are examples of “accretionary ID,” where an ID is built up over time through a series of interactions with others Other blockchain ID applications simply use a blockchain ledger as the back-end

database for a more traditional ID system. This may improve ransparency, but the differences will likely be invisible to typical users.

User-controlled ID \(self-sovreign\)

In contrast to systems where institutions provide ID credentials, user-controlled IDs build on the premise that people will control the formalization of their identity.

### Blockchain-Backed ID

#### What is it? Blockchain-backed ID can refer to:

An accretionary ID, where an identity is built up over time through a series of transactions stored on a blockchain and veriﬁed by others

Use of a blockchain-based distributed ledger platform as the back-end database for a more traditional ID system

Use of a blockchain-based distributed ledger platform to log transactions linked to a previously established identity Example use cases

Economic ID: Provides a permanent, accessible record of transaction history Humanitarian cash transfers: Eliminates opportunities to falsely claim assets to which someone else is entitled

Land titling: Securely maintains important records What problems can it solve? Resilience: Blockchain records are permanent and accessible from any participating node, offering protection from destruction and loss

Persistence: Blockchain records are very diﬃcult to alter; this protects data integrity Corruptibility of Centralized Authority: Democratizing reading and writing to database bypasses role of institutions What problems does it NOT solve?

Data Validity: Any documents or assets stored using blockchain need to be veriﬁed through other means; the integrity of the data is only protected after it is entered

### User-Controlled ID What is it?

An approach rather than one technology • Increases individual control over identity information, management, and sharing Example use cases

DigiLocker, Yoti, Sovrin and Hyperledger Indy What problems can it solve? Allows users to share only data needed and no more, better preserving privacy What problems does it NOT solve?

Requires high levels of digital literacy and ability to manage personal data Policy/regulatory environment may not provide suﬃcient protections for users \(e.g., clear deﬁnitions of data ownership, means of recourse in the case of data loss or breach\)

Clear standards to judge the reliability of a self-asserted ID in terms similar to institutionally granted IDs What problems could it create?

Signiﬁcant and poorly understood dependencies \(e.g., digital infrastructure, digital literacy\)

Ineﬃcient or ineffective investment in a premature technology solution What is current state of play?

There are several apps that incorporate elements of user-controlled systems catering to high-income contexts, generally very early stages

To our knowledge, there are no examples of fully user-controlled systems to date

## Resources

[Uport - Basics of](https://medium.com/uport/the-basics-of-decentralized-identity-d1ff01f15df1) [DiD](http://identity.foundation/) [DID Foundation](http://identity.foundation/)

[IDENTITY IN A DIGITAL](https://www.usaid.gov/sites/default/files/documents/15396/IDENTITY_IN_A_DIGITAL_AGE.pdf) [AGE](https://www.usaid.gov/sites/default/files/documents/15396/IDENTITY_IN_A_DIGITAL_AGE.pdf)

ERC-1484

[https://github.com/NoahZinsmeister/ERC-1484](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md) [https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md](http://eips.ethereum.org/EIPS/eip-1484) [http://eips.ethereum.org/EIPS/eip-1484](http://eips.ethereum.org/EIPS/eip-1484)

ERC-1056

## ERC 1056 - Lightweight Identity

ERC: Lightweight Identity \#1056 Simple Summary

A registry for key and attribute management of lightweight blockchain identities.

Resources

[https://medium.com/uport/erc1056-erc780-an-open-identity-and-claims-protocol- for-](https://medium.com/uport/erc1056-erc780-an-open-identity-and-claims-protocol-for-ethereum-aef7207bc744) [ethereum-aef7207bc744](https://medium.com/uport/erc1056-erc780-an-open-identity-and-claims-protocol-for-ethereum-aef7207bc744)

Abstract

[This ERC describes a standard for creating and updating identities with a limited use of blockchain resources. An identity can have an unlimited number of delegates and attributes associated with it. Identity creation is as simple as creating a regular key pair ethereum account, which means that it's fee \(no gas costs\) and all ethereum accounts are valid identities. Furthermore this ERC is fully](https://w3c-ccg.github.io/did-spec/) DID compliant.

Motivation

As we have been developing identity systems for the last couple of years at uPort [it has become apparent that the cost of identity creation is a large issue. The previous Identity proposal](https://github.com/ethereum/EIPs/issues/725) [ERC725](https://github.com/ethereum/EIPs/issues/725) faces this exact issue. Our requirements when creating this ERC is that identity creation should be free, and should be possible

to do in an oﬄine environment \(e.g. refugee scenario\). However it must also be possible to rotate keys without changing the primary identiﬁer of the identity.

The identity system should be ﬁt to use off-chain as well as on-chain. Deﬁnitions

Identiﬁer: a piece of data that uniquely identiﬁes the identity, an ethereum address delegate: an address that is delegated for a speciﬁc time to perform some sort of function on behalf of an identity

delegateType: the type of a delegate, is determined by a protocol or

application higher up Examples:

did-jwt raiden

attribute: a piece of data associated with the identity

Speciﬁcation

This ERC speciﬁes a contract called EthereumDIDRegistry that is deployed once and can then be commonly used by everyone.

Identity ownership

By default an identity is owned by itself, meaning whoever controls the ethereum account with that address. The owner can be updated to a new key pair account or to a multisig account etc.

identityOwner

Returns the owner of the given identity. changeOwner

Sets the owner of the given identity to another ethereum account. changeOwnerSigned

Same as above but with raw signature. Delegate management

Delegates can be used both on- and off-chain. They all have a delegateType

which can be used to specify the purpose of the delegate. validDelegate

Returns true if the given delegate is a delegate with type delegateType of identity.

addDelegate

Adds a new delegate with the given type. validity indicates the number of

seconds that the delegate will be valid for, after which it will no longer be a delegate of identity.

addDelegateSigned

Same as above but with raw signature. revokeDelegate

Revokes the given delegate for the given identity. revokeDelegateSigned

Same as above but with raw signature. Attribute management

Attributes contain simple data about the identity. They can be managed only by the owner of the identity.

setAttribute

Sets an attribute with the given name and value, valid for validity seconds. setAttributeSigned

Same as above but with raw signature. revokeAttrubte

Revokes an attribute. revokeAttributeSigned

Same as above but with raw signature. Events

DIDOwnerChanged

MUST be triggered when changeOwner or changeOwnerSigned was successfully called. event DIDOwnerChanged\( address indexed identity, address owner,

uint previousChange

\);

DIDDelegateChanged

MUST be triggered when a change to a delegate was successfully made. event DIDDelegateChanged\(

address indexed identity, bytes32 delegateType, address delegate, uint validTo,

uint previousChange

\);

DIDAttritueChanged

MUST be triggered when a change to an attribute was successfully made. event DIDAttributeChanged\(

address indexed identity, bytes32 name, bytes value, uint validTo,

uint previousChange

\);

Eﬃcient lookup of events through linked identity events

Contract Events are a useful feature for storing data from smart contracts exclusively for off-chain use. Unfortunately current ethereum implementations provide a very ineﬃcient lookup mechanism. By using linked events that always link to the previous block with a change for the identity, we can solve this problem with much improved performance. Each identity has its previously changed block stored in the changed mapping.

1. Lookup previousChange block for identity
2. Lookup all events for given identity address using web3, but only for the previousChange block
3. Do something with event
4. Find previousChange from the event and repeat Example code: const history = \[\]

previousChange = await didReg.changed\(identity\) while \(previousChange\) {

const ﬁlter = await didReg.allEvents\({topics: \[identity\], fromBlock: previousChange, toBlock: previousChange}\)

const events = await getLogs\(ﬁlter\) previousChange = undeﬁned for \(let event of events\) { history.unshift\(event\)

previousChange = event.args.previousChange

}

}

Building a DID document for an identity

The primary owner key should be looked up using identityOwner\(identity\). This should be the ﬁrst of the publicKeys listed. Iterate through the DIDDelegateChanged events to build a list of additional keys and authentication sections as needed. The list of delegateTypes to include is still to be determined. Iterate through DIDAttributeChanged events for service entries,

encryption public keys and other public names. The attribute names are still to be determined.

Rationale

For on-chain interactions Ethereum has a built in account abstraction that can be used regardless of whether the account is a smart contract or a key pair. Any transaction has a msg.sender as the veriﬁed send of the transaction.

Since each Ethereum transaction has to be funded, there is a growing trend of on- chain transactions that are authenticated via an externally created signature and not by the actual transaction originator. This allows 3rd party funding services or receiver pays without any fundamental changes to the underlying Ethereum architecture. These kinds of transactions have to be signed by an actual key pair and thus can not be used to represent smart contract based Ethereum accounts.

We propose a way of a Smart Contract or regular key pair delegating signing for various purposes to externally managed key pairs. This allows a smart contract to be represented both on-chain as well as off-chain or in payment channels through temporary or permanent delegates.

Backwards Compatibility

[All ethereum accounts are valid identities \(and DID compatible\) using this standard. This means that any wallet provider that uses key pair accounts already supports the bare minimum of this standard, and can implement delegate and attribute functionality by simply using the ethr-did referenced below. As the DID Auth standard solidiﬁes it also means that all of these wallets will be compatible with the](https://github.com/decentralized-identity) [DID decentralized login](https://github.com/decentralized-identity) system.

Implementation

[ethr-did-registry](https://github.com/uport-project/ethr-did-registry/blob/develop/contracts/EthereumDIDRegistry.sol) [\(EthereumDIDRegistry contract implementation\)](https://github.com/uport-project/ethr-did-resolver) [ethr-did-resolver](https://github.com/uport-project/ethr-did-resolver) \(DID compatible resolver\)

[ethr-did](https://github.com/uport-project/ethr-did) \(javascript library for using the identity\) Deployment

The address for the EthereumDIDRegistry will be speciﬁed here once deployed.

ERC-780

Abstract

This text describes a proposal for an Ethereum Claims Registry \(ECR\) which allows persons, smart contracts, and machines to issue claims about each other, as well as self issued claims. The registry provides a ﬂexible approach for claims that makes no distinction between different types of Ethereum accounts. The goal of the registry is to provide a central point of reference for on-chain claims on Ethereum.

Motivation

On-chain claims is becoming increasingly relevant as lots of different smart contracts might want to verify certain attributes about its users. However that is only one out of a very large space of use cases for on-chain claims. By providing a central repository for claims, developers are equipped with a common ground for experimentation. A standardized registry also makes claim lookup simple and gas eﬃcient. Third party contracts only need to make one external call, no need for adding logic for verifying signatures, lookup identity signing keys, etc.

Speciﬁcation

The ECR is a contract that is deployed once and can then be commonly used by everyone. Therefore it's important that the code of the registry has been reviewed by lots of people with different use cases in mind. The ECR provides an interface for adding, getting, and removing claims. Claims are issued from an issuer to a subject with a key, which is of type bytes32. The claims data is stored as type bytes32.

Claim types

The key parameter is used to indicate the type of claim that is being made. There are three ways that are encuraged for use in the ECR: • Standardised claim types use syntax borrowed from HTTP and do not start with X-. The key is the hash of the claim type \(eg, keccak256\('Owner-Address'\)\) • Private types not intended for interchange use the same syntax, but with X- preﬁx. The key is the hash of the claim type \(eg, keccak256\('X-My-

Thing'\)\) • Ad-hoc types use 32 random bytes for the key, enabling allocation of new globally used keys without the need for standardisation ﬁrst.

Standard claim types New claim types can be added by making a PR to modify this table.

\| Claim type \| Description \| \| ERC780Example \| This is an example \| Registry speciﬁcation

The ECR provides the following functions:

setClaim Used by an issuer to set the claim value with the key about the subject.

function setClaim\(address subject, bytes32 key, bytes32 value\) public;

setSelfClaim Convenience function for an issuer to set a claim about themself.

function setSelfClaim\(bytes32 key, bytes32 value\) public;

getClaim Used by anyone to get a speciﬁc claim.

function getClaim\(address issuer, address subject, bytes32 key\) public const

removeClaim Used by an issuer to remove a claim it has made.

function removeClaim\(address issuer, address subject, bytes32 key\) public;

Type conversions The value parameter was choosen to have the type bytes32. This in order to make the registry entries as general as possible while maintaining a very simple code

base. However it is likely that usecases where other types such as address and uint etc. will emerge. Support for this can be added in various ways. We suggest that a library is implemented that can covert between various solidity types. This means that the registry itself keeps its simplicity. Contracts that need speciﬁc types can use such a library to convert the bytes32 into their desired type and back.

Deployment After some discussion on the design of the registry contract it will be deployed and its address should be written in this document. This should include the addresses for the ropsten, rinkeby, and kovan testnets as well.

Updates and governance In the future there might be new features needed in the registry. In order for such an event to be as transparent as possible the new features should be proposed and approved in a new EIP that contains the new contract code as well as a way of migrating any old claims if deemed necessary.

Appendix: Registry implementation

1 contract EthereumClaimsRegistry {

2

3 mapping\(address =&gt; mapping\(address =&gt; mapping\(bytes32 =&gt; bytes32\)\)\) pu

4

1. event ClaimSet\(
2. address indexed issuer,
3. address indexed subject,
4. bytes32 indexed key,
5. bytes32 value,
6. uint updatedAt\);

11

1. event ClaimRemoved\(
2. address indexed issuer,
3. address indexed subject,
4. bytes32 indexed key,
5. uint removedAt\);

17

1. // create or update clams
2. function setClaim\(address subject, bytes32 key, bytes32 value\) public
3. registry\[msg.sender\]\[subject\]\[key\] = value;
4. emit ClaimSet\(msg.sender, subject, key, value, now\);

22 }

23

1. function setSelfClaim\(bytes32 key, bytes32 value\) public {
2. setClaim\(msg.sender, key, value\);

26 }

27

28

29

30

31

32

33

34

35

36

37

function getClaim\(address issuer, address subject, bytes32 key\) public return registry\[issuer\]\[subject\]\[key\];

}

function removeClaim\(address issuer, address subject, bytes32 key\) pub require\(msg.sender == issuer\);

delete registry\[issuer\]\[subject\]\[key\];

emit ClaimRemoved\(msg.sender, subject, key, now\);

}

}

uPort Claims

ERC1056 describing an identity system for ethereum compliant with the DID standard. Decentralized Identiﬁers \(or DIDs\) are the core component of self-sovereign identity systems. Aside from DIDs a fundamental building block of a secure user-centric identity is veriﬁable claims. A veriﬁable claim consists of two parts. A claim, which is a statement, i.e. Joel is the author of this article, and a signature from the identity that makes the claim. In the uPort protocol there are two types of claims, on-chain \(using ERC780\) and off-chain \(using JWTs\).

A lightweight identity system

ERC1056 describes an identity system in which all ethereum accounts are valid DIDs.

All identities can assign delegates which are valid for a speciﬁed amount of time and that can be either revocable or not. A delegate can be used both on- and off-chain to sign on behalf of the identity that assigned it. This includes signing of JWTs, ethereum transactions, as well as arbitrary data.

Using delegates in state channels

An interesting use case for delegates is state channels. The user’s identity lives on the phone, but user could assign a delegate to act on its behalf in the browser. Note that this requires the delegate to be added as non-revocable. Otherwise the owner of the identity could revoke the key before the state channel settles making all off-chain transactions invalid.

Veriﬁable claims

ERC780 is useful not only for on-chain claims but has features that are helpful for off-chain claims as well.

Off-chain claims and user privacy

By creating off-chain claims that are signed by a DID that is anchored on-chain, claims can remain private while retaining the security of the blockchain.

The issuer simply signs some data and encodes it into a format called Json Web Tokens \(JWT\). These JWT can later be veriﬁed by anyone. For more details on how JWTs work take a look at our did-jwt repo.

On-chain “Badges”

ERC780 is an api for issuing, revoking, and looking up claims on-chain. But what is an on- chain claim really? I like to contrast claims \(or badges\) with non-fungible tokens \(NFT\). A NFT has dynamic ownership and you can send it to anyone. Badges on the other hand are given to an identity and can never change hands. It could be an indicator of a particular achievement or status within a group etc.

ERC-735

[https://github.com/ethereum/EIPs/issues/735](https://github.com/ethereum/EIPs/issues/735)

ERC-725

## Introduction

We will use the term “proxy identity” or “identity” in short to refer to a smart contract acting on behalf of its owner\(s\), such as humans, groups, objects, and machines and controlled by keys. I.e. anyone or anything can use a cryptographic key pair to deploy and control a proxy identity contract on the Ethereum blockchain.

[A proxy identity is much better than just a key pair because all of the functionality needed to manage keys throughout their lifecycle is already programmed inside the smart contract, using the standard interface described in](https://github.com/ethereum/EIPs/issues/725) ERC725.

Managing keys using smart contracts enables them to become objects with properties which can be manipulated by their owners just like computer programs. Proxy identities will mainly do the following actions on behalf of their owner\(s\):

Store – information \(claims signed by other users/identities\) about the identity itself, and a list of other keys allowed to manage the identity or execute actions on its behalf. Execute – calls to other smart contracts or send ether/tokens. Can only be executed by owner or authorized keys.

Sign – claims about other identities or keys.

Verify – claims made by other identities to other identities. For example, a KYC proof can be signed by an authorized KYC provider \(the issuer\), about a subject \(prover\) who then presents that proof to a smart contract or person \(veriﬁer\) in order to prove their identity.

Encrypt – data to hold in claims.

Both authentication and authorization between users of this system are enforced by the smart contract’s logic, instead of through a centralized database. This decentralization allows all users \(both humans and machines\) to reliably verify information about each other, in other words this allows mutual authentication between any two users \(identities\), without relying on any centralized infrastructure.

This concept is called “decentralized identity”3 or “self-sovereign identity”, because all users are in possession of the key\(s\) they use to identify themselves, sign claims about other

users’ identities, and execute operations through smart contracts. This puts users in control of their identity, allowing individuals to maintain full control over their privacy, as well as decide how and what data is shared. It enables people to monetize their personal information themselves.

1

2

3

4

5

6

7

// 1: MANAGEMENT keys, which can manage the identity uint256 public constant MANAGEMENT\_KEY = 1;

// 2: EXECUTION keys, which perform actions in this identities name \(signi uint256 public constant EXECUTION\_KEY = 2;

8

9

10

11

// 3: CLAIM signer keys, used to sign claims on other identities which nee uint256 public constant CLAIM\_SIGNER\_KEY = 3;

// 4: ENCRYPTION keys, used to encrypt data e.g. hold in claims.

uint256 public constant ENCRYPTION\_KEY = 4;

As already mentioned, identities can represent both individuals and entities such as businesses. These entities can be represented as an ERC725 smart contract with multiple owners. Keys can be assigned a speciﬁc clearance level or purpose; the following example shows an identity contract conﬁguration where any added key can be assigned up to four different purposes based on their access level:

In the above example, the management key has clearance level 1, meaning it can also do any of the actions with clearance levels above one. This is just an example, anyone can arbitrarily code access control logic into their smart contract identities by assigning a different purpose to each key.

## What is ERC 725?

ERC 725 is a proposed standard for blockchain-based identity authored by Fabian Vogelsteller, creator of ERC 20 and Web3.js. ERC 725 describes proxy smart contracts that can be controlled by multiple keys and other smart contracts. ERC 735 is an associated standard to add and remove claims to an ERC 725 identity smart contract. These identity

smart contracts can describe humans, groups, objects, and machines. ERC 725 lives on the Ethereum blockchain.

[“ERC-725” refers to a speciﬁc smart contract design that acts as the individual’s core identity manager on the blockchain . The fundamental logic of the](https://github.com/ethereum/EIPs/issues/725) [ERC-725](https://github.com/ethereum/EIPs/issues/725) [gives the individual the ability to associate themselves with signing and encryption keys, but the ERC- 725 also includes logic for executing and approving Ethereum transactions and is designed ﬂexibly to be used in combination with a claims contract \(e.g.](https://github.com/ethereum/EIPs/issues/735) ERC-735[,](https://github.com/ethereum/EIPs/issues/780) ERC-780\) to expand the types of identity data recorded on the blockchain.

## Why ERC 725?

ERC 725 allows for self-sovereign identity. Users should be able to own and manage their identity instead of ceding ownership of identity to centralized organizations. We have seen the negative effects of having centralized identity with damaging leaks and unfair selling of user data and identity. An open, portable standard for identities will enable decentralized reputation, governance, and more. Users will be able to take their identity across different Dapps and platforms that support this standard.

Foundational reading

[ERC-725](https://github.com/ethereum/EIPs/issues/725) [EIP](https://github.com/ethereum/EIPs/issues/725)

[Origin Protocol: ERC-725 Implementation & UI](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09) -

[Origin DApp gets ERC 725 Identity, Transaction Steps, Escrow, and](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5) [Reviews Matthew De Silva: A Self-Sovereign Identity Standard For](https://www.ethnews.com/erc725-a-self-sovereign-identity-standard-for-ethereum) [Ethereum](https://www.ethnews.com/erc725-a-self-sovereign-identity-standard-for-ethereum)

[Fractal Blockchain: First impressions with ERC-725 and ERC-735 — identity](https://hackernoon.com/first-impressions-with-erc-725-and-erc-735-identity-and-claims-4a87ff2509c9) an[d](https://hackernoon.com/first-impressions-with-erc-725-and-erc-735-identity-and-claims-4a87ff2509c9) [claims](https://hackernoon.com/first-impressions-with-erc-725-and-erc-735-identity-and-claims-4a87ff2509c9)

-

[Dream AC: QA with Fabian Vogelsteller, author of ERC-20,](https://blog.dream.ac/qa-with-dream-advisor-fabian-vogelsteller-author-of-erc20-erc725/) [ERC-725 Crypto Slate: self-sovereign identity management on the](https://cryptoslate.com/what-is-erc725-self-sovereign-identity-management-on-the-blockchain/) [blockchain](https://erc725alliance.org/) [https://erc725alliance.org/](https://erc725alliance.org/)

[https://identity.foundation/](https://identity.foundation/)

ERC-725 Timeline

[Oct 2017 —](https://github.com/ethereum/EIPs/issues/725) [ERC-725](https://github.com/ethereum/EIPs/issues/725) standardproposed by Fabian Vogelstellar

[Feb 2018 —](https://github.com/JosefJ/IdentityContract)[JosefJ writes initial smart contracts for](https://github.com/JosefJ/IdentityContract) ERC-725 [Apr 2018 —](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09)[Origin Protocol ERC-725 implementation and](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09) [UI](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09)

[Apr 2018 —](https://github.com/mirceapasoi/erc725-735)[Mircea Pasoi from Coinbase implements ERC-725/735](https://github.com/mirceapasoi/erc725-735) contracts [May 2018 —](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5)[Origin Protocol implements ERC-725 on the Origin](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5) [dApp](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5)

[Aug 2018 —](https://erc725alliance.org/)[ERC-725 Alliance](https://erc725alliance.org/) [formed and website launched](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09) [Origin Protocol: ERC-725 Implementation & UI](https://medium.com/originprotocol/managing-identity-with-a-ui-for-erc-725-5c7422b38c09) -

[Origin DApp gets ERC 725 Identity, Transaction Steps, Escrow, and](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5) [Reviews](https://medium.com/originprotocol/tech-update-dapp-gets-erc-725-identity-transaction-steps-escrow-reviews-298a7c91b7a5)

uPort

uPort

[https://](https://github.com/uport-project)github.com/uport-project Reading

[Uport: Meta](https://medium.com/uport/making-uport-smart-contracts-smarter-part-3-fixing-user-experience-with-meta-transactions-105209ed43e0) Transactions, i.e. users can interact with the Ethereum blockchain without holding any Ether.

https://github.com/uport-project/uport-identity/blob/develop/contracts/TxRelay.sol News

[https://medium.com/uport/ﬁrst-oﬃcial-registration-of-a-zug-citizen-on- ethereum-](https://medium.com/uport/first-official-registration-of-a-zug-citizen-on-ethereum-3554b5c2c238) [3554b5c2c238](https://medium.com/uport/first-official-registration-of-a-zug-citizen-on-ethereum-3554b5c2c238)

[uPort Connect](https://developer.uport.me/categories/uport-connect) [https://developer.uport.me/categories/uport-connect](https://developer.uport.me/categories/uport-connect)

Single sign-on and transaction signing for your client-side app uPort Credentials

https://developer.uport.me/categories/uport-credentials [Request, sign, and issue credentials from your app server uPort Transports](https://developer.uport.me/categories/uport-transports) [https://developer.uport.me/categories/uport-transports](https://developer.uport.me/categories/uport-transports)

Set up communication channels between your app and uPort clients. Tools

[EthrDID](https://developer.uport.me/categories/ethr-did) [https://developer.uport.me/categories/ethr-did](https://developer.uport.me/categories/ethr-did)

Create Decentralized Identiﬁers and manage their interactions in your app. Using the Ethr-DID library, you can:

Create and manage keys for DID identities Sign JSON Web Tokens

Authorize third parties to sign on a DID's behalf

Enable discovery of service endpoints \(e.g. decentralized identity management

services\)

The Ethr-DID library conforms to ERC-1056 and supports the proposed Decentralized Identiﬁers spec from the W3C Credentials Community Group.

[EthrDID Registry](https://developer.uport.me/categories/ethr-did-registry) [https://developer.uport.me/categories/ethr-did-registry](https://developer.uport.me/categories/ethr-did-registry)

Smart contract for the resolution and management of decentralized identiﬁers \(DIDs\)

3Box

3Box\`

#### 3Box is a distributed database for Ethereum accounts that allows users to easily store and share public and encrypted information without needing to store any information on the blockchain. The 3box.io application allows Ethereum users to create a social proﬁle, which they can use to collect data, log into Ethereum applications, and build connections.

Once we begin replacing hex identiﬁers with human-readable proﬁles consisting of names and images, we can generate social context and welcome a mainstream audience to Web3.

3Box-js [-](https://github.com/3box/3box-js/) [https://github.com/3box/3box-js/](https://github.com/3box/3box-js/)

3box-js is a JavaScript API that allows applications to integrate with 3Box, allowing them to onboard users and set/get data into 3Box proﬁles.

Blogs

3Box.js: 7 Use Cases for Social Dapps

https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44 Towards a More Human Web3

https://medium.com/3box/towards-a-more-human-web3-553d6145a5b Announcing Ethereum ProﬁleS

[https://medium.com/3box/announcing-ethereum-proﬁles-1-0-0-is-live-](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23) f0316e15ce23 3Box.js

3Box.js: 7 Use Cases for Social Dapps

[https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44](https://medium.com/3box/3box-js-7-use-cases-for-social-dapps-e6263787ba44)

[This page describes some of the ways that your application can use the 3Box.js library. For more detailed documentation see the](https://github.com/3box/3box-js/) [Github](https://github.com/3box/3box-js/) repo.

1. Public User Proﬁles

Wouldn’t it be great if we could replace hex addresses with social user proﬁles in our dapps? 3Box is a system for social user proﬁles, and we make it easy to

discover other users.

The getProﬁle method allows you to get the public proﬁle for one or more ethereum accounts at any time. This allows you to replace hex addresses with user names and images throughout your application , which provides users with a huge amount of social context. getProﬁle is also useful for onboarding new users, which we will describe in more detail in \#3.

The box.public.set\(\) and box.public.get\(\) methods allow you to set and get data in the public proﬁle. For example, you might store community or group aﬃliations in the public proﬁle .

1. Private Encrypted Storage

3Box provides encryption to Ethereum users. This powerful functionality e nables users and apps to store and exchange private information . This information will only ever be able to be read by those approved by the user. Encryption is not not currently supported by standard Ethereum wallets, but we can enable it which brings on a whole new set of Ethereum use cases.

The box.private.set\(\) and box.private.get\(\) methods allow you to set and get private encrypted data in 3Box. For example, you might store a private image in the user’s private proﬁle .

1. User Onboarding

As an outcome of having \#1 and \#2 above, apps can easily onboard new users to their dapp by calling getProﬁle, box.public.get\(\), and box.private.get\(\) for public and private information. One nice thing about this system is that users can share all of their

information with an application that requests it with one click, making ﬁlling out forms and sharing information a simpliﬁed experience.

1. Decentralized Messaging or Public Key Infrastructure \(PKI\) Applications can store public user communication keys and other veriﬁable information in 3Box and other users and applications can verify messages or claims against the user’s 3Box proﬁle . This allows 3Box to act as a user’s very own decentralized public key infrastructure , making it trivial to implement encrypted messaging systems or veriﬁable identity credentials using 3Box.
2. Distributed User Content Storage and Management

You might also think about using 3Box as a generalized way to store and keep track of user generated content without needing to store the content hashes on the blockchain; these hashes would instead be kept with the user in either the public or private proﬁle. These bits of content can represent everything from tweets and peeps, to videos and likes. All you need is the hash of an IPFS object and you can store it in 3Box: be creative!

1. Sharing Data Between Apps

One of the beneﬁts of storing data with the user \( free of silos\) is that the data becomes more portable and interoperable wherever the user goes. This data can be shared with anyone or any application that calls getProﬁle, box.public.get\(\), or box.private.get\(\).

1. Shared Reputation Systems

One of the most obvious use cases for sharing data between applications is building shared reputation systems. We are certain that these will emerge as the decentralized application ecosystem matures and we need better and more sophisticated ways to provide context to our users. 3Box is the obvious place to store most of this data in an off-chain, distributed manner, with the user.

TODO Reading

Towards a More Human Web3

https://medium.com/3box/towards-a-more-human-web3-553d6145a5b Announcing Ethereum ProﬁleS

[https://medium.com/3box/announcing-ethereum-proﬁles-1-0-0-is-live-](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23) [f0316e15ce23](https://medium.com/3box/announcing-ethereum-profiles-1-0-0-is-live-f0316e15ce23)

Origin Protocol

At Origin, we’re building a platform for decentralized, peer-to-peer marketplaces. You can imagine a future Airbnb-like DApp allowing users to rent out their homes, accept payment in crypto currencies and save up to 25% in fees. But you don’t want to rent your home out to just anyone… you want to be sure the person is reputable and won’t trash your house.

This is no different in the ERC 725 world… we need to trust a third party to issue veriﬁable claims about someone, and we need a way to verify those claims on- chain. If we can do that, we can protect a contract by having it verify that a claim exists on an identity before allowing execution to continue.

Like everything we build at Origin, the smart contracts and Identity Playground DApp are open-source and available to all under the MIT license.

## Example

Let’s walk through an example. Imagine there’s a DApp built on Origin called BlockBnb, which allows buyers to rent properties from sellers. For a buyer to interact with a BlockBnb listing contract, they must do so via an identity contract containing a valid claim issued by BlockBnb. The whole process would look like this:

Buyer deploys a new identity contract \(or reuses one they deployed earlier\) Buyer visits blockbnb.com/verify and obtains a cryptographic signature proving that they control a particular email and phone number

Buyer adds this claim to their identity contract

Buyer tries to rent a property via a BlockBnb listing contract

Listing contract looks at buyer’s identity for a claim issued by BlockBnb

Listing contract recovers the public key from the claim signature and veriﬁes it is still valid on the Blockbnb issuer contract

Transaction is allowed to proceed

Now that the buyer has a veriﬁed claim on their identity from BlockBnb, they can

interact with any other contracts also accepting claims issued by BlockBnb. Taking this a step further, imagine the US Postal Service deploy their own claim issuer contract, offering veriﬁable claims about an identity’s postal address. Any contract requiring a veriﬁed address can simply look for a claim issued by the US Postal Service contract before allowing interaction… and do so all on-chain.

DApp Demo

[From a technical perspective, it’s nonintuitive but amazing to realize that there isn’t a traditional backend powering our DApp. The DApp runs entirely on client- side code \(HTML and JavaScript\) and doesn’t rely on centralized servers or private databases controlled by Origin or anyone. Thanks to the Ethereum blockchain and IPFS we can create marketplace experiences that are truly decentralized and available everywhere. As a result, our DApp can be run from any machine or directly accessed from any gateway on the IPFS network. Try it on](https://ipfs.infura.io/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/) Infura[,](https://siderus.io/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/) Siderus[,](https://ipfs.jes.xxx/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby) Jes[, or](https://gateway.originprotocol.com/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/) [Origin’s own IPFS gateway](https://gateway.originprotocol.com/ipfs/QmWP28bNAJbkiKrXHAHzotKCvLyNragErycSYQQR9KiFby/) .

[You can try the DApp at](https://demo.originprotocol.com/) [demo.originprotocol.com](https://demo.originprotocol.com/)

[Usage instructions](https://medium.com/originprotocol/origin-demo-dapp-is-now-live-on-testnet-835ae201c58) [https://medium.com/originprotocol/origin-demo-dapp-is-now- live-](https://medium.com/originprotocol/origin-demo-dapp-is-now-live-on-testnet-835ae201c58) [on-testnet-835ae201c58](https://medium.com/originprotocol/origin-demo-dapp-is-now-live-on-testnet-835ae201c58)

First, the foundation of our platform is a series of smart contracts running on the Ethereum Blockchain. These smart contracts enable functionality that replaces the need for trusted intermediaries to manage marketplaces, and allows us to instead create marketplaces that are governed by a set of open, fair and transparent rules.

[Second, our Origin.js library gives developers a way to interact with our smart contracts without having to worry about the challenges of developing software on the blockchain. Origin.js is under active and rapid development, but there are already plenty of examples of how to use the library in](https://docs.originprotocol.com/) [our documentation](https://docs.originprotocol.com/) that you can try for yourself.

Finally, our demo DApp is the ﬁrst application that is built using Origin.js. This DApp is meant to illustrate the types of marketplaces that are possible and to give both developers and end users a chance to experiment with this new type of distributed marketplace.

We’ve crammed a lot of new features into this latest release, including support for user identity \(supporting the ERC 725 identity standard\), transaction steps, escrow, ratings, and

reviews.

#### Identity

Origin \(the company, not the open-source platform\) is running an ERC 725 attestation service as part of our open-source bridge server. This means we can

verify pieces of your identity like your email address, phone number, and social media accounts, and then sign attestations to be published on the blockchain. It’s a bit like using a notary service where you can choose which third parties you trust to verify your identity and/or the identities of others. If you add your name, proﬁle picture, or bio we will publish that information to IPFS and store the resulting content hash on the Ethereum blockchain. We believe it would be irresponsible for us to encourage people to publish their phone numbers to an immutable database, so for more sensitive information like your phone number, email address, or social media usernames, we’ll only store an attestation to the existence of those accounts on the blockchain instead of the data itself.

Ultimately, the best privacy conscious solution will probably involve zero- knowledge proofs and Merkle trees.

#### Transition progress & escrow

Another big improvement to our platform is tracking transaction progress with state transitions. Our smart contracts keep funds escrowed until the buyer conﬁrms receipt of the goods or services. Our DApp has been updated to show both the buyer and seller where they are in the purchase ﬂow and the next steps that need to be taken .

#### Ratings and reviews

This feedback is now included as a core part of Origin.js. On Origin, both buyers and sellers are required to leave reviews as part of the transaction ﬂow. Buyers can be conﬁdent that reviews came from veriﬁed purchasers, and sellers can trust that they’re dealing with reputable buyers. We support 5-star ratings as well as an optional text-based reviews.

Reviews are solicited for each completed transaction and can be aggregated for a speciﬁc user or product.

Resources

[live](https://erc725.originprotocol.com/) [ERC 725 identity playground](https://erc725.originprotocol.com/) [that’s currently running on the Rinkeby test net.](https://github.com/OriginProtocol/identity-playground) [GitHub](https://github.com/OriginProtocol/identity-playground) [repo](https://github.com/OriginProtocol/identity-playground)

[Origin Demo Dapp](http://www.github.com/originprotocol/demo-dapp) \(github\)

[You can try the DApp at](https://demo.originprotocol.com/) demo.originprotocol.com.

[https://erc725.originprotocol.com/\#/](https://erc725.originprotocol.com/%23/)

Learn more about Origin:

[Buy and sell on our DApp:](http://dapp.originprotocol.com/) dapp.originprotocol.com [Create your own marketplace:](https://www.originprotocol.com/creator) [originprotocol.com/creator](https://www.originprotocol.com/creator)

[Connect with our team on Discord:](http://originprotocol.com/discord) originprotocol.com/discord [Learn more on our website:](http://originprotocol.com/) [originprotocol.com](http://originprotocol.com/)

Veriﬁable Claims

**Example credential creation fl'ow**

Jane

I

User Agent I Credential  Repository1 1 I s.-s r I

1 a igateto we.b s1te

I

&gt;'

2 Rcq uc i;t C .-cd cn tia\[

I

3

I - , Verify

id!entiity

::;::=J

I

1 4 G nerat credwiti.al

. 5 Issue credcll!iial

;:=J

,..:; 6 Di.s\]Jl uy c red c.n tlul

7 Save creden tial

I

, 8 Store crodCll!tial

9 List of credentials

I

10 Show cred ntia.l Ji t 1

I                                             

X

Ja\_ne

Figu re 4 Verifiable Credential Creation Flow Diagram

'.M,e,rc h n1 lil."."c;J U ill!!!!ii Lh J t

wet, &ice u er be .1

l a,1 2L \) ' '" '' ., f a.gt:'

J,om:::

8 Il,:-diree 1 ro ·- b

' al£ '

ICrl!dcnlin\] Rcpa  toey I I User Agent I IOcdenti 1Con umcr I

## Veriﬁable Credentials Data Model

[https://www.w3.org/blog/news/archives/7927](https://www.w3.org/blog/news/archives/7927) [https://www.w3.org/2017/vc/WG/](https://www.w3.org/2017/vc/WG/) [- veriﬁable claims working group](https://github.com/search?q=verifiable-claims%2Borg%3Aw3c&type=Repositories) [https://github.com/search?q=veriﬁable- claims+org%3Aw3c&type=Repositories](https://github.com/search?q=verifiable-claims%2Borg%3Aw3c&type=Repositories) [- githhub repo](https://github.com/search?q=verifiable-claims%2Borg%3Aw3c&type=Repositories) [https://w3c.github.io/vc-use-cases/](https://w3c.github.io/vc-use-cases/) [- use cases](https://cointelegraph.com/news/will-blockchain-stop-personal-data-leaks) [https://cointelegraph.com/news/will-blockchain-stop-personal-data-leaks](https://cointelegraph.com/news/will-blockchain-stop-personal-data-leaks) [- practical use cases](https://www.w3.org/TR/2019/PR-vc-data-model-20190905/) [https://www.w3.org/TR/2019/PR-vc-data-model-20190905/](https://www.w3.org/TR/2019/PR-vc-data-model-20190905/) - W3C Veriﬁable Credentials Data Model 1.0 - September 2019

DIDs Spec

## Resources

[DID-spec](https://w3c.github.io/did-core/) [https://w3c.github.io/did-core/](https://w3c.github.io/did-core/)

[https://w3c-ccg.github.io/did-spec/](https://w3c-ccg.github.io/did-spec/) [- DiD spec by w3c](https://medium.com/uport/the-basics-of-decentralized-identity-d1ff01f15df1) [https://medium.com/uport/the-basics-of-decentralized-identity-d1ff01f15df1](https://medium.com/uport/the-basics-of-decentralized-identity-d1ff01f15df1)

DID Methods

[https://w3c-ccg.github.io/did-method-registry/](https://w3c-ccg.github.io/did-method-registry/)

1. The Registration Process Software implementers may ﬁnd that the existing Decentralized Identiﬁer Methods listed in this repository are not suitable for their use case and may need to add a new method to this registry. Adding a Decentralized Identiﬁer Method to this list is designed to be a lightweight, community-driven process. In order to add a new method to this registry, an implementer MUST: 1. Implement at least an experimental version of the new DID Method.
2. [Create a speciﬁcation describing the new DID Method that is publicly available and intended to be conformant with the DID speciﬁcation at](https://w3c-ccg.github.io/did-spec/) [https://w3c-ccg.github.io/did- spec/](https://w3c-ccg.github.io/did-spec/)[. Ensure that the title in the speciﬁcation contains a version number. 3.](https://w3c-ccg.github.io/did-spec/) Request that the speciﬁcation be added to this registry by submitting a Github Pull Request that adds the new method to the list of existing methods, with URL. Speciﬁcations that do not meet these criteria will not be accepted. Old listings which fall out of conformance may be removed. Implementers that would like help or guidance during this process are urged to join the W3C Credentials Community Group and request assistance via the mailing list.

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

Method Name

Status DLT or Network Authors Link

did:abt: PROVISIONAL

did:btcr: did:stack: did:erc725:

PROVISIONAL PROVISIONAL

ABT Network Bitcoin Bitcoin

ArcBlock ABT DID Method Christopher Allen, Ryan Grant, Kim Ham

Jude Nelson

Blockstack DID Method

PROVISIONAL

Ethereum Markus Sabadello, Fabian Vogelstelle

did:example: PROVISIONAL

DID Specification

W3C Credentials Community

did:ipid: did:life: did:sov:

PROVISIONAL PROVISIONAL

PROVISIONAL

IPFS TranSendX

IPID DID method

RChain lifeID Foundation

lifeID DID Method

Sovrin Mike Lodder Ethereum uPort

Sovrin DID Method

did:uport: PROVISIONAL

did:v1: did:dom: did:ont: did:vvo: did:icon: did:iwt:

PROVISIONAL PROVISIONAL PROVISIONAL PROVISIONAL

PROVISIONAL PROVISIONAL

Veres One Ethereum Ontology

Digital Bazaar Veres One DID Method Dominode

Ontology Foundation

Ontology DID Metho

Vivvo Vivvo Application Studios

Vivvo DID Meth

ICON ICON Foundation ICON DID Method InfoWallet Raonsecure InfoWallet DID Method

2. The Registry This table summarizes the DID method speciﬁcations currently in development. The links will be updated as subsequent Implementer’s Drafts are produced.

18

19

20

21

22

23

24

25

26

27

28

did:ockam: PROVISIONAL did:ala: PROVISIONAL

Ockam Ockam Ockam DID Method

did:op:

PROVISIONAL

Alastria Alastria National Blockchain Ecosystem Ocean Protocol Ocean Protocol Ocean Protocol DID

did:jlinc: PROVISIONAL

JLINC Protocol Victor Grey

JLINC Protocol DID

did:ion: did:jolo: did:ethr: did:bryk: did:peer:

PROVISIONAL PROVISIONAL PROVISIONAL PROVISIONAL PROVISIONAL

Bitcoin

Various DIF contributors ION DID Method

Ethereum Jolocom Jolocom DID Method

Ethereum uPort

ETHR DID Method

bryk Marcos Allende, Sandra Murcia, Flavia Munh peer Daniel Hardman peer DID Method

did:selfkey: PROVISIONAL

Ethereum SelfKey

SelfKey DID Method

did:meta:

PROVISIONAL

Metadium Metadium Foundation

Metadium DID Met

## did:erc725 method

[https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/topics-and-advance-](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/topics-and-advance-readings/DID-Method-erc725.md) [readings/DID-Method-erc725.md](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/topics-and-advance-readings/DID-Method-erc725.md)

[did:erc725 method 27th February 2018 Markus Sabadello](mailto:markus@danubetech.com) markus@danubetech.com[, Fabian Vogelsteller](mailto:fabian@ethereum.org) fabian@ethereum.org[, Peter Kolarov](mailto:pkolarov@finid.me) [pkolarov@ﬁnid.me](mailto:pkolarov@finid.me)

Decentralized Identiﬁers \(DIDs, see \[1\]\) are designed to be compatible with any distributed ledger or network \(called the target system\). In the Ethereum community, a pattern known as ERC725 \(see \[2\]\) utilizes smart contracts for standard key management functions. We propose a new DID method that allows ERC725 identities to be treated as valid DIDs. One advantage of this DID method over others appears to be the ability to use the full ﬂexibility of Ethereum smart contracts for key management purposes.

DID Method Name The namestring that shall identify this DID method is: erc725 A DID that uses this method MUST begin with the following preﬁx: did:erc725. Per the DID speciﬁcation, this string MUST be in lowercase. The remainder of the DID, after the preﬁx, is speciﬁed below.

Method Speciﬁc Identiﬁer The method speciﬁc identiﬁer is composed of an optional Ethereum network identiﬁer with a : separator, followed by a Hex-encoded Ethereum ERC725 smart contract address \(without a 0x preﬁx\). erc725-did = "did:erc725:" erc725- speciﬁc-idstring erc725-speciﬁc-idstring = \[ erc725-network ":" \] erc725-address erc725- network = "mainnet" / "ropsten" / "rinkeby" / "kovan" erc725-address = 40\*HEXDIG The

smart contract address is case-insensitive, but it is recommended to use mixed-case checksum address encoding \(see \[3\]\). This speciﬁcation currently only supports Ethereum "mainnet", "ropsten", "rinkeby", and "kovan", but can be extended in the future to support arbitrary Ethereum instances \(including private ones\).

Example Example erc725 DIDs: • did:erc725:2F2B37C890824242Cb9B0FE5614fA2221B79901E • did:erc725:mainnet:2F2B37C890824242Cb9B0FE5614fA2221B79901E • did:erc725:ropsten:2F2B37C890824242Cb9B0FE5614fA2221B79901E

DID Document

[Example { "@context": "](https://w3id.org/did/v1)[https://w3id.org/did/v](https://w3id.org/did/v1)1", "id": "did:erc725:ropsten:2F2B37C890824242Cb9B0FE5614fA2221B79901E", "publicKey": \[{ "id": "did:erc725:ropsten:2F2B37C890824242Cb9B0FE5614fA2221B79901E\#key-1", "type":

\["Secp256k1SignatureVeriﬁcationKey2018", "ERC725ManagementKey"\], "publicKeyHex": "1a0cb8f32c94921649383b14523cb6df04858cfbd4f77711371321cd8ebd87d72efe02b69 ca4b02b35a848404101ad17efbf962441733135cb7d833313c3d37b" }, { "id": "did:erc725:ropsten:2F2B37C890824242Cb9B0FE5614fA2221B79901E\#key-2", "type":

\["Secp256k1SignatureVeriﬁcationKey2018", "ERC725ActionKey"\], "publicKeyHex": "00e17b0f13af42bd7c992ef991ebd75f8345b5edb8e937eb0c9c3dea80af23448419faa1d7 562054e31bf56ab1af485944b3a327085c4502e38d723129fd5cf666" }\], "authentication": { "type": "Secp256k1SignatureAuthentication2018", "publicKey": "did:erc725:ropsten:2F2B37C890824242Cb9B0FE5614fA2221B79901E\#key-2" }, "service": \[\] }

[JSON-LD Context Deﬁnition The erc725 method deﬁnes additional JSON-LD terms for the supported ERC725 key types MANAGEMENT, ACTION, CLAIM, and ENCRYPTION. The deﬁnition of the erc725 JSON-LD context is: { "@context": { "ERC725ManagementKey": "](https://github.com/ethereum/EIPs/issues/725#ERC725ManagementKey)[https://github.com/ethereum/EIPs/issues/725\#ERC725ManagementKe](https://github.com/ethereum/EIPs/issues/725#ERC725ManagementKey)y[", "ERC725ActionKey": "](https://github.com/ethereum/EIPs/issues/725#ERC725ActionKey)[https://github.com/ethereum/EIPs/issues/725\#ERC725ActionKe](https://github.com/ethereum/EIPs/issues/725#ERC725ActionKey)y[", "ERC725ClaimKey": "](https://github.com/ethereum/EIPs/issues/725#ERC725ClaimKey)[https://github.com/ethereum/EIPs/issues/725\#ERC725ClaimKe](https://github.com/ethereum/EIPs/issues/725#ERC725ClaimKey)y[", "ERC725EncryptionKey": "](https://github.com/ethereum/EIPs/issues/725#ERC725EncryptionKey)[https://github.com/ethereum/EIPs/issues/725\#ERC725EncryptionKe](https://github.com/ethereum/EIPs/issues/725#ERC725EncryptionKey)y" } }

CRUD Operation Deﬁnitions

Create \(Register\) In order to create a erc725 DID, a smart contract compliant with the ERC725 standard must be deployed on Ethereum. The holder of the private key that created the smart contract is the entity identiﬁed by the DID. The Ethereum network identiﬁer together with the smart contract address becomes the DID as per the syntax rules above.

Read \(Resolve\) To construct a valid DID document from an erc725 DID, the following steps are performed: 1. Determine the Ethereum network identiﬁer \("mainnet", "ropsten", "rinkeby", or "kovan"\). If the DID contains no network identiﬁer, then the default is "mainnet". 2. Invoke the getKeysByType function for each of the supported key types, i.e. MANAGEMENT, ACTION, CLAIM, and ENCRYPTION. 3. For each returned key address, look up the secp256k1 public key associated with the key address. 4. For each MANAGEMENT public key:1. Add a publicKey of type Secp256k1SignatureVeriﬁcationKey2018 \(see \[4\]\) and ERC725ManagementKey to the DID Document.

1. For each ACTION public key:1. Add a publicKey element of type Secp256k1SignatureVeriﬁcationKey2018 and ERC725ActionKey to the DID Document.
2. Add an authentication element of type Secp256k1SignatureAuthentication2018, referencing the publicKey.
3. For each CLAIM public key:1. Add a publicKey element of type Secp256k1SignatureVeriﬁcationKey2018 and ERC725ClaimKey to the DID Document.
4. For each ENCRYPTION public key:1. Add a publicKey element of type Secp256k1SignatureVeriﬁcationKey2018 and ERC725EncryptionKey to the DID Document.
5. Add an encryption element of type Secp256k1Encryption2018 to the DID Document, referencing the publicKey.

Note: Service endpoints and other elements of a DID Document may be supported in future versions of this speciﬁcation.

Update The DID Document may be updated by invoking the relevant smart contract functions as deﬁned by the ERC725 standard: • function addKey\(address \_key, uint256

\_type\) public returns \(bool success\); • function removeKey\(address \_key\) public returns \(bool success\); Note that these methods are written in the Solidity language. Ethereum smart contracts are actually executed as binary code running in the Ethereum Virtual Machine \(EVM\).

Delete \(Revoke\) Revoking the DID can be supported by executing a selfdestruct\(\) operation that is part of the smart contract. This will remove the smart contract's storage and code from the Ethereum state, effectively marking the DID as revoked.

Security Considerations TODO Privacy Considerations TODO

Performance Considerations In Ethereum, looking up a raw public key from a native 20-byte address is a complex and resource-intensive process. The DID community may want to consider allowing hashed public keys in the DID documents instead of \(or in addition to\) the raw public keys. It seems this would make certain DID methods such as erc725 much simpler to implement, while at the same time not really limiting the spirit and potential use cases of DIDs.

[References \[1\]](https://w3c-ccg.github.io/did-spec/) [https://w3c-ccg.github.io/did-spec/](https://w3c-ccg.github.io/did-spec/) \[2\]

[https://github.com/ethereum/EIPs/issues/725](https://github.com/ethereum/EIPs/issues/725) \[3\]

[https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md](https://w3c-dvcg.github.io/lds-koblitz2016/) [\[4\]](https://w3c-dvcg.github.io/lds-koblitz2016/) [https://w3c-](https://w3c-dvcg.github.io/lds-koblitz2016/) [dvcg.github.io/lds-koblitz2016/](https://w3c-dvcg.github.io/lds-koblitz2016/)

