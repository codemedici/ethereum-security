# Default Visibility

## Function Default Visibility

Functions that do not have a function visibility type specified are `public` by default. This can lead to a vulnerability if a developer forgot to set the visibility and a malicious user is able to make unauthorized or unintended state changes.

Function visibility can be specified as either: public, private, internal, or external. It's important to consider which visibility is best for your smart contract function.

Many smart contract attacks are caused by a developer forgetting or forgoing to use a visibility modifier. The function is then set as public by default, which can lead to unintended state changes.

## State Variable Default Visibility

It's common for developers to explicitly declare function visibility, but not so common to declare variable visibility. State variables can have one of three visibility identifiers: `public`, `internal`, or `private`. Luckily, the default visibility for variables is internal and not public, but even if you intend on declaring a variable as internal, it's important to be explicit so that there are no incorrect assumptions as to who can access the variable.

## Remediation

Functions can be specified as being `external`, `public`, `internal` or `private`. It is recommended to make a conscious decision on which visibility type is appropriate for a function. This can dramatically reduce the attack surface of a contract system.

## Examples

#### visibility\_not\_set.sol

```text
123456789101112131415161718192021/*
 * @source: https://github.com/sigp/solidity-security-blog#visibility
 * @author: SigmaPrime 
 * Modified by Gerhard Wagner
 */

pragma solidity ^0.4.24;

contract HashForEther {

    function withdrawWinnings() {
        // Winner if the last 8 hex characters of the address are 0. 
        require(uint32(msg.sender) == 0);
        _sendWinnings();
     }

     function _sendWinnings() {
         msg.sender.transfer(this.balance);
     }
}

```

**visibility\_not\_set.yaml**

```text
123456789101112description: Default function visibility
issues:
- id: SWC-100
  count: 2
  locations:
  - bytecode_offsets: {}
    line_numbers:
      visibility_not_set.sol: [11]
  - bytecode_offsets: {}
    line_numbers:
      visibility_not_set.sol: [17]

```

#### visibility\_not\_set\_fixed.sol

```text
123456789101112131415161718192021/*
 * @source: https://github.com/sigp/solidity-security-blog#visibility
 * @author: SigmaPrime
 * Modified by Gerhard Wagner
 */

pragma solidity ^0.4.24;

contract HashForEther {

    function withdrawWinnings() public {
        // Winner if the last 8 hex characters of the address are 0.
        require(uint32(msg.sender) == 0);
        _sendWinnings();
     }

     function _sendWinnings() internal{
         msg.sender.transfer(this.balance);
     }
}

```

**visibility\_not\_set\_fixed.yaml**

```text
123456description: Default function visibility
issues:
- id: SWC-100
  count: 0
  locations: []

```

## Resources

* [https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/function-default-visibility.md](https://github.com/KadenZipfel/smart-contract-attack-vectors/blob/master/vulnerabilities/function-default-visibility.md)
* [https://swcregistry.io/docs/SWC-100](https://swcregistry.io/docs/SWC-100)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/\#explicitly-mark-visibility-in-functions-and-state-variables](https://consensys.github.io/smart-contract-best-practices/recommendations/#explicitly-mark-visibility-in-functions-and-state-variables)
* [https://swcregistry.io/docs/SWC-108](https://swcregistry.io/docs/SWC-108)
* [https://consensys.github.io/smart-contract-best-practices/recommendations/\#explicitly-mark-visibility-in-functions-and-state-variables](https://consensys.github.io/smart-contract-best-practices/recommendations/#explicitly-mark-visibility-in-functions-and-state-variables)
* [Ethereum Smart Contract Best Practices - Explicitly mark visibility in functions and state variables](https://consensys.github.io/smart-contract-best-practices/recommendations/#explicitly-mark-visibility-in-functions-and-state-variables)
* [SigmaPrime - Visibility](https://github.com/sigp/solidity-security-blog#visibility)

