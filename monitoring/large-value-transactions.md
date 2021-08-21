---
description: >-
  Notify and review all protocol transactions involving significantly large
  amounts of assets.
---

# Large Value Transactions

### Description

Often, large-scale hacks are performed with few transactions that operate over significant amounts of assets. Therefore, this type of interaction with the protocol might indicate attempts to manipulate or steal funds by exploiting a newly discovered vulnerability. In order to avoid repetitive attacks and potentially larger losses, monitor and review all large transactions. Even though high-value transactions may not be easily distinguishable from “whales” legitimately using the protocol, they still should be flagged for close inspection.

### Example

Implement a client to monitor all mined transactions and to review events logged by critical user-facing functions to identify transactions with asset values above a predefined threshold. Combine that information with off-chain price data from trusted sources to convert values of token assets to a stable currency, and trigger an alert to administrators or stakeholders if the total value moved by the transaction is higher than a predefined threshold.

