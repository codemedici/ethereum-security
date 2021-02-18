# SWC-100 Function Default Visibility

### Relationships

[CWE-710: Improper Adherence to Coding Standards](https://cwe.mitre.org/data/definitions/710.html)

### Description

Functions that do not have a function visibility type specified are `public` by default. This can lead to a vulnerability if a developer forgot to set the visibility and a malicious user is able to make unauthorized or unintended state changes.

### Remediation

Functions can be specified as being `external`, `public`, `internal` or `private`. It is recommended to make a conscious decision on which visibility type is appropriate for a function. This can dramatically reduce the attack surface of a contract system.

### References

* [Ethereum Smart Contract Best Practices - Explicitly mark visibility in functions and state variables](https://consensys.github.io/smart-contract-best-practices/recommendations/#explicitly-mark-visibility-in-functions-and-state-variables)
* [SigmaPrime - Visibility](https://github.com/sigp/solidity-security-blog#visibility)

### Contract Samples

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

