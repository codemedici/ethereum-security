# Prevent Replay Attacks When Using Signatures

When using signed messages for authorizing operations on behalf of a signer, an attacker could reuse the signed message to trigger the same operation multiple times. Use replay attack protections to guard against this.

### Description

A common pattern for gasless interactions is to ask users to sign a message instead of a transaction to authorize an action. A third-party relayer, such as a Defender Relayer, can then send the message in a transaction, without requiring the user to pay for the gas fees. The recipient contract then verifies the message signature, and executes the action requested by the signer.

This pattern is used extensively in meta-transaction scenarios, whether it is in general-purpose like [EIP-2771](https://eips.ethereum.org/EIPS/eip-2771), or action-specific like gasless [ERC-20](https://eips.ethereum.org/EIPS/eip-20) transfers in [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612) or [EIP-3009](https://eips.ethereum.org/EIPS/eip-3009).

As an example, the following code snippet executes a token transfer on behalf of a signer, by requiring a signed message that includes the recipient and amount. It uses [OpenZeppelin Contracts ECDSA library](https://docs.openzeppelin.com/contracts/3.x/api/cryptography#ECDSA) for recovering the signer.

```text
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/cryptography/ECDSA.sol";

// Insecure code, do not use!!
contract MyERC20 is ERC20 {
  using ECDSA for bytes32;

  function transferWithSignature(
    address recipient, 
    uint256 amount,
    bytes memory signature
  ) public returns (bool) {
    bytes32 hashed = keccak256(abi.encode(recipient, amount));
    // Recover the signer address from the signature
    address sender = hashed.recover(signature);
    
    _transfer(sender, recipient, amount);
  }
}
```

A variant of the pattern above is to include the signer as a parameter, and validate that it matches the address recovered from the signature. Although this consumes more gas, this leads to clearer errors in case of signature mismatches, instead of simply operating on a different address.

```text
// Insecure code, do not use!!
contract MyERC20 is ERC20 {
  using ECDSA for bytes32;

  function transferWithSignature(
    address recipient, 
    uint256 amount, 
    address sender, 
    bytes memory signature
  ) public returns (bool) {
    bytes32 hashed = keccak256(abi.encode(recipient, amount));
    // Validate the recovered address matches the intended sender
    address recovered = hashed.recover(signature);
    require(recovered == sender, "Signature mismatch");

    _transfer(sender, recipient, amount);
  }
}
```

However, without replay attack protection, a single signed message could be submitted multiple times to drain the signer's funds. Replay attack protection is required within the same contract, across different contracts, and also across different chains.

#### Using Nonces

The most secure pattern for replay attack protection within the same contract is to use _nonces_. A nonce is a value that can be used at most once. Once an action is processed by the recipient, the associated nonce should be marked as used. Any new signed messages that use the same nonce must be rejected.

Note that the Ethereum network itself uses nonces for replay attack protection at the transaction level. Each Ethereum transaction includes a unique increasing nonce to ensure it is only processed once.

A simple implementation for nonce management is to track used nonces per signer in the verifying contract. Every time a message is processed, the associated nonce is marked as used. For reference, [EIP-3009](https://eips.ethereum.org/EIPS/eip-3009) follows this approach.

An alternative is to maintain a single sequential nonce per signer. This has the advantage of being more gas-efficient, since each signer uses a single storage slot instead of once per message sent. However, it enforces messages to be processed in-order, which may or may not be desired depending on the use case. For reference, [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612) follows this approach.

#### Replay Across Contracts

While using nonces protects against the same message being replayed on the verifying contract, it does not prevent it from being replayed on a different contract. An attacker could gather a message signed for one contract, but then replay it on a different one with the same interface.

Following the fictitious example of the ERC-20 `transferWithSignature`, an attacker could reuse the signed message submitted to an ERC-20 to a different token contract, thus draining the sender of funds on any other ERC20 that supports this method.

An easy technique to protect against this is to include the address of the verifying contract as part of the signed message. The verifier then checks that the address in the signature matches its own, and rejects the messages otherwise.

#### Replay Across Chains

There is still one more scenario where a signed message can be replayed: across different chains. The same contract could be deployed on the same address on different networks, such as Rinkeby, Mainnet, and xDai. An attacker could grab a signed message destined to a contract on one of the networks, and replay it on the others. A protection against this type of replay attack is to include the chain identifier as part of the signature. The EVM provides a `chainid` opcode that can be used for this.

Note that, as with nonces, Ethereum itself uses this protection for transactions as defined in [EIP-155](https://eips.ethereum.org/EIPS/eip-155).

#### Relevant EIPs

The [EIP-191](https://eips.ethereum.org/EIPS/eip-191) _Signed Data Standard_ defines different types of signed messages under different _versions_. In particular, version `0x00` defines an intended validator, which standardizes how the recipient address `address(this)` should be included as part of the signature.

EIP-191 also introduces another type of replay protection. Given that signed messages use the same signing method as Ethereum transactions, it is possible that an application-specific signed message can also be a valid Ethereum transaction and vice-versa. To prevent this, EIP-191 introduces a `0x19` prefix which is deemed invalid in the context of an Ethereum transaction.

The [ERC-712](https://eips.ethereum.org/EIPS/eip-712) for _Ethereum typed structured data hashing and signing_ provides a standard for signing typed data. Instead of arbitrarily concatenating the message parameters into a single bytestring and signing it, this standard defines a way to sign structured data, which enables wallets to provide end-users with a clear view of the data being signed. EIP-712 is defined on top of EIP-191.

ERC-712 also defines how to include both the verifier address `address(this)` and the chain identifier `chainid` as part of the [`DOMAIN_SEPARATOR`](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator) component of the signature, plus an optional `salt` if further disambiguation if needed. Given this, EIP-712 is the recommended way of managing signatures. Modern EIPs like the aforementioned [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612), [EIP-2771](https://eips.ethereum.org/EIPS/eip-2771), and [EIP-3009](https://eips.ethereum.org/EIPS/eip-3009) all build on top of it.

Note that **neither EIP-191 nor EIP-712 include nonce management to prevent replay attacks** within the same contract. It is left to you as the developer to implement that manually. See [_Note on replay attacks_ on EIP-712](https://eips.ethereum.org/EIPS/eip-712#note-on-replay-attacks).

### Example Using EIP-191

The following example uses EIP-191 version `0x00` to verify the intended recipient, and includes the chainId as part of the message to prevent cross-chain replay attacks. It also uses sequential nonces to prevent replay attacks within the contract.

_Note that EIP-712 is preferred over EIP-191. See the second example for an implementation using EIP-712._

```text
contract EIP191SequentialNonceERC20 is ERC20 {
  constructor() public ERC20("MyERC20", "MYT") { }

  mapping(address => uint256) nonces;

  function transferWithSignature(
    address recipient, 
    uint256 amount, 
    address sender, 
    uint256 nonce, 
    bytes memory signature
  ) public returns (bool) {
    // Calculate EIP-191 compliant hashed data
    // Includes intended recipient address, nonce, and chain id
    bytes32 hashed = keccak256(abi.encodePacked(
      byte(0x19), byte(0), address(this), recipient, amount, nonce, chainID()
    ));

    // Recover signer and verify it
    address recovered = ECDSA.recover(hashed, signature);
    require(recovered == sender, "Signature mismatch");
    
    // Verify and increase nonce
    require(nonces[sender] == nonce, "Repeated nonce");
    nonces[sender]++;
    
    // Execute action authorized by sender
    _transfer(sender, recipient, amount);
  }

  function chainID() private pure returns (uint256) {
    uint256 chainID;
    assembly {
      chainID := chainid()
    }
    return chainID;
  }
}
```

### Example Using EIP-712

The following example uses EIP-712 instead of EIP-191. It follows a very similar approach to [EIP-3009](https://eips.ethereum.org/EIPS/eip-3009) by combining EIP-712 with non-sequential nonces. While more extensive than previous one due to requirements of EIP-712, this standard is preferred over plain EIP-191.

```text
contract EIP712RandomNonceERC20 is ERC20 {
  
  constructor() public ERC20("MyERC20", "MYT") { 
    // Domain separator defined by EIP-712
    domainSeparator = keccak256(abi.encode(
      keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
      keccak256("MyERC20"),
      keccak256(bytes("1")),
      chainID(),
      address(this)
    ));
  }

  // Identifies protocol, verifier, and chain as specified by EIP-712
  bytes32 immutable domainSeparator;

  // Required by EIP-712 to identify the struct being signed
  bytes32 private immutable TRANSFER_TYPEHASH = keccak256(
    "TransferWithSignature(address sender,address recipient,uint256 amount,uint256 nonce)"
  );

  // Non-sequential nonces
  mapping(address => mapping(bytes32 => bool)) nonces;

  function transferWithSignature(
    address recipient, 
    uint256 amount, 
    address sender, 
    bytes32 nonce, 
    bytes memory signature
  ) public returns (bool) {

    // Hash of structured data as defined according to the TYPEHASH
    bytes32 hashStruct = keccak256(abi.encode(
      TRANSFER_TYPEHASH,
      sender,
      recipient,
      amount,
      nonce
    ));

    // Signed hash as defined by EIP-712
    bytes32 hashed = keccak256(abi.encodePacked(
      "\x19\x01",
      domainSeparator,
      hashStruct
    ));

    address recovered = ECDSA.recover(hashed, signature);
    require(recovered == sender, "Signature mismatch");
    
    require(!nonces[sender][nonce], "Repeated nonce");
    nonces[sender][nonce] = true;
    
    _transfer(sender, recipient, amount);
  }

  function chainID() private pure returns (uint256) {
    uint256 chainID;
    assembly {
      chainID := chainid()
    }
    return chainID;
  }
}
```

Note that the example above precalculates the `DOMAIN_SEPARATOR`, which is used \(among other things\) to protect against cross-chain replay attacks, since it includes the `chainID`. This assumes that the `chainID` is immutable, though this is not the case in the unlikely event of [a fork](https://en.wikipedia.org/wiki/Ethereum_Classic). If you want your contract to be resistant to replay attacks across chain forks, you would need to recalculate the `DOMAIN_SEPARATOR` instead of precomputing it, at the expense of higher a gas cost per transaction.

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/prevent-replay-attacks-when-using-signatures?](https://defender.openzeppelin.com/#/advisor/docs/prevent-replay-attacks-when-using-signatures?)

