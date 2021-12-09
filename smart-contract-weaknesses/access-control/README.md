# Access Control

## Introduction

Access Control issues are common in all programs, not just smart contracts. In fact, it's number 5 on the OWASP top 10. One usually accesses a contract's functionality through its public or external functions. While insecure visibility settings give attackers straightforward ways to access a contract's private values or logic, access control bypasses are sometimes more subtle. These vulnerabilities can occur when contracts use the deprecated `tx.origin` to validate callers, handle large authorization logic with lengthy `require` and make reckless use of `delegatecall` in [proxy libraries](https://blog.openzeppelin.com/proxy-libraries-in-solidity-79fbe4b970fd/) or [proxy contracts](https://blog.indorse.io/ethereum-upgradeable-smart-contract-strategies-456350d0557c).

Loss: estimated at 150,000 ETH (\~30M USD at the time)

Real World Impact:\
• Parity Multi-sig bug 1\
• Parity Multi-sig bug 2\
• Rubixi

## Resources

[https://dasp.co/#item-2](https://dasp.co/#item-2)
