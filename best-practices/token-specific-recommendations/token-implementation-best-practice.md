# Token Implementation Best Practice

## [Token Implementation Best Practice](https://consensys.github.io/smart-contract-best-practices/tokens/) (Consensys)

{% hint style="info" %}
Last checked: **20 August 2021**. Remember to check the [Github page](https://consensys.github.io/smart-contract-best-practices/tokens/) for any updates to this section.
{% endhint %}

Implementing Tokens should comply with other best practices, but also have some unique considerations.

### Comply with the latest standard[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#comply-with-the-latest-standard) <a href="#comply-with-the-latest-standard" id="comply-with-the-latest-standard"></a>

Generally speaking, smart contracts of tokens should follow an accepted and stable standard.

Examples of currently accepted standards include:

* [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [EIP721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md) (non-fungible token)
* More at [eips.ethereum.org](https://eips.ethereum.org/erc#final)

### Be aware of front running attacks on EIP-20[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#be-aware-of-front-running-attacks-on-eip-20) <a href="#be-aware-of-front-running-attacks-on-eip-20" id="be-aware-of-front-running-attacks-on-eip-20"></a>

The EIP-20 token's `approve()` function creates the potential for an approved spender to spend more than the intended amount. A [front running attack](https://consensys.github.io/smart-contract-best-practices/known\_attacks/#transaction-ordering-dependence-tod-front-running) can be used, enabling an approved spender to call `transferFrom()` both before and after the call to `approve()` is processed. More details are available on the [EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#approve), and in [this document](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA\_jp-RLM/edit).

### Prevent transferring tokens to the 0x0 address[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#prevent-transferring-tokens-to-the-0x0-address) <a href="#prevent-transferring-tokens-to-the-0x0-address" id="prevent-transferring-tokens-to-the-0x0-address"></a>

At the time of writing, the "zero" address ([0x0000000000000000000000000000000000000000](https://etherscan.io/address/0x0000000000000000000000000000000000000000)) holds tokens with a value of more than 80$ million.

### Prevent transferring tokens to the contract address[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#prevent-transferring-tokens-to-the-contract-address) <a href="#prevent-transferring-tokens-to-the-contract-address" id="prevent-transferring-tokens-to-the-contract-address"></a>

Consider also preventing the transfer of tokens to the same address of the smart contract.

An example of the potential for loss by leaving this open is the [EOS token smart contract](https://etherscan.io/address/0x86fa049857e0209aa7d9e616f7eb3b3b78ecfdb0) where more than 90,000 tokens are stuck at the contract address.

#### Example[¶](https://consensys.github.io/smart-contract-best-practices/tokens/#example) <a href="#example" id="example"></a>

An example of implementing both the above recommendations would be to create the following modifier; validating that the "to" address is neither 0x0 nor the smart contract's own address:

```
    modifier validDestination( address to ) {
        require(to != address(0x0));
        require(to != address(this) );
        _;
    }
```

The modifier should then be applied to the "transfer" and "transferFrom" methods:

```
    function transfer(address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... your logic ...)
    }

    function transferFrom(address _from, address _to, uint _value)
        validDestination(_to)
        returns (bool) 
    {
        (... your logic ...)
    }
```
