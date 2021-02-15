# Preventative Techniques

## Preventative Techniques

## Preventative Techniques

Solidity provides the **`library`** keyword for implementing library contracts \([see the docs](https://solidity.readthedocs.io/en/latest/contracts.html?highlight=library#libraries) for further details\). This ensures the library contract is stateless and non-self-destructable. Forcing libraries to be stateless mitigates the complexities of storage context demonstrated in this section. Stateless libraries also prevent attacks wherein attackers modify the state of the library directly in order to affect the contracts that depend on the library’s code.  
As a general rule of thumb, when using `DELEGATECALL` pay careful attention to the possible calling context of both the library contract and the calling contract, and whenever possible build stateless libraries.

