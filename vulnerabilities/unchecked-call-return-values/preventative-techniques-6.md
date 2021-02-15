# Preventative Techniques

## Preventative Techniques

## Preventative Techniques

Whenever possible, use the transfer function rather than send, as transfer will revert if the external transaction reverts. If send is required, always check the return value.  
The use of low level "call" should be avoided whenever possible. It can lead to unexpected behavior if return values are not handled properly.  
A more robust [recommendation](http://bit.ly/2CSdF7y) is to adopt a withdrawal pattern. In this solution, each user must call an isolated withdraw function that handles the sending of ether out of the contract and deals with the consequences of failed send transactions. The idea is to logically isolate the external send functionality from the rest of the codebase, and place the burden of a potentially failed transaction on the end user calling the withdraw function.

