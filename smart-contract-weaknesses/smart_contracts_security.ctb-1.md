# Deprecated Code

## Constructor Typos

### Constructors

Constructors can be specified in two ways. Up to and including in Solidity v0.4.21, the constructor is a function whose name matches the name of the contract, as you can see here:

`contract MEContract {  
 function MEContract() {  
 // This is the constructor  
 }  
}`

The difficulty with this format is that if the contract name is changed and the constructor function name is not changed, it is no longer a constructor. Likewise, if there is an accidental typo in the naming of the contract and/or constructor, the function is again no longer a constructor. This can cause some pretty nasty, unexpected, and difficult-to-find bugs. Imagine for example if the constructor is setting the owner of the contract for purposes of control.

## Use of var to declare variables

**`var`** is deprecated **since version** **`0.4.20`**, but is still be available in versions below 0.5.0 for compatibility. The issue with var is that the type of the variable is assumed from it's first assigned value.

This can cause a number of issues, as assigning 0 or any low value using var will result in an uint8, which will overflow if it exceeds 255. This causes an infinite loop in the case below:

 `for (var i = 0; i < 500; i++) {  
 // do something  
 }`

The issue would be resolved with `uint i`.

