---
description: >-
  Ensure that all administrative private keys are stored in an offline hardware
  wallet or in a secure software vault that is only used for signing.
---

# Secure All Administrative Keys

## Description

All private keys that are used by individuals to sign administrative transactions to smart contracts should be stored in hardware wallets that are offline except when used to sign transactions, with the mnemonic for account recovery stored on paper also offline. For further protection, never share the private keys across devices and never use the same internet-connected device to sign transactions with multiple administrative keys. In other words, try to maintain a "one internet-connect device one administrative key" policy at all times.

Some applications also require the use of administrative keys stored on servers and used as "hot wallets" to sign transactions submitted to smart contracts. This may be required for cron jobs that call administrative functions that need to be activated at specific intervals, or at specific times, or in response to identified events. Server side signing keys may also be used for server-to-server integrations or meta-transactions.

Any administrative keys stored on servers to sign transactions should be stored within a secure key vault. Once created, private keys should never be exported from the key vault and should only be invoked \(via secure connection to the vault\) for transaction signing. Examples of secure key vaults you might use are the [AWS Key Management Service](https://aws.amazon.com/kms/) or [HashiCorp Vault](https://www.vaultproject.io/). Also note that Defender Relayers and Autotasks provide secure key vaults automatically.

As further protection, rotate all administrative keys every six months, including the keys used for signing transactions and any keys required to obtain access rights to signing \(for example keys required to invoke signing when using a secure key vault\). Rotation further reduces the chance of the administrative keys being leaked, and practicing key rotation is also useful in case a key leak is detected and you are required to quickly rotate the keys to stop potential attackers. If you are using Defender Relayers, rotate the API keys every 6 months.

Utilizing the above best practices along with requiring multiple signatures for every administrator-signed transaction \(using a multi-sig wallet such as [Gnosis Safe](https://gnosis-safe.io/)\) provides adequate security protections against stolen administrative keys.

## Example Use of a Secure Key Vault

The following is example code that can be used to securely sign a transaction on a server without extracting the private key from the vault, using AWS KMS. Note that this would also require the implementation of Ethereum signature details. Defender Relayers provide this functionality automatically.

```text
import KMS from 'aws-sdk/clients/kms';

public async sign(kms: KMS, keyIdOrAlias: string, payload: Buffer): Promise<Buffer> {
    const params: KMS.SignRequest = {
        KeyId: keyIdOrAlias,
        SigningAlgorithm: 'ECDSA_SHA_256',
        Message: payload,
        MessageType: 'DIGEST',
    };

    const response = await this.kms.sign(params).promise();
    if (Buffer.isBuffer(response.Signature)) {
        return response.Signature;
    }
    throw new Error(`Error using key ${keyIdOrAlias} signing: ${payload.toString()}`);
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/secure-all-administrative-keys?](https://defender.openzeppelin.com/#/advisor/docs/secure-all-administrative-keys?)

