---
description: >-
  Areas of traditional cybersecurity that can be improved by implementing
  blockchain solutions instead.
---

# Blockchain for Cybersecurity Infrastructure

## DPKI

### Deploying PKI-Based Identity with Blockchain

Organizations have many applications to manage, and these are hosted by different systems and servers. Organizations have deployed several ways to authenticate users, based on methods such as multi-factor authentication, one for each system/application, single sign-on \(SSO\), and the directory server; however, authenticating users on the internet is a comparatively difficult mechanism. It is also extremely important to achieve trust over the internet before exchanging information because the internet has been kept open for trusted and untrusted parties. In order to established trust over a public network, there is the need for an independent trusted party. A public key infrastructure \(PKI\) is an open framework built to resolve trust factors between internet-connected users.

### PKI

The internet allows anyone to connect to anyone else and, unlike the real world, geographical/physical barriers don't exist. This makes it difficult to identify a person over the internet and establish trust for further communication. The PKI solves this problem by appending a trusted third party \(TTP\) between Bob and Alice.

• Network devices: access to routers with 802.1X authentication  
• Applications that need signed certs to run in the OS  
• IPSec tunnels: to authenticate other endpoints over the internet  
• Radius servers: LDAP query is protected by PKI

### Challenges of the existing PKI model

The challenges of the existing PKI model are as

**the need for additional security**: According to a report from the Ponemon Institute's 2016 research, 62% of businesses have deployed cloud-based applications using PKI, with an increase of 50% in 2015. If the central certificate repository gets compromised, it will lead to a massive data breach and account theft. Organizations tend to use an additional layer of security such as hardware security modules \(**HSMs**\) to secure their PKIs. HSMs are deployed to protect PKIs for the most critical root and for issuing CA private keys. Organizations are opting for **multi-factor authentication** for administrators and HSM use.

**central authority**: In the current state of the internet, a central authority \(**root authority**\) is responsible for managing DNS requests and responses \(root authority\), X.509 certificates, and much more. Therefore, all internet-connected devices and systems have to trust the third party to manage public keys and identifiers. Let's take an example of a domain name; even though it has been purchased by its owner, it practically belongs to third parties, such as the Internet Corporation for Assigned Names and Numbers \(ICANN\), domain registrars, and certificate authorities.

Furthermore, these trusted third parties are very much capable of intercepting and compromising the integrity and security of users worldwide.There have been several cases where these trusted third parties have shared their customer's information to security agencies and other bodies. They can either do this for financial gain or to prepare customer behavior analytics.

### How can Blockcahin help

A **decentralized public key infrastructure \(DPKI\)** is an innovative concept that creates authentication systems over public systems without depending on a single third party that can compromise the integrity and security of the system.

A principal can be given direct control over global readable identifiers, such as a website domain, by registering the identifier in the blockchain.

### Deployment method

The PKI is implemented as a function in a smart contract in an Ethereum blockchain. Each entity can have multiple attributes to authenticate ownership. This entity can be a public key or an Ethereum address. A smart contract is used to program the events and functions of various operations in the PKI, such as create, derive, remove, destroy, and many more.

• **The registration of an entity**: Users or systems are added to the PKI system by calling a registration event from the smart contract. The entity can be as simple as an Ethereum address, public key, attribute ID, data, and data hashes.  
• **The signing of attributes**: An entity can be characterized using a registration event. Each attribute of the entity can be signed by the PKI system through a smart contract, and a transaction will be issued.  
• **The retrieval of attributes**: The attributes of the entities can be located by applying a filter to the blockchain using the respective IDs of events that have been configured on the smart contract.  
• **Revoke signature**: This is one of the most critical functions required by any PKI solution: to revoke the digital signature on attributes or entities. Revocation becomes extremely important when a user loses his/her key or it is compromised. Smart contracts can be configured to invoke the revocation event and revoke the signature on a specific entity.

### Requirements

In the DPKI deployment, the registrar still has a role in the infrastructure, but it is restricted as follows to ensure that the identities of entities are represented in the network:

• It is required to ensure that software is always under the control of principals and corresponding keys.  
• Private keys have to be generated in a decentralized way to ensure that they remain under the control of the principal.  
• The generation of a key pair on behalf of a principal has to be strictly prohibited.  
• There has to be no single entity that can change other entities without consent from the principal.  
• Once a namespace is created within a blockchain through an Ethereum smart contract, it can't be destroyed.  
• The registration and renewal of identifiers has to be transparent.  
• By default, software that manages identifiers must ensure that all activities such as creating, updating, renewing, or deleting identifiers is forwarded through a decentralized mechanism.

### Testing

Info about RPs and DCPs: [IKP: Turning a PKI Around with Blockchains](https://eprint.iacr.org/2016/1018.pdf#targetText=A%20reaction%20policy%20should%20be,policies%20in%20addition%20to%20certificates) \(https://eprint.iacr.org/2016/1018.pdf\)

CAs can issue **Reaction Policies \(RPs\)**, which take effect if an unauthorized certificate for a domain is issued. In the process of testing, we need to register **Domain Certificate Policies \(DCPs\)** and create RPs. The testing can be done with the following steps on our local  system:

`pragma solidity ^0.5.0`

`contract DPKI {`

 `// We first need to add a detector and register it. The following script is required to add a detector by defining its detector ID:`

 `function addDetector(address detectorAddress) public returns (uint detectorID) {  
 detectorID = detectors.length++;  
 Detector storage detector = detectors[detectorID];  
 detector.authority = detectorAddress;  
 emit DetectorAdded(detectorID, detectorAddress);  
 }`

 `// We will now register a CA used by the domain owner to issue certificates. It is required to define CA ID, CA owner address, and  name, shown as follows:`

 `function registerCA(address caAddress, string caName) public returns (uint caID) {  
 caID = cas.length++;  
 CertificateAuthority storage ca = cas[caID];  
 ca.caOwner = caAddress;  
 ca.caName = caName;  
 emit CAAdded( calD, caAddress, caName);  
 }`

 `//cRegister DCP with the CAs`

 `function registerDCP(string identifier, string data, string certHash, uint certExpiry, address CA) public returns (uint dcpID) {  
 dcpID = dcps.length++;  
 DomainCertificatePolicy storage dcp = dcps (dcpID);  
 dcp.identifier = identifier;  
 dcp.owner = msg.sender;  
 dcp.data = data;  
 dcp.CA = CA;  
 dcp.certHash = certHash;  
 dcp.certExpiry = certExpiry;  
 emit DCPAdded(dcpID, msg. sender, identif , data, certHash, CertExpiry, CA);  
 }`

 `// Create an associated RP under the smart contract, shown as follows:`

 `function signRP(uint dcpID, uint expiry) public returns (uint signatureID) {  
 if (dcps[dcpID].CA == msg.sender) {  
 signatureID = rps.length**;  
 ReactionPolicy storage rp = rps[signatureID];  
 rp.CA = dcps[dcpID].CA;  
 rp.signer = msg.sender;  
 rp.attributeID = dcpID;  
 rp.expiry = expiry;  
 emit RPSigned(signatureID, msg.sender, rp.CA, dcpID, expiry);  
 }  
 }`

 `// The detector can now blacklist the CA when a rogue CA misbehaves frequently, shown as follows:`

 `function blacklistCA(uint caIndex, uint detectorIndex) public {  
 // detectors can blacklist CAS if they breach a threshold.  
 if (detectors[detectorlndex].authority == msg.sender {  
 if (cas.length > 1) {  
 cas[caIndex] = cas[cas.length—1];  
 delete(cas[cas.length—1]);  
 }  
 cas.length——;  
 }  
 emit CABlacklisted(caIndex, detectorlndex);  
 }  
}`

this way, we have successfully deployed the PKI with an Ethereum blockchain. With this infrastructure, we have described the full process, from registering a CA to claiming reaction payouts. We have successfully developed a model describing reaction payouts, and developed a method to enforce accountability on CAs that are misbehaving.

### Resources

**Blockchain related**  
The Sidetree Protocol [https://medium.com/decentralized-identity/the-sidetree-scalable-dpki-for-decentralized-identity-1a9105dfbb58](https://medium.com/decentralized-identity/the-sidetree-scalable-dpki-for-decentralized-identity-1a9105dfbb58)  
short paper outlining DPKI implementation and privacy features [https://isrdc.iitb.ac.in/blockchain/workshops/2017-iitb/papers/paper-11%20-%20Decentralized%20PKI%20in%20blockchain%20and%20Smart%20contract.pdf](https://isrdc.iitb.ac.in/blockchain/workshops/2017-iitb/papers/paper-11%20-%20Decentralized%20PKI%20in%20blockchain%20and%20Smart%20contract.pdf)  
DPKI Overview by Vitalik et al [https://github.com/WebOfTrustInfo/rwot1-sf/blob/master/draft-documents/Decentralized-Public-Key-Infrastructure-CURRENT.md](https://github.com/WebOfTrustInfo/rwot1-sf/blob/master/draft-documents/Decentralized-Public-Key-Infrastructure-CURRENT.md)  
IKP: Turning a PKI Around with Blockchains at [https://eprint.iacr.org/2016/1018.pdf](https://eprint.iacr.org/2016/1018.pdf)  
**Web 2.0 PKI**  
PKI Technical Standards at [http://www.oasis-pki.org/resources/techstandards/.](http://www.oasis-pki.org/resources/techstandards/)  
PKI - Public Key Infrastructure at [https://www.ssh.com/pki/](https://www.ssh.com/pki/).  
**Ethereum Alternatives**  
KSI \(Alternative to ethereum PKI\) by Guardtime - [https://guardtime.com/technology/blockchain-developers](https://guardtime.com/technology/blockchain-developers)

## Key Management

### Blockchain-based Dynamic Key Management

A recent report from the US Department of Transport \(DoT\) indicates that nearly 82% of traffic accidents can be prevented by introducing intelligent transportation system \(ITS\) into the existing traffic systems \[1\]. ITS is proposed as the only candidate to solve the current problems within transportation systems, such as road safety, navigation, and congestion control. As a submodule of the ITS, Vehicular Communication System \(VCS\) supports the exchange of messages between vehicles and also with infrastructural facilities \[2\].

 In addition to the message exchange among multiple vehicles, VCS supports message communications between vehicles and infrastructure as well.

To guarantee the trustfulness and legality of safety messages, the messages are supposed to be encrypted with a pre-agreed secret key. Thus, the problem of providing VCS application layer security can be mapped into the problem of how to reliably distribute or update secret keys among all the communicating participants, especially how to timely deliver the secret key to another security domain to finish the node handover progress.

high mobility, a massive number of devices, and a wide range of vehicle activities pose extra challenges to VCS-centralized management and Access Point \(AP\) deployment. For this reason, **distributed VCS management structures are considered as a possible method to achieve higher network management efficiency, mild network manager burden, and lower infrastructure building cost.**

The current solution to achieve trusted safety message exchange among the VCS area is to encrypt and authenticate the message \[3\] before broadcasting the message to VCS. In a paper \[5\], it was proposed to use blockchain in line with a decentralized system to manage personal data over IoT devices. The access control of personal data is monitored by blockchain.

Thus, the first aim of key management research is to **reduce the overall broadcast messages**, also known as the communication overhead, whereas the second aim is to **speed up the key management processing time**.

The authentication server is located at the top level of the system architecture and is responsible for the management, issuance, and initialization of cryptographic materials, such as secret keys and certificates.

In VCS, SM-A knows the vehicle is about to join the coverage area of SM-B according to the driving direction, speed, position, and all the cryptographic materials. Thus, SM-A informs SM-B about the message handover action in order to let SM-B update keys to the vehicle. To sum up, the handover schemes in the mobile network need a round trip between three entities \(MN, FA, and HA\) to finish, while only a one-way communication is needed in VCS.

Figure 6.3 VCS network structure; \(a\) traditional structure; \(b\) blockchain-based structure.

the function of Blockchain enables nodes to share information without the need for centralized supervision of this ledger by a central manager.  
As presented in Figure 6.3\(b\), the central manager \(PKI\) is placed in an isolated environment to dedicatedly generate cryptographic materials for all the nodes. Cryptographic materials, such as vehicle identities, pseudonyms, and pseudonym certificates, are supposed to be kept in a secured facility for privacy and security purposes \[11\].

Blockchain for Distributed Systems Security . Wiley. Kindle Edition.

### Key management risks

**Key management risk:** While the consensus protocol immutably seals a blockchain ledger and no corruption of past transactions is possible, it’s still susceptible to private keys theft and the takeover of assets associated with public addresses. Digital assets could become irretrievable in the case of accidental loss or private key theft, especially given the lack of a single controller or a potential escalation point within the framework.

### Keyless signatures

Guardtime, “KSI Technology Stack”. \[Online\]. 2018. Available: [https://guardtime.com/technology/ksi-technology.](https://guardtime.com/technology/ksi-technology.)

A. Buldas, A. Kroonmaa, and R. Laanoja, “Keyless signatures infrastructure: How to build global distributed hash-trees,” in Nordic Conference on Secure IT Systems. Springer, 2013, pp. 313–320.

Ericsson, “Ericsson and Guardtime create secure cloud and big data.” \[Online\]. 2014. Available: [https://www.ericsson.com/news/1853499.](https://www.ericsson.com/news/1853499.)

## 2FA

### Two-Factor Authentication with Blockchain

### How does 2FA work?

2FA can be deployed in two modes: a cloud-based solution and an on-premises solution.

**Cloud-based solution**: This is heavily used by e-commerce, online banking, and other online service-related web applications.

**On-premises solution**: Organizations hesitate to allow cloud-based security solutions and tend to prefer on-premises solutions where an employee accessing web applications applications puts in a combination of a username and password. Now, this information goes to the internal VPN integrator, which handles credentials and exchanges a key between organizations and third-party 2FA providers. The third-party 2FA provider will generate the OTP and share this with the employee over SMS or through mobile applications. This model helps achieve privacy for an organization, as it doesn't have to share the credentials with a third-party 2FA provider.

### Challenges

Although 2FA increases the level of security with the second layer of authentication, it still encounters the drawback of having the centralized database store a list of secret user information. The central database can be tampered with or corrupted by targeted threats, and this can lead to massive data breaches.

### How can blockchain transform 2FA?

**Can we also achieve MFA with Ethereum, and if so, how?**  
• Ethereum can be used to develop a multi-factor authentication platform by programming a smart contract. This smart contract has to be programmed to connect with several integrations, such as biometric and mobile applications.  
**How can we integrate SMS-based 2FA with an Ethereum smart contract?**  
• In order to achieve an SMS-based 2FA platform, Ethereum's smart contract has to be programmed to integrate with the SMS gateway to send a One Time Password \(OTP\) for a second level of authentication. This OTP protects against Man-in-the-Middle \(MITM\) attacks.

In this system, user devices will be authenticated by a third-party 2FA provider through the blockchain network. Each party in the blockchain network will hold the endpoint information securely and will activate the 2FA system to generate the second-level password.

This can either be deployed in the public domain, or even a private network with a third-party API call:

### Solution Architecture

A user accesses the web portal and enters the first level of credentials. A web application will communicate to the Ethereum-based repository to generate the OTP and shares this with the user. Finally, the user enters the same OTP and gains access to the web application.

### Basic Functionality Story

• User logs in to example.com with username-password.  
• example.com asks the user to send a transaction to their 2FA contract, and starts waiting for an Authenticated event. \(Note: events before this login are ignored\). A reasonable timeout should be set to account for fees and network congestion — after which this login is rejected if no Authenticated event is heard for this user's Ethereum address.  
• User sends a transaction \(does not send ETH, only calls the contract function and pays the gas fee\) to the example.com 2FA contract.  
• example.com sees an Authenticated event on the contract, and provided it is within the timeout and was created by the user's Ethereum address, allows the login.  
• User is authenticated.

### Critical Issues with Blockchain 2FA

Reasons that 2FA through Ethereum contracts is an awful idea:

• It does not add any real security. If an attacker already has your password, if they also know your public Ethereum address, they could simply listen to the blockchain for 2FA activity on your address and attempt their login around the same time \(in the hope for a second login attempt in succession\).  
• Every 2FA would cost the user at least the minimum network fee, and may be subject to high fees or slow confirmations due to network congestion.  
• The usability of this system would likely be worse than a typical 2FA system such as one-time-passwords provided by 1Password, Google Authenticator, or Authy.

### Implementation

To turn up the entire project, we will have to deploy the sub-component of this project. The source has been taken from GitHub, which can be found at the following link:

[https://github.com/hoxxep/Ethereum-2FA](https://github.com/hoxxep/Ethereum-2FA)

Note this concept has security and usability flaws, and is therefore merely a demonstration of a basic smart contract and testing with truffle

Dependencies:

Node v7.6+  
Ethereum TestRPC  
Truffle

`# Install Truffle and the Ethereum TestRPC dev tools.  
npm install -g truffle ethereumjs-testrpc`

`# Install local dependencies  
npm install`

`# Start the ethereum testrpc or Ganache, they both listen on port 8545  
testrpc // or ganache-cli`

`# In a new terminal window, run the truffle tests  
truffle test`

## DNS

### Challenges with current DNS

Firewalls leave port 53 open and never look inside each query. Let's look at one of the most widely used DNS-based attacks:

• **DNS cache poisoning**: An attacker can take advantage of cached DNS records and can then perform spoofing by injecting a forged DNS entry into the DNS server. As a result, all users will now be using that forged DNS entry until the time the DNS cache expires.

• **Compromising a DNS server**: A DNS server is the heart of the entire DNS infrastructure. An attacker can use several attack vectors to compromise a DNS server and can provide the IP address of a malicious web server against each legitimate DNS query.

• **Man-in-the-middle \(MITM\) attack**: In this type of attack, a threat actor keeps listening to conversations between clients and a DNS server. After gathering information and sequence parameters, it starts spoofing the client by pretending to be the actual DNS server and provides the IP addresses of malicious websites.

• **Zone transfer**

### Blockchain-based DNS solution: DNSChain

**DNSChain** is one of the most active projects to transform the DNS framework and protect it from spoofing challenges.

DNSChain is a blockchain-based DNS software suite that replaces X.509 public key infrastructure \(PKI\) and delivers MITM proofs of authentication. It allows internet users to set a public DNSChain server for DNS queries and access that server with domains ending in `.bit`.

#### X.509 PKI replacement

X.509 is a standard framework that defines the format of PKI to identify users and entities over the internet. It helps internet users to know whether the connection to a specific website is secure or not. DNSChain has the capability to provide a scalable and decentralized replacement that doesn't depend on third parties.

#### MITM-proof DNS infrastructure

This uses a public key pinning technique to get rid of the MITM attack problem. Public key pinning specifies two **pin-sha256** values; that is, it pins two public keys \(one is the pin of any public key in the current certificate chain and the other is the pin of any public key not in the current certificate chain\):  
• It works in parallel with existing DNS servers  
• Websites and individuals store their public key in the blockchain  
• The keys are shared over the DNSChain software framework

[DNSChain](Cybersecurity--DNS--DNSChain.html)

DNSChain is an Ethereum-based DNS platform to connect client and name server without any involvement of a third party in between, hosted on GitHub at [https://github.com/okTurtles/dnschain](https://github.com/okTurtles/dnschain)

### ENS

### NameCoin

### DNSChain

#### What is DNSChain?

DNSChain makes it possible to be certain that you're communicating with who you want to communicate with, and connecting to the sites that you want to connect to, without anyone secretly listening in on your conversations in between.

#### DNSChain replaces X.509 PKI with the blockchain

[X.509](https://en.wikipedia.org/wiki/X.509) makes and breaks today's Internet security. It's what makes your browser think "The connection to this website is secure" when [it's not.](https://okturtles.org/#not-secure) It's what we have to get rid of, and DNSChain provides a scalable, distributed, and decentralized replacement that doesn't depend on untrustworthy third-parties.

DNSChain provides security properties that are opposite of X.509. In X.509, the more Certificate Authorities there are, the less secure all SSL/TLS connections are. On the other hand, the more DNSChain servers one queries, the more likely it is one will have accurate information.

### MITM-proof all Internet connections

"One Pin To Rule Them All"  
Connections between DNSChain and the clients it serves are MITM-proofed through the well-known technique of public-key pinning. The key difference with DNSChain is that a single pin is all that's required to MITM-proof all other Internet connections:

• DNSChain works side-by-side with full blockchain nodes that run on the same server.  
• Websites and individuals store their public key in a blockchain \(DNSChain supports several\), and from then on that key becomes MITM-proof.  
• The latest and most correct key is sent to clients over the secure channel between DNSChain and its clients. They are then able to establish further MITM-proof connections with the owner\(s\) of those key\(s\).

It bears emphasizing that the DNSChain server itself could be malicious, and therefore users should only query the server \(or servers\) that they have good reason to trust. If they don't have access to a trustworthy DNSChain server, they should query several independently run servers and verify that the responses match.

📺 Watch: [Securing online communications with the blockchain](https://www.youtube.com/watch?v=Qy1x3Ud8LCI)

### Simple and secure GPG key distribution

The blockchain is a secure, decentralized database. The information stored in it can be read at any location around the world, and DNSChain provides easy access to it via a [blockchain agnostic API](Cybersecurity--DNS--DNSChain.html#API).

Easily share your GPG key!  


Any DNSChain server can be used to retrieve the same information. Storing information in the blockchain is a bit more difficult, but with time, this too will become simple.

### RESTful API to Any Blockchain

okTurtles is working with Onename to develop [a spec](https://github.com/okTurtles/openname-specifications/blob/resolvers/resolvers.md) for RESTful access to blockchains. Here's what it looks like:

`https://api.example.com/v1/namecoin/key/id%2Fbob  
https://api.example.com/v1/bitcoin/addr/MywyjpyBbFTsHkevcoYnSaifShG2Et8R3S  
https://api.example.com/v1/namecoin/key/id%2Fclinton/transfer?to_addr=ea3df...  
http://api.example.com/v1/resolver/fingerprint`

The URL follows this pattern:

`/{version}/{chain|resolver}/{resource}/{property}/{operation}{resp_format}?{args}`

Secret: You can even access traditional DNS via this API!

DNSChain lets you access traditional DNS records over HTTP! Note that DNS, unlike blockchains, is not secure, so even though you might be accessing it over a MITM-proof channel to DNSChain, there's little preventing DNSChain's access to the rest of the old DNS system from being MITM'd.

Still, if for some reason you need it, it's there:

`GET` [https://api.example.com/v1/icann/key/okturtles.com](https://api.example.com/v1/icann/key/okturtles.com)  
`=> {"version":"0.0.1","header":{"datastore":"icann"},"data":{"edns_options":[],"answer":[{"name":"okturtles.com","type":1,"class":1,"ttl":299,"address":"192.184.93.146"}],"authority":[],"additional":[]}}`

📄 See complete details: [Openname Resolver Specification](https://github.com/okTurtles/openname-specifications/blob/resolvers/resolvers.md)

### Free SSL certificates become possible

Certificates issued by Certificate Authorities \(CAs\) can be undermined by thousands of entities \(other CAs, their employees, governments, and hackers\).

It does not matter whether you pay $300 per year for the fancy green bar in a browser or $0/year, your website's visitors can still be attacked.

Together, DNSChain and the blockchains it works with replace CAs by providing a means for distributing public keys in a way that is secure from MITM attacks. Because of this, free self-signed certificates can be used. Unlike CAs, users are given actual reason to trust DNSChain: they choose the server they trust \(their own, or a friend's\).

### Prevents DDoS attacks

  
****  
Unlike traditional DNS servers, DNSChain encourages widespread deployment of the server \(ideally, "one for every group of friends", similar to how people rely on personal routers today\). This distributed, flat topology eliminates the need for open resolvers by making it practical to limit clients to a small, trusted set. Additionally, whereas traditional DNS resolvers must query other DNS servers to answer queries, blockchain-based DNS resolvers have no such requirement because all of the data necessary to answer queries is stored locally on the server.

Another DoS attack relates to the centralized manner in which today's SSL certificates are checked for revocation:

### Certificate revocation that actually works

The wonderful thing about blockchains is that they have no conception of revocation because they do not need one. Instead, the most recent value for any particular key in a blockchain is the most accurate and up-to-date value.

TODO: [Explain OCSP](https://news.ycombinator.com/item?id=7556909) and DoS plays a role in it.

### DNS-based censorship circumvention

The developers of Unblock.us.org and DNSChain are teaming up to bring the anti-censorship features of Unblock.us into DNSChain. Each project benefits from the other: DNSChain ensures MITM-free communication and Unblock.us ensures that the communication passes through firewalls.

The Unblock.us feature is optional and is up to the server administrator to enable and configure to their needs. It uses MITM to defeat censorship at its own game.

Unblock.us works by hijacking the DNS lookups for the domains on a list defined by the server administrator. The server then accepts all HTTP and HTTPS traffic addressed to those domains and forwards it intelligently. Even though it can't decrypt SSL traffic, it can still forward it. It's as fast as a VPN \(unlike Tor\) and ONLY tunnels the traffic to those domains, meaning that it doesn't affect other online activites \(unlike VPNs and Tor\) and isn't costly in server bandwidth. Finally, there's no software to install, only DNS settings to change. It has been confirmed to work in Turkey, UK, Kuwait, UAE and many additional Middle Eastern countries.

For now, Deep Packet Inspection techniques used in Pakistan and China can still beat Unblock.us, but the next version will address that issue using a technique called Host Tunneling. Short of cutting entire countries off the internet, DNSChain/Unblock.us will be able to get through.

### Other features: testing suite, rate-limiting, and caching

DNSChain's tests cover all the core functionality and excellent code coverage.  
Protects against DNS amplification attacks with built-in rate-limiting features for both DNS and HTTP requests that are completely configurable.  
Supports caching of both DNS and HTTP responses via Redis. Also configurable to your liking.  
The .dns metaTLD

metaTLDs are useful when you need to talk to the DNS server you're connected to, but do not have access to DNS information. This makes them very useful whenever browser extensions, or other software environments where access to DNS is not simple.

Note that metaTLDs cannot be "registered" because they resolve to the DNS server that the user is connected to, if that server supports metaTLDs.

📄 I[ntroducing the dotDNS metaTLD](https://blog.okturtles.org/2014/02/introducing-the-dotdns-metatld/)

### DNSChain + Namecoin + PowerDNS

[https://github.com/okTurtles/dnschain.](https://github.com/okTurtles/dnschain.)

### How do I run my own DNSChain Server?

[https://github.com/okTurtles/dnschain/blob/master/docs/How-do-I-run-my-own.md](https://github.com/okTurtles/dnschain/blob/master/docs/How-do-I-run-my-own.md)

 Requirements  
Getting Started  
Configuration  
Guide: Setting up DNSChain + Namecoin + PowerDNS  
Get yourself a Linux server \(they come as cheap as $2/month\), and then make sure you have the following software installed:

Requirements  
nodejs \(or iojs\), and npm - We recommend using a package manager to install them  
coffee-script \(version 1.7.1+\) - install via npm install -g coffee-script  
A supported blockchain daemon like namecoind  
Getting Started  
Install DNSChain using: npm install -g dnschain \(you may need to put sudo in front of that\).  
Run namecoind in the background. You can use systemd and create a namecoin.service file for it based off of dnschain.service.  
If an update is released, update your copy using: npm update -g dnschain  
Verify it runs  
Test DNSChain by simply running dnschain from the command line \(developers see here\). Have a look at the configuration section below, and when you're ready, run it in the background as a daemon. As a convenience, DNSChain comes with a systemd unit file that you can use to run it.

By default, it will start listening for DNS requests on port 53 if you run it as root, and 5333 otherwise. You can test .bit resolution \(for properly registered Namecoin domains\) using dig:

$ dig @localhost -p 5333 example.bit  
📄 Guide to Setting Up DNSChain + Namecoin + PowerDNS on Debian Wheezy

Verify the SSL/TLS key and certificate fingerprint  
By default, DNSChain will automatically use the openssl command to generate a random public/private keypair for you.

By default, it will place the private key in ~/.dnschain/key.pem and ~/.dnschain/cert.pem, and it will chmod 600 the private key \(making it unreadable by other user accounts on the machine\). You should verify the permissions are correct yourself.

If you want, you can generate the key and certificate yourself using a command similar to this:

openssl req -new -newkey rsa:4096 -days 730 -nodes -sha256 -x509 \  
 -subj "/C=US/ST=Florida/L=Miami/O=Company/CN=[www.example.com"](http://www.example.com/) \  
 -keyout key.pem \  
 -out cert.pem  
The autogen'd certificate uses the hostnameof your machine for the CN \("Common Name"\).

When you run DNSChain, it will output the certificate's fingerprint.

You should see DNSChain say something like this:

2015-02-22T06:19:39.935Z - info: \[TLSServer\] Your certificate fingerprint is: E2:3D:01:5D:3C:27:26:67:12:12:05:FA:11:4A:CB:D6:0D:3E:21:1E:4C:D3:43:C0:FC:79:DB:24:91:31:EE:18  
That string of hexadecimals is your server's "One Pin To Rule Them All" that clients will need to verify \(once only\) if they want to establish a man-in-the-middle-proof connection to DNSChain over TLS \(for example, to query its RESTful API\).

Using nginx for SSL/TLS instead of DNSChain  
DNSChain runs several servers, including its own HTTP and HTTPS servers.

You can, if you want, use nginx as the HTTPS server, and then proxy traffic to DNSChain's HTTP server. Just make sure to configure the various ports correctly \(you'll need to tell DNSChain to not listen on port 443\), and see to it that nginx is using the same certificate and key that DNSChain is using.

Configuration  
❗️ Have a look at config.coffee to see all configuration options and defaults!  
DNSChain uses nconf for all of its configuration purposes. This means that you can configure it using files, command line arguments, and environment variables.

DNSChain looks for its configuration file in one of the following locations:

dnschain.conf locations \(in order of preference\):  
$HOME/.dnschain/dnschain.conf \(Recommended location\)  
$HOME/.dnschain.conf  
/etc/dnschain/dnschain.conf  
Blockchain configuration  
DNSChain will look for the configuration files of supported blockchains. If a blockchain has a configuration file, and DNSChain cannot find it, then it will disable support for that blockchain.

You can manually tell DNSChain the location of a blockchain's config file by specifying its path in DNSChain's config file. To do this, create a section with the name of the blockchain \(all lowercase, should match the file name of one of the supported blockchains\), and set the config variable, like so:

\[namecoin\]  
config = /weird/path/to/namecoin.conf  
DNSChain uses these files to retrieve the information it needs to speak to that blockchain \(for example, the JSON-RPC username and password\).

Example DNSChain configuration  
The format of the configuration file is similar to INI, and is parsed by the NodeJS properties module \(in tandem with nconf\).

Here's an example of a possible dnschain.conf:

\[log\]  
level=info

\[dns\]  
port = 5333  
\# no quotes around IP  
oldDNS.address = 8.8.8.8

\# disable traditional DNS resolution \(default is NATIVE\_DNS\)  
oldDNSMethod = NO\_OLD\_DNS

\[http\]  
port=8088  
tlsPort=4443  
The values you place here will override the values of the defaults variable in config.coffee.

Notice that the first level of that object specifies a section in the configuration file \(like log, dns\), and after that deeper levels are accessed by using a ., as in: oldDNS.address = 8.8.8.8

Possible configuation options  
We hope to add a super-simple web admin interface to DNSChain \(most of this work is completed in the admin branch\). Until that's ready, look at the defaults variable in config.coffee and read the comments.

There are settings for http and dns servers, Redis caching to improve performance, and anti-DDoS ratelimiting settings as well \(for both http and dns requests\). The Unblock setting is still in development.

Guide: Setting up DNSChain + Namecoin + PowerDNS  
As a guide we have an example server setup using Debian 7 and PowerDNS, along with a Namecoin node. This setup will resolve .bit domain names and should serve as an example which can be used with other blockchains.

📄 Guide to Setting Up DNSChain + Namecoin + PowerDNS on Debian Wheezy

📄 Guide to Setting Up DNSChain + Namecoin + PowerDNS on Ubuntu

📄 Guide to Setting Up DNSChain + Namecoin + PowerDNS on FreeBSD

## DDoS

### Challenges with current DDoS solutions

• Single point of failure

### How can blockchain help

In order to protect networks from DDoS attacks, organizations can be made distributed between multiple server nodes that provide high resilience and remove the single point of failure. There are two main advantages to using blockchain, as follows:

• Blockchain technology can be used to deploy a decentralized ledger to store blacklisted IPs  
• eliminates single point of failure

### Gladius

[https://github.com/gladiusio/gladius-contracts](https://github.com/gladiusio/gladius-contracts)

[https://github.com/gladiusio/gladius-control-daemon](https://github.com/gladiusio/gladius-control-daemon)  
[https://github.com/gladiusio/gladius-control-daemon/tree/master/restdocs/settings\#post-start](https://github.com/gladiusio/gladius-control-daemon/tree/master/restdocs/settings#post-start)

### Resources

[https://hackernoon.com/gladius-gla-ddos-security-and-page-load-time-solution-using-blockchain-technology-53178e7c69ab](https://hackernoon.com/gladius-gla-ddos-security-and-page-load-time-solution-using-blockchain-technology-53178e7c69ab) - laymans explaination

[https://medium.com/@gladiusio](https://medium.com/@gladiusio)  
[https://github.com/gladiusio](https://github.com/gladiusio)

A Blockchain-Based Architecture for Collaborative DDoS Mitigation with Smart Contracts at  
[https://www.springer.com/cda/content/document/cda\_downloaddocument/9783319607733-c2.pdf?SGWID=0-0-45-1609389-p180909480](https://www.springer.com/cda/content/document/cda_downloaddocument/9783319607733-c2.pdf?SGWID=0-0-45-1609389-p180909480)

Collaborative DDoS Mitigation Based on Blockchains at  
[https://files.ifi.uzh.ch/CSG/staff/Rafati/Jonathan%20Burger-BA.pdf](https://files.ifi.uzh.ch/CSG/staff/Rafati/Jonathan%20Burger-BA.pdf)

