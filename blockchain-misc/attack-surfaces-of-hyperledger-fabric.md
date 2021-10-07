# Attack Surfaces of Hyperledger Fabric

Article from Extropy Research:

{% hint style="success" %}
**Source:** [**https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit)\*\*\*\*
{% endhint %}

## **Introduction**

**Fabric has a number of vulnerabilities as outlined in this document -**  


* **Compromised MSP**
* **Malicious Ordering Service**
* **Malicious Validating Nodes**
* **External Attacks**
* **Protocol Based Attacks**
* **Chaincode Vulnerabilities**
* **Implementation/Architectural**
* **HSM**

**Most of the attacks would result in the entire network being severely affected. Endorsement could stop, data leaked, network stopped, long delays in endorsement, loss of privacy, chaincode failure, and other failures.**  


**As Fabric is permissioned based if an attacker gains control of an administrator role then severe damage could be done.**  


**A detailed analysis is needed of an actual implementation to determine the extent of the vulnerability for a particular network.**

## **Summary of Attacks**

| **Type of Attack** | **Vulnerability** | **Risk** |
| :--- | :--- | :--- |
| **Compromised MSP** |  |  |
| **Sybil**   | **Fake Peers join the network** | **Via majority endorsement the network is compromised** |
| **Boycott**   | **Genuine nodes have their access revoked** | **Nodes are excluded from the endorsement process** |
| **Blacklisting**   | **CRL changed or iCA access revoked** | **Genuine CAs cannot be used - authority is compromised on the network** |
| **Malicious Ordering Service** |  |  |
| **Sabotage**   | **Transactions ignored by selected peers** | **Endorsement affected** |
| **Intentional Fork**  | **Different blocks sent to different peers** | **Forks created** |
| **Block Size** | **BatchSize adjusted to a very small or large number** | **Network is flooded with blocks or has no blocks** |
| **Batch Time** | **BatchTimeout is set to a very low or high figure** | **Network is flooded with blocks or has no blocks** |
| **Block Withholding** | **Blocks not sent to peers** | **No updates done** |
| **Transaction Reordering** | **Change sequence of transactions** | **Prevent transactions that look like double spends when they are genuine** |
| **Malicious Validating Node** |  |  |
| **Double Spend**   | **Peer ignores the version checks in the read-write-set** | **Double Spend is possible** |
| **dDoS**  | **Peer fetchs from the OS flooding the OS** | **Network slows down** |
| **Wormhole** | **Peer leaks state and other details** | **Privacy lost without detection** |
| **External Attacks** |  |  |
| **Collusion** | **Peers create an alternative blockchain** | **Network blockchain has false data** |
| **Interface Attacks** | **Poorly coded interface** | **Leaking of confidential data** |
| **SSL Stripping** | **MITM attack**  | **Confidential details obtained** |
| **Malicious Clients** | **Invalid transactions sent to the Ordering Service** | **Network slows down** |
| **Protocol Based Attacks** |  |  |
| **CFT** | **Byzantine node attack** | **No consensus** |
| **Gossip Protocol** | **Eclipse Attack** | **Non-genuine nodes get control** |
| **Eclipse Attack** | **Node surrounded by fake nodes** | **False blockchain given to new nodes** |
| **Malleability** | **Proposal modified by an attacker in the network** | **No endorsement** |
| **Chaincode Vulnerabilities** |  |  |
| **Unrestricted Chaincode Containers** | **Nmap to map the network, attacker machine deploys attacker’s smart contract** | **Deploy fake smart contract** |
| **Non-deterministic chaincode** | **Use of global variables** | **Values lost when container restarted** |
| **Halting Problem** | **Certain inputs cause the smart contract to stop to stop an infinite loop** | **Smart contract does not run** |
| **Low Level Access** | **System chaincodes corrupted** | **Endorsement and validation fails.** |
| **Lack of Input Validation** | **Poorly formatted inputs enter the system; inject JSON** | **States are invalid following an update** |
| **Implementation/Architectural** |  |  |
| **Docker CVE Bugs** | **Exploiting docker bugs** | **Whole system could be corrupted** |
| **CouchDB** |  |  |
| **Web Interface** | **Access without a password** | **Database accessed** |
| **Dictionary** | **Dictionary attacked to match hashes in blockchain** | **Data revealed** |
| **TLS** |  |  |
| **TLS not-enabled** | **Attacker can access messages** | **Network compromised** |
| **HSM** |  |  |
| **Firmware** | **Exploit buffer overflows, upload firmware** | **Dump HSM Secrets** |
| **Quantum Resistant** |  |  |
| **Quantum**  | **ECC on which PKI depends**  | **Keys exposed** |

##   ****

## **Details**

[**Attack Surfaces of Hyperledger Fabric**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.1kb17p5qhjgj)

[**Introduction**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.v0x2k7gveark)

[**Summary of Attacks**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.g2gjomfumenu)

[**Details**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.rlg27ga1ubkm)

[**A Compromised MSP**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.dw9ytlkgso48)

[**Sybil Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.u6z2oedv11uu)

[**Boycott Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.k2tmj9be6m7y)

[**Blacklisting Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.61q85ob9agsb)

[**Malicious Ordering Service**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.t8mdkj3xqsqi)

[**Sabotage Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.q5tilwnzwxu5)

[**Intentional Fork Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.3t521rgf31wx)

[**Block Size Attack - BatchSize \(too large or too small\)**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.15k9qonqv8fk)

[**Batch Time Attack - BatchTimeout \(too large or too small\)**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.4g6ubbdluhom)

[**Block Withholding Attacks**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.otm7phocizrr)

[**Transaction Reordering Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.154uzzi3lwb4)

[**Malicious Validating Nodes**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.m18c08ly4yd7)

[**Double Spend Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.lg91jil3la8m)

[**DDoS Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.987hk8gr6q71)

[**Wormhole Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.rewqnus5u5wf)

[**External Attacks**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.vc1w711wmem4)

[**Collusion**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.l8p8hhdob7xg)

[**Interface Attacks**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.xpzcgkg5okn3)

[**SSL Stripping**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bwfott9nkjz0)

[**Malicious Clients**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.t0kwrkfxyiim)

[**Protocol Based Attacks**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.ou3y8auifl1k)

[**CFT**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.4ql8hces2ucw)

[**Gossip Protocol**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.aydx8a22ggpq)

[**Eclipse attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.sxl6kjan96z)

[**Malleability Attack**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.d189qn78ki9d)

[**Chaincode Vulnerabilities**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.sw9v1rv4ml2e)

[**Unrestricted Chaincode Containers**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.6thw2usq6b5h)

[**Non-deterministic Chaincode**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bqdhkbjhriou)

[**If a global variable is used that could be a non-deterministic code implementation. Hence the value could change if the container is restarted.**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bqdhkbjhriou)

[**Halting Problem**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bqdhkbjhriou)

[**Low Level Access**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bqdhkbjhriou)

[**Lack of Input Validation**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.bqdhkbjhriou)

[**Architectural**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.3dx9m2c2qk8m)

[**Docker**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.pgx5u4g37kal)

[**CouchDB**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.pyrr8416e34h)

[**TLS**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.wxx4qqucw2w2)

[**HSM**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.5niyg84w9f4v)

[**Quantum**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.ii92efo5slp8)

[**References**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.p7qx4qbyk5vm)

[**The fork detection and rollback feature is available in Hyperledger Fabric since version 1.4**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.hc9ar5xaivtj)

[**Understanding the Read, Write Set in Hyperledger Fabric**](https://docs.google.com/document/d/1vKYcMCG1AUjNxwZC6JJSYLixoPZNKtRIeGhy1ZxcZ0g/edit?ts=5e6bb535#heading=h.57jsorxsqi5q)  
  
****

### 

### **A Compromised MSP**

**Identities are controlled via a CA in Fabric, and these digital identities are then mapped to an organizational identity by the MSP which defines permissions \(admin, peer, client, orderer\). The MSP is defined locally for all actors \(LocalMSP\), and for a channel there is a channelMSP.**  


**The MSP controls the permissions and organizational identities without any reference to anything else. Hence if the MSP is compromised, the network can be attacked. The MSP is controlled by a single administrator. Hence if an administrator were compromised the entire identity and permission structure could be compromised.**   


**Valid identities for this MSP instance are required to satisfy the following conditions:**

* **They are in the form of X.509 certificates with a verifiable certificate path to exactly one of the root of trust certificates;**
* **They are not included in any CRL;**
* **And they list one or more of the Organizational Units of the MSP configuration in the OU field of their X.509 certificate structure.**

#### **Sybil Attack**

**If an attacker takes control of an MSP for an organization then the attacker could create peers under the attacker’s control which are then use via a majority endorsement policy to control the network. This would work by normal endorsements taking place on peers but the peers would be under the control of an attacker, hence any write sets could be created which would then be validated and committed into the world state and marked as valid in the blockchain.**   


**Therefore, the entire network would have incorrect results.**  


#### **Boycott Attack**

**If the attacker controls the MSP then new nodes could be denied access to the network and even existing nodes could have their access revoked. This would disable the organizational contribution to the network.**  


#### **Blacklisting Attack**

**The MSP is manually configured by a config\_update message for the MSP instance of a channel. The MSP could remove an immediate CA from the list of trusted CA certs in a localMSP configuration or add the CA to CRL.**  


**These methods would disable the CA. Any nodes using that CA would have no authority. These methods could be used to disrupt and disable the network.**   
  
  


###  ****

### **Malicious Ordering Service**

**The Ordering Service consists of orderers or ordering service nodes. These nodes receive transactions from the client applications and they consolidate them into blocks which are saved to the orderer’s ledger on the channel and distributed to all the peers on the channel. The order of the transactions make the blocks which are sent to the peers.**

**The ordering service nodes \(OSNs\) belong to one or more organizations, like other nodes, and organizational CAs issue digital certificates to ordering nodes MSPs will define the organizational identity of the ordering nodes.**

**The ordering nodes may use Kafka in a crash fault tolerant manner or the raft consensus algorithm.** 

**For the Kafka implementation there are several implementations - about 10 - which are possible; one example of an implementation is each channel has a corresponding partition which then returns an ordered list of transactions to all the OSNs. The node signs the block. These implementations are very dependent on BatchTimeout and BatchSize which will be considered below.**

**For Raft, every channel runs on a separate instance of the Raft protocol allowing each instance to elect a different leader. This allows more decentralization where ordering nodes are controlled by different organizations. Channel creators \(and channel admins\) have the ability to pick a subset of the available orderers. Only the ordering nodes need to know the leader.**

**We shall consider how an attack vector may occur.**

#### **Sabotage Attack**

**Similar to a fork, in this attack a malicious ordering service just could just ignore transactions endorsed by certain peers. This would in effect render an organization unable to endorse transactions. This would completely neutralize the ability of an organization to endorse transactions.**  


#### **Intentional Fork Attack**

**In a healthy network, a fork is not possible. The blocks are all the same and sent to the validating peers to validate. Providing endorsement policies are met a normal network will validate the blocks and the transactions update the world state and the blockchain transactions are flagged as valid.**  


**A malicious ordering service could deliberately send different blocks to different leader peers. This could cause validating peers to have different blocks. Depending on the endorsement policies, the outcome could be to create forks in the network or to reject the blocks completely.**  


**But in the cases of different blocks being sent the network will fail to work and either not update anything correctly, or start to build forks on the ledgers on different peers.**   


**This attack comes from a malicious ordering service sending different blocks to different peer leaders.**   
  


#### **Block Size Attack - BatchSize \(too large or too small\)**

**The ordering service sets the channel configurations which are stored on the peer ledgers in a config-block. The latest config-block is kept in memory. If the BatchSize is set to a very large number of a very small number then the network will be adversely affected.This could be done by a malicious ordering service admin.**   


**If BatchSize is very large, then the consensus algorithm will wait for enough transactions to cut a block but for a large figure that will never happen and hence no blocks are ever created and the network effectively stops.**  


**If the BatchSize is very small, then the peers will be flooded with blocks and the blockchain itself for the channel will contain a very large number of blocks. That will slow down the network and lead to inefficiency.**  


**Hence a malicious OS admin could completely stop the network or slow it down to the point of very poor usability by changing the config-block and the BatchSize.**  


#### **Batch Time Attack - BatchTimeout \(too large or too small\)**

**As with attacks based on block size, a batch time attack is designed to flood the network or stop it. If BatchTimeout is set to a very high figure the consensus algorithm will wait for a long period before cutting a block and hence the network slows down to the point of almost stopping.**   


**If the batch time out is very low, the network will have a very large number of blocks meaning the ledger is flooded with blocks. This could lead to a network inefficiency.**   


**This attack is based on a malicious orderer admin setting the BatchTimeout to a value which is too small or too large causing the network to slow down or effectively stop.**  


#### **Block Withholding Attacks**

**In this attack, the ordering service could just not disseminate certain blocks to the validating peers. They could be cut by the consensus algorithm and then not sent to update the world state. This would control what state updates were done. The validating peers would just never get the blocks.**  


#### **Transaction Reordering Attack**

**In this attack, the ordering service artificially reorders transactions so that it appears one transaction happened after another. For example if T1 referred to a payment and T2 also to a payment sometime later, the ordering service could delay T1 from being included into a block until T2 was committed into the blockchain. That could make T1 look like a double-spend and be ignored when T2 was the actual double-spend.**   


**This attack is based on an ordering service controlling the order of transactions based on a criteria to favour a particular transaction.**   
  


### 

### **Malicious Validating Nodes**

#### **Double Spend Attack**

**A read-write set is prepared by the endorser. The read set contains keys and committed version numbers, and the write set contains keys and values. The version in the read-set must match the world state version for a valid transaction.**   


**A malicious peer could ignore this validation check and even when versions disagree then add the transaction to the ledger. Hence a malicious peer could allow multiple updates for the same key allowing a double-spend.** 

#### **DDoS Attack**

**A peer could flood the ordering service with a lot of fetch calls using -**  
  


| **Peer channel fetch config** |
| :--- |


**This could cause the ordering service to respond and for a lot of fetches the orderers could be overwhelmed. The network is very sensitive to bandwidth usage.**  


#### **Wormhole Attack**

**Within a private network, a malicious peer creates a virtual private network with the outside network and leaks the information of its own private network. This attack can be launched without any knowledge of honest members of the private network.**

### **External Attacks**

#### **Collusion**

**The permissioned ledgers have a small number of peers typically. If peers colluded then the blockchain could provide an alternative history. Unlike permissionless blockchains, there is no third party verification of the blockchain in a ledger. It is all confirmed by peers within the channel.**   


**Hence if a third party which was not a peer \(eg a company using the network as a service\) used the network to confirm results, the network could provide false results via collusion when several peers agree to collude.**   


#### **Interface Attacks**

**Transactions are invoked using an interface which would typically involve confidential data passed in the input. A careless dApp could leak confidential data. SSL Stripping is an example.**  


#### **SSL Stripping**

**Under this attack, an attacker is a MITM and intercepts communications from the client which is tricked into using an Http protocol and hence all the confidential details are seen by the attacker.**  


#### **Malicious Clients**

**A malicious client is able to send unendorsed transactions to the ordering service which would cut blocks. Although they would be invalid, the OSNs would send the blocks to the peers which include the blocks into the blockchain, marking the transactions as invalid.**  


**But this attack could pollute the network with a lot of invalid transactions.**  


### 

### **Protocol Based Attacks**

#### **CFT**

**Kakfa and Raft are Crash Fault Tolerant \(CFT\) and that means if a leader node fails, consensus can be found still. They are vulnerable to Byzantine nodes. Hence if there is just one malicious node the network can fail to reach consensus.**  


#### **Gossip Protocol**

**The Ordering service broadcasts the blocks to the leader peer in an Organization which then sends the blocks via a gossip protocol to the validating peers making it vulnerable to an eclipse attack.**  


#### **Eclipse attack**

**The target node is isolated and malicious nodes surround it. These nodes can provide a fake ledger to new nodes.**   


#### **Malleability Attack**

**When a sender broadcasts his transaction to other peers, the adversary can modify the content of a transaction i.e. the signature or even modify the receiver identity and then recalculate the transaction hash and further broadcast the transaction. Now, the sender waits for the endorsement of his transaction, which never happens as the**

**transaction hash was modified by the adversary. In this situation, the sender being confused, resend the transaction to**

**the receiver.**  


### 

### **Chaincode Vulnerabilities**

**Chaincode takes the form of ledger based code to implement business level functions, and system level to implement the endorser systems and validation systems. There are certain vulnerabilities with chaincode as follows.**   


#### **Unrestricted Chaincode Containers**

**Fabric’s smart contracts run in a Docker container not segmented from the rest of the network. Deploy smart contract, install nmap and map the network, use SSH to forward a port to an attacker machine, connect directly to the internal state database of a peer, and modify an insurance contract. The modified insurance contract is stored on the blockchain and propagated through the network discreetly.**  


#### **Non-deterministic Chaincode**

#### **If a global variable is used that could be a non-deterministic code implementation. Hence the value could change if the container is restarted.**

#### **Halting Problem**

**Certain inputs cause the smart contract to stop to stop an infinite loop**

#### **Low Level Access**

**System level chaincode is corrupted and hence core systems do not work.**

#### **Lack of Input Validation**

**Poorly formatted or invalid data updates the state.**  


## **Architectural** 

#### **Docker**

**Docker software containers are used to automate the deployment of applications. Containers are intended to provide all the resources required by an application in order to run and so mean an application can be more independent of the host environment. As such, poorly configured Docker environments can weaken the security of the host environment and could allow unauthorised access to hosts and containers.**  


**The security of the Docker environment can therefore be improved by validating the Docker configuration. This review will highlight areas where improvements can be made, identify issues and ensure that the advantages offered by the Docker environment are operating securely.**  


**The secure configuration of the Docker daemon is reviewed to ensure that it is appropriately locked down and that the various defence in depth mechanisms available in a containerised environment have been correctly deployed.**

**The security of the image files used as part of the Docker deployment will also be reviewed if they are available. This will be done to understand whether security good practice measures have been appropriately followed when the images were created.**

**If relevant, the configuration of Docker swarm clusters and networks will also be covered.**

**Typically, Extropy will explore the following areas as part of the Docker review:**

* **Docker patch status**
* **Docker host file permissions and audit configuration**
* **Docker daemon authentication and authorisation**
* **Use of defence in depth security mechanisms such as AppArmor and SELinux**
* **Configuration of Linux Namespaces and Capabilities on running images**
* **Docker network configuration**
* **Secrets management for Docker containers**
* **Docker swarm configuration and key management**

**Resources:**

**CVE vulnerabilities -** [**https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=docker**](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=docker)  
  
****

#### **CouchDB**

**Updates via a web interface.**  


**A dictionary attack is possible when expected formats are known, eg trade id, name, address.**  


**We will assess the back-end database, ensuring that no unnecessary user accounts are enabled, that no user account has excessive permissions, that default settings have been changed, that sensitive data is encrypted, and that any passwords stored are hashed, not stored in plain text. Authentication methods will be checked, to ensure that only users with the correct permissions have access to the system.**  


**We will also ensure that the database is not listening on any external network ports, rendering it vulnerable to brute-force password guessing.**  


**Along with these stages, we will look for a number of application-specific vulnerabilities, including the use of outdated versions of the SNMP protocol \(early versions of which do not support encryption\), default credentials, the possibility of persistent cross-site scripting \(where malicious JavaScript can be stored in the application database\), and \(in systems implementing RMON\) the presence of hidden and default communities.**  


#### **TLS**

**TLS should be enabled.**  


**We will assess multiple certificate, cipher and protocol issues with TLS and known attacks which would compromise the confidentiality of information or the availability of the servicise \(DoS attacks\).**

### **HSM**

1. **Use legitimate SDK access to upload a firmware module that would give a shell inside the HSM.** 
2. **Use the shell to run a fuzzer on the internal implementation of PKCS\#11 commands to find reliable, exploitable buffer overflows.**
3. **Exploit these buffer overflows from outside the HSM, i.e. by just calling the PKCS\#11 driver from the host machine**
4. **Write a payload that would override access control and, via another issue in the HSM, allowing upload arbitrary \(unsigned\) firmware. It's important to note that this backdoor is persistent – a subsequent update will not fix it.**
5. **Write a module that would dump all the HSM secrets, and uploaded it to the HSM**

**the private signing keys should be stored securely in a Hardware Security Module \(HSM\), and a strong process surrounding their usage should be implemented to prevent abuse. If strong processes around the use of these keys are not implemented, then the signing process may be vulnerable to insider attacks, such as those listed in the references.**

**The Scary and Terrible Code Signing Problem You Don’t Know You Have - https://www.sans.org/reading-room/whitepapers/critical/scary-terrible-code-signing-problem-you-36382**

**Securing Your Private Keys as Best Practice for Code Signing Certificates - http://www.symantec.com/content/en/us/enterprise/white\_papers/b-securing-your-private-keys-csc-wp.pdf**

### **Quantum**

**HLF, enterprise blockchains, and current global PKI that relies on the PKI X.509 standard to ensure secure communication between various network participants are utterly vulnerable to the quantum computing threat.**  


**It has been shown that quantum computers break ECC on which PKI depends and therefore exposes its implementers and users to potentially massive fines for non-compliance and security incidents with GDPR, FINRA and HIPAA laws.**

### **References**

**Ref -** [**https://par.nsf.gov/servlets/purl/10083311**](https://par.nsf.gov/servlets/purl/10083311)  
****

**Attack Surface Analysis of Permissioned Blockchain Platforms for Smart Cities**  


**Ref -** [**https://www.researchgate.net/publication/334405589\_Vulnerabilities\_on\_Hyperledger\_Fabric**](https://www.researchgate.net/publication/334405589_Vulnerabilities_on_Hyperledger_Fabric)  
****

**Vulnerabilities on Hyperledger Fabric**  


**Ref -** [**https://www.researchgate.net/publication/337276742\_Ripping\_the\_Fabric\_Attacks\_and\_Mitigations\_on\_Hyperledger\_Fabric**](https://www.researchgate.net/publication/337276742_Ripping_the_Fabric_Attacks_and_Mitigations_on_Hyperledger_Fabric)  
****

**Ripping the Fabric: Attacks and Mitigations on Hyperledger Fabric**  


**Ref -** [**https://arxiv.org/pdf/1903.08856.pdf**](https://arxiv.org/pdf/1903.08856.pdf)  
****

**Impact of Network Delays on Fabric**  


**Ref -** [**https://arxiv.org/pdf/1904.03487.pdf**](https://arxiv.org/pdf/1904.03487.pdf)  
****

**Exploring the Attack Surface of Blockchain: A Systematic Overview**  


**Ref -** [**http://www.bchainledger.com/2019/12/fork-detection-and-rollback-in.html**](http://www.bchainledger.com/2019/12/fork-detection-and-rollback-in.html)  
****

### **The fork detection and rollback feature is available in Hyperledger Fabric since version 1.4**

**Ref -** [**https://docs.google.com/document/d/19JihmW-8blTzN99lAubOfseLUZqdrB6sBR0HsRgCAnY/edit**](https://docs.google.com/document/d/19JihmW-8blTzN99lAubOfseLUZqdrB6sBR0HsRgCAnY/edit)  
****

**A Kafka-based Ordering Service for Fabric**  


**Ref -** [**https://jbba.scholasticahq.com/article/9902.pdf**](https://jbba.scholasticahq.com/article/9902.pdf)  
****

**Transitioning to a Hyperledger Fabric Hybrid Quantum Resistant Classical Public Key Infrastructure**  


**Ref -**  [**https://medium.com/@spsingh559/understanding-the-read-write-set-in-hyperledger-fabric-92109d2a87ed**](https://medium.com/@spsingh559/understanding-the-read-write-set-in-hyperledger-fabric-92109d2a87ed)  
****

## **Understanding the Read, Write Set in Hyperledger Fabric**

**Ref -** [**https://hyperledger-fabric.readthedocs.io/en/release-2.0/readwrite.html**](https://hyperledger-fabric.readthedocs.io/en/release-2.0/readwrite.html)  
****

**Read-Write set semantics**  


**Ref -** [**https://arxiv.org/pdf/1901.09873.pdf**](https://arxiv.org/pdf/1901.09873.pdf)

**Physical Access Control Management System Based on Permissioned Blockchain**  


**Ref -** [**https://arxiv.org/pdf/1811.01410.pdf**](https://arxiv.org/pdf/1811.01410.pdf)

**Design of Anonymous Endorsement System in Hyperledger Fabric**  


[**https://www.researchgate.net/publication/331241229\_Endorsement\_in\_Hyperledger\_Fabric\_via\_Service\_Discovery**](https://www.researchgate.net/publication/331241229_Endorsement_in_Hyperledger_Fabric_via_Service_Discovery)

**Service Discovery for Hyperledger Fabric**  


[**https://hyperledger-fabric.readthedocs.io/en/release-2.0/peers/peers.html**](https://hyperledger-fabric.readthedocs.io/en/release-2.0/peers/peers.html)

**Peers**  


[**https://www.epfl.ch/labs/dedis/wp-content/uploads/2020/01/report-2018\_2-marie-jeanne-security-assessment.pdf**](https://www.epfl.ch/labs/dedis/wp-content/uploads/2020/01/report-2018_2-marie-jeanne-security-assessment.pdf)

**Security Assessment of Authentication and Authorization Mechanisms in Ethereum, Quorum, Hyperledger Fabric and Corda**  
  
  
  


