# Initialize()

## Custom Initialization Functions

A smart contract designates the address which initializes it as the contract's owner. This is a common pattern for granting special privileges such as the ability to withdraw the contract's funds.\
Unfortunately, the initialization function can be called by anyone — even after it has already been called. Allowing anyone to become the owner of the contract and take its funds.\
Code Example:

In the following example, the contract's initialization function sets the caller of the function as its owner. However, **the logic is detached from the contract's constructor, and it does not keep track of the fact that it has already been called**.

```
function initContract() public {
 owner = msg.sender;
}
```

In the Parity multi-sig wallet, this initialization function was detached from the wallets themselves and defined in a "library" contract. Users were expected to initialize their own wallet by calling the library's function via a `delegateCall`. Unfortunately, as in our example, the function did not check if the wallet had already been initialized. Worse, [since the library was a smart contract, anyone could initialize the library itself and call for its destruction](https://github.com/paritytech/parity-ethereum/issues/6995#issuecomment-342409816).

## Initialize()

There are _two?_  bugs that I'm aware of.

### Uninitialized `initialize()` Function

I.e. The contract got deployed and nobody bothered calling `initialize()`, perhaps thinking nobody would notice. The `initializer` modifier only prevents from calling the function twice, it doesn't enforce the caller's address to be the contract's owner.

### Missing `initializer` modifier.

Eg. [Punk Finance got rekt](https://medium.com/punkprotocol/punk-finance-fair-launch-incident-report-984d9e340eb) because the `initialize()` function didn't use the `initializer` modifier, preventing it to be called again!

It is entirely possible to use a custom modifier for the initialize() function,  the point is there should at least be some kind of access control modifier.

## Resources

* [https://www.rekt.news/punkprotocol-rekt/](https://www.rekt.news/punkprotocol-rekt/)

