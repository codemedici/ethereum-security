# Initialize\(\)

There are _two?_  bugs that I'm aware of.

## Uninitialized `initialize()` Function

I.e. The contract got deployed and nobody bothered to call it

## Missing `initializer` modifier.

Eg. [Punk Finance got rekt](https://medium.com/punkprotocol/punk-finance-fair-launch-incident-report-984d9e340eb) because the `initialize()` function didn't use the `initializer` modifier, preventing it to be called again!

It is entirely possible to use a custom modifier for the initialize\(\) function,  the point is there should at least be some kind of access control modifier.



