# Constructor Typos

## Constructors

Constructors can be specified in two ways. Up to and including in Solidity v0.4.21, the constructor is a function whose name matches the name of the contract, as you can see here:

`contract MEContract {  
 function MEContract() {  
 // This is the constructor  
 }  
}`

The difficulty with this format is that if the contract name is changed and the constructor function name is not changed, it is no longer a constructor. Likewise, if there is an accidental typo in the naming of the contract and/or constructor, the function is again no longer a constructor. This can cause some pretty nasty, unexpected, and difficult-to-find bugs. Imagine for example if the constructor is setting the owner of the contract for purposes of control.

