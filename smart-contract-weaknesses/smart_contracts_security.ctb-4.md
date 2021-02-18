# External Contract Referencing

External Contract Referencing  
One of the benefits of the Ethereum “world computer” is the ability to reuse code and interact with contracts already deployed on the network. As a result, a large number of contracts reference external contracts, usually via external message calls. These external message calls can mask malicious actors' intentions in some nonobvious ways, which we’ll now examine.

The Vulnerability  
In Solidity, any address can be cast to a contract, regardless of whether the code at the address represents the contract type being cast. This can cause problems, especially when the author of the contract is trying to hide malicious code. Let’s illustrate this with an example.

Consider a piece of code like Rot13Encryption.sol, which rudimentarily implements the ROT13 cipher.

Example 8. Rot13Encryption.sol  
// encryption contract  
contract Rot13Encryption {

 event Result\(string convertedString\);

 // rot13-encrypt a string  
 function rot13Encrypt \(string text\) public {  
 uint256 length = bytes\(text\).length;  
 for \(var i = 0; i &lt; length; i++\) {  
 byte char = bytes\(text\)\[i\];  
 // inline assembly to modify the string  
 assembly {  
 // get the first byte  
 char := byte\(0,char\)  
 // if the character is in \[n,z\], i.e. wrapping  
 if and\(gt\(char,0x6D\), lt\(char,0x7B\)\)  
 // subtract from the ASCII number 'a',  
 // the difference between character &lt;char&gt; and 'z'  
 { char:= sub\(0x60, sub\(0x7A,char\)\) }  
 if iszero\(eq\(char, 0x20\)\) // ignore spaces  
 // add 13 to char  
 {mstore8\(add\(add\(text,0x20\), mul\(i,1\)\), add\(char,13\)\)}  
 }  
 }  
 emit Result\(text\);  
 }

 // rot13-decrypt a string  
 function rot13Decrypt \(string text\) public {  
 uint256 length = bytes\(text\).length;  
 for \(var i = 0; i &lt; length; i++\) {  
 byte char = bytes\(text\)\[i\];  
 assembly {  
 char := byte\(0,char\)  
 if and\(gt\(char,0x60\), lt\(char,0x6E\)\)  
 { char:= add\(0x7B, sub\(char,0x61\)\) }  
 if iszero\(eq\(char, 0x20\)\)  
 {mstore8\(add\(add\(text,0x20\), mul\(i,1\)\), sub\(char,13\)\)}  
 }  
 }  
 emit Result\(text\);  
 }  
}  
This code simply takes a string \(letters a–z, without validation\) and encrypts it by shifting each character 13 places to the right \(wrapping around z\); i.e., a shifts to n and x shifts to k. The assembly in the preceding contract does not need to be understood to appreciate the issue being discussed, so readers unfamiliar with assembly can safely ignore it.

Now consider the following contract, which uses this code for its encryption:

import "Rot13Encryption.sol";

// encrypt your top-secret info  
contract EncryptionContract {  
 // library for encryption  
 Rot13Encryption encryptionLibrary;

 // constructor - initialize the library  
 constructor\(Rot13Encryption \_encryptionLibrary\) {  
 encryptionLibrary = \_encryptionLibrary;  
 }

 function encryptPrivateData\(string privateInfo\) {  
 // potentially do some operations here  
 encryptionLibrary.rot13Encrypt\(privateInfo\);  
 }  
 }  
The issue with this contract is that the encryptionLibrary address is not public or constant. Thus, the deployer of the contract could give an address in the constructor that points to this contract:

// encryption contract  
contract Rot26Encryption {

 event Result\(string convertedString\);

 // rot13-encrypt a string  
 function rot13Encrypt \(string text\) public {  
 uint256 length = bytes\(text\).length;  
 for \(var i = 0; i &lt; length; i++\) {  
 byte char = bytes\(text\)\[i\];  
 // inline assembly to modify the string  
 assembly {  
 // get the first byte  
 char := byte\(0,char\)  
 // if the character is in \[n,z\], i.e. wrapping  
 if and\(gt\(char,0x6D\), lt\(char,0x7B\)\)  
 // subtract from the ASCII number 'a',  
 // the difference between character &lt;char&gt; and 'z'  
 { char:= sub\(0x60, sub\(0x7A,char\)\) }  
 // ignore spaces  
 if iszero\(eq\(char, 0x20\)\)  
 // add 26 to char!  
 {mstore8\(add\(add\(text,0x20\), mul\(i,1\)\), add\(char,26\)\)}  
 }  
 }  
 emit Result\(text\);  
 }

 // rot13-decrypt a string  
 function rot13Decrypt \(string text\) public {  
 uint256 length = bytes\(text\).length;  
 for \(var i = 0; i &lt; length; i++\) {  
 byte char = bytes\(text\)\[i\];  
 assembly {  
 char := byte\(0,char\)  
 if and\(gt\(char,0x60\), lt\(char,0x6E\)\)  
 { char:= add\(0x7B, sub\(char,0x61\)\) }  
 if iszero\(eq\(char, 0x20\)\)  
 {mstore8\(add\(add\(text,0x20\), mul\(i,1\)\), sub\(char,26\)\)}  
 }  
 }  
 emit Result\(text\);  
 }  
}  
This contract implements the ROT26 cipher, which shifts each character by 26 places \(i.e., does nothing\). Again, there is no need to understand the assembly in this contract. More simply, the attacker could have linked the following contract to the same effect:

contract Print{  
 event Print\(string text\);

 function rot13Encrypt\(string text\) public {  
 emit Print\(text\);  
 }  
 }  
If the address of either of these contracts were given in the constructor, the encryptPrivateData function would simply produce an event that prints the unencrypted private data.

Although in this example a library-like contract was set in the constructor, it is often the case that a privileged user \(such as an owner\) can change library contract addresses. If a linked contract doesn’t contain the function being called, the fallback function will execute. For example, with the line encryptionLibrary.rot13​Encrypt\(\), if the contract specified by encryptionLibrary was:

 contract Blank {  
 event Print\(string text\);  
 function \(\) {  
 emit Print\("Here"\);  
 // put malicious code here and it will run  
 }  
 }  
then an event with the text Here would be emitted. Thus, if users can alter contract libraries, they can in principle get other users to unknowingly run arbitrary code.

Warning  
The contracts represented here are for demonstrative purposes only and do not represent proper encryption. They should not be used for encryption.

Preventative Techniques  
As demonstrated previously, safe contracts can \(in some cases\) be deployed in such a way that they behave maliciously. An auditor could publicly verify a contract and have its owner deploy it in a malicious way, resulting in a publicly audited contract that has vulnerabilities or malicious intent.

There are a number of techniques that prevent these scenarios.

One technique is to use the new keyword to create contracts. In the preceding example, the constructor could be written as:

constructor\(\) {  
 encryptionLibrary = new Rot13Encryption\(\);  
}  
This way an instance of the referenced contract is created at deployment time, and the deployer cannot replace the Rot13Encryption contract without changing it.

Another solution is to hardcode external contract addresses.

In general, code that calls external contracts should always be audited carefully. As a developer, when defining external contracts, it can be a good idea to make the contract addresses public \(which is not the case in the honey-pot example in the following section\) to allow users to easily examine code referenced by the contract. Conversely, if a contract has a private variable contract address it can be a sign of someone behaving maliciously \(as shown in the real-world example\). If a user can change a contract address that is used to call external functions, it can be important \(in a decentralized system context\) to implement a time-lock and/or voting mechanism to allow users to see what code is being changed, or to give participants a chance to opt in/out with the new contract address.

Real-World Example: Reentrancy Honey Pot  
A number of recent honey pots have been released on the mainnet. These contracts try to outsmart Ethereum hackers who try to exploit the contracts, but who in turn end up losing ether to the contract they expect to exploit. One example employs this attack by replacing an expected contract with a malicious one in the constructor. The code can be found here:

pragma solidity ^0.4.19;

contract Private\_Bank  
{  
 mapping \(address =&gt; uint\) public balances;  
 uint public MinDeposit = 1 ether;  
 Log TransferLog;

 function Private\_Bank\(address \_log\)  
 {  
 TransferLog = Log\(\_log\);  
 }

 function Deposit\(\)  
 public  
 payable  
 {  
 if\(msg.value &gt;= MinDeposit\)  
 {  
 balances\[msg.sender\]+=msg.value;  
 TransferLog.AddMessage\(msg.sender,msg.value,"Deposit"\);  
 }  
 }

 function CashOut\(uint \_am\)  
 {  
 if\(\_am&lt;=balances\[msg.sender\]\)  
 {  
 if\(msg.sender.call.value\(\_am\)\(\)\)  
 {  
 balances\[msg.sender\]-=\_am;  
 TransferLog.AddMessage\(msg.sender,\_am,"CashOut"\);  
 }  
 }  
 }

 function\(\) public payable{}

}

contract Log  
{  
 struct Message  
 {  
 address Sender;  
 string Data;  
 uint Val;  
 uint Time;  
 }

 Message\[\] public History;  
 Message LastMsg;

 function AddMessage\(address \_adr,uint \_val,string \_data\)  
 public  
 {  
 LastMsg.Sender = \_adr;  
 LastMsg.Time = now;  
 LastMsg.Val = \_val;  
 LastMsg.Data = \_data;  
 History.push\(LastMsg\);  
 }  
}  
This post by one reddit user explains how they lost 1 ether to this contract by trying to exploit the reentrancy bug they expected to be present in the contract.

