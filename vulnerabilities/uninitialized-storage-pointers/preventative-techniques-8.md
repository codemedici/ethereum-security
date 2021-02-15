# Preventative Techniques

The Solidity compiler shows a warning for unintialized storage variables; developers should pay careful attention to these warnings when building smart contracts. The current version of Mist \(0.10\) doesn’t allow these contracts to be compiled. It is often good practice to explicitly use the memory or storage specifiers when dealing with complex types, to ensure they behave as expected.

