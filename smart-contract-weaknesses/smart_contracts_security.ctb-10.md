# Short Address/Parameter Attack

## Short Address/Parameter Attack

This attack is not performed on Solidity contracts themselves, but on third-party applications that may interact with them. This section is added for completeness and to give the reader an awareness of how parameters can be manipulated in contracts.

For further reading, see “The ERC20 Short Address Attack Explained”, “ICO Smart Contract Vulnerability: Short Address Attack”, or this Reddit post.

### The Vulnerability

When passing parameters to a smart contract, the parameters are encoded according to the ABI specification. It is possible to send encoded parameters that are shorter than the expected parameter length \(for example, sending an address that is only 38 hex chars \(19 bytes\) instead of the standard 40 hex chars \(20 bytes\)\). In such a scenario, the EVM will add zeros to the end of the encoded parameters to make up the expected length.

This becomes an issue when third-party applications do not validate inputs. The clearest example is an exchange that doesn’t verify the address of an ERC20 token when a user requests a withdrawal. This example is covered in more detail in Peter Vessenes’s post, “The ERC20 Short Address Attack Explained”.

Consider the standard ERC20 transfer function interface, noting the order of the parameters:

function transfer\(address to, uint tokens\) public returns \(bool success\);  
Now consider an exchange holding a large amount of a token \(let’s say REP\) and a user who wishes to withdraw their share of 100 tokens. The user would submit their address, 0xdeaddeaddeaddeaddeaddeaddeaddeaddeaddead, and the number of tokens, 100. The exchange would encode these parameters in the order specified by the transfer function; that is, address then tokens. The encoded result would be:

a9059cbb000000000000000000000000deaddeaddea \  
ddeaddeaddeaddeaddeaddeaddead0000000000000  
000000000000000000000000000000000056bc75e2d63100000  
The first 4 bytes \(a9059cbb\) are the transfer function signature/selector, the next 32 bytes are the address, and the final 32 bytes represent the uint256 number of tokens. Notice that the hex 56bc75e2d63100000 at the end corresponds to 100 tokens \(with 18 decimal places, as specified by the REP token contract\).

Let us now look at what would happen if one were to send an address that was missing 1 byte \(2 hex digits\). Specifically, let’s say an attacker sends 0xdeaddeaddeaddeaddeaddeaddeaddeaddeadde as an address \(missing the last two digits\) and the same 100 tokens to withdraw. If the exchange does not validate this input, it will get encoded as:

a9059cbb000000000000000000000000deaddeaddea \  
ddeaddeaddeaddeaddeaddeadde00000000000000  
00000000000000000000000000000000056bc75e2d6310000000  
The difference is subtle. Note that 00 has been added to the end of the encoding, to make up for the short address that was sent. When this gets sent to the smart contract, the address parameters will be read as 0xdeaddeaddeaddeaddeaddeaddeaddeaddeadde00 and the value will be read as 56bc75e2d6310000000 \(notice the two extra 0s\). This value is now 25600 tokens \(the value has been multiplied by 256\). In this example, if the exchange held this many tokens, the user would withdraw 25600 tokens \(while the exchange thinks the user is only withdrawing 100\) to the modified address. Obviously the attacker won’t possess the modified address in this example, but if the attacker were to generate any address that ended in 0s \(which can be easily brute-forced\) and used this generated address, they could steal tokens from the unsuspecting exchange.

Preventative Techniques  
All input parameters in external applications should be validated before sending them to the blockchain. It should also be noted that parameter ordering plays an important role here. As padding only occurs at the end, careful ordering of parameters in the smart contract can mitigate some forms of this attack.

### Resources

* [https://vessenes.com/the-erc20-short-address-attack-explained/](https://vessenes.com/the-erc20-short-address-attack-explained/)

