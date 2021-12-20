# Table of contents

* [Ethereum Security](README.md)

## Smart Contract Weaknesses

* [Reentrancy](smart-contract-weaknesses/reentrancy/README.md)
  * [Examples](smart-contract-weaknesses/reentrancy/examples.md)
  * [Preventative Techniques](smart-contract-weaknesses/reentrancy/preventative-techniques/README.md)
    * [Pitfalls in Reentrancy Solutions](smart-contract-weaknesses/reentrancy/preventative-techniques/pitfalls-in-reentrancy-solutions.md)
* [Unexpected Ether](smart-contract-weaknesses/unexpected-ether.md)
* [Integer Under/Overflow](smart-contract-weaknesses/integer-under-over-flow.md)
* [Price Oracle Manipulation](smart-contract-weaknesses/price-oracle-manipulation/README.md)
  * [Synthetix MKR Manipulation](smart-contract-weaknesses/price-oracle-manipulation/synthetix-mkr-manipulation.md)
  * [Synthetix sKRW Oracle Malfunction](smart-contract-weaknesses/price-oracle-manipulation/synthetix-skrw-oracle-malfunction.md)
  * [bZx First Hack](smart-contract-weaknesses/price-oracle-manipulation/bzx-first-hack.md)
  * [bZx Second Hack](smart-contract-weaknesses/price-oracle-manipulation/bzx-second-hack.md)
  * [bZx and Fulcrum (Bug Bounty)](smart-contract-weaknesses/price-oracle-manipulation/bzx-and-fulcrum-bug-bounty.md)
  * [yVault (Bug Bounty)](smart-contract-weaknesses/price-oracle-manipulation/yvault-bug.md)
  * [Harvest Finance Hack](smart-contract-weaknesses/price-oracle-manipulation/harvest-finance-hack.md)
  * [DDEX (Bug Bounty)](smart-contract-weaknesses/price-oracle-manipulation/ddex-hydro-protocol.md)
* [Inadherence to Standards](smart-contract-weaknesses/inadherence-to-standards.md)
* [Access Control](smart-contract-weaknesses/access-control/README.md)
  * [Initialize()](smart-contract-weaknesses/access-control/initialize.md)
  * [Default Visibility](smart-contract-weaknesses/access-control/default-visibility.md)
  * [Default Visibilities](smart-contract-weaknesses/access-control/default-visibilities.md)
  * [Tx.origin Authentication](smart-contract-weaknesses/access-control/tx.origin-authentication.md)
* [Floating Point and Precision](smart-contract-weaknesses/floating-point-and-precision.md)
* [Constructors](smart-contract-weaknesses/constructors.md)
* [delegatecall](smart-contract-weaknesses/delegatecall.md)
* [Unchecked CALL Return Values](smart-contract-weaknesses/unchecked-call-return-values.md)
* [Entropy Illusion](smart-contract-weaknesses/entropy-illusion/README.md)
  * [Timestamp Dependence](smart-contract-weaknesses/entropy-illusion/timestamp-dependence.md)
* [Assembly Attacks](smart-contract-weaknesses/assembly-attacks.md)
* [Signature Replay](smart-contract-weaknesses/signature-replay.md)
* [Unprotected Selfdestruct](smart-contract-weaknesses/unprotected-selfdestruct.md)
* [Shadowing State Variables](smart-contract-weaknesses/shadowing-state-variables.md)
* [Assert Violation](smart-contract-weaknesses/assert-violation.md)
* [Outdated Compiler Version](smart-contract-weaknesses/outdated-compiler-version.md)
* [Honeypot](smart-contract-weaknesses/honeypot.md)
* [Floating Pragma](smart-contract-weaknesses/floating-pragma.md)
* [Insufficient Gas Griefing](smart-contract-weaknesses/insufficient-gas-griefing.md)
* [Uninitialized Storage pointers](smart-contract-weaknesses/uninitialized-storage-pointers.md)
* [Unhandled Exception](smart-contract-weaknesses/unhandled-exception.md)
* [Short Address/Parameter Attack](smart-contract-weaknesses/short-address-parameter-attack.md)
* [Function Overriding](smart-contract-weaknesses/function-overriding.md)
* [External Contract Referencing](smart-contract-weaknesses/external-contract-referencing.md)
* [Unsafe Functions](smart-contract-weaknesses/unsafe-functions.md)
* [Design Flaws](smart-contract-weaknesses/design-flaws.md)
* [Deprecated Code](smart-contract-weaknesses/deprecated-code.md)
* [Denial of Service](smart-contract-weaknesses/denial-of-service.md)
* [MEV](smart-contract-weaknesses/mev/README.md)
  * [MEV Exploits](smart-contract-weaknesses/mev/mev-exploits/README.md)
    * [Frontrunning](smart-contract-weaknesses/mev/mev-exploits/frontrunning.md)
    * [Backrunning](smart-contract-weaknesses/mev/mev-exploits/backrunning.md)
    * [Uncle-bandit](smart-contract-weaknesses/mev/mev-exploits/uncle-bandit.md)
    * [Sandwiching](smart-contract-weaknesses/mev/mev-exploits/sandwiching.md)
  * [MEV Mitigation](smart-contract-weaknesses/mev/mev-mitigation/README.md)
    * [Flashbots](smart-contract-weaknesses/mev/mev-mitigation/flashbots.md)

## Auditing

* [Checklists](auditing/checklists.md)
* [Auditing process](auditing/auditing-process.md)
* [Program Analysis](auditing/program-analysis/README.md)
  * [Echidna](auditing/program-analysis/echidna.md)
  * [Manticore](auditing/program-analysis/manticore.md)
  * [Mythril](auditing/program-analysis/mythril.md)
  * [Slither](auditing/program-analysis/slither.md)
* [Example Audits](auditing/example-audits.md)
* [Rekt](auditing/rekt.md)

## Best Practices

* [Development Guidelines](best-practices/development-guidelines.md)
* [Protocol specific recommendations](best-practices/protocol-specific-recommendations.md)
* [Solidity specific recommendations](best-practices/solidity-specific-recommendations.md)
* [Token Specific Recommendations](best-practices/token-specific-recommendations.md)

## Operations

* [OZ Defender](operations/oz-defender.md)
* [Key Management](operations/key-management/README.md)
  * [Use multiple signatures for critical administrative tasks](operations/key-management/use-multiple-signatures-for-critical-administrative-tasks.md)
  * [Secure All Administrative Keys](operations/key-management/secure-all-administrative-keys.md)
* [Post-Mortem Analysis](operations/post-mortem-analysis.md)
* [Emergency Response Plan](operations/emergency-response-plan.md)

## Monitoring

* [Data Out of Sync](monitoring/data-out-of-sync.md)
* [Drop In System Funds](monitoring/drop-in-system-funds.md)
* [Priviledged Administration Transactions](monitoring/priviledged-administration-transactions.md)
* [Spikes in Account Activity](monitoring/spikes-in-account-activity.md)
* [Collateral Ratios](monitoring/collateral-ratios.md)
* [Dependency Changes](monitoring/dependency-changes.md)
* [Large Value Transactions](monitoring/large-value-transactions.md)
* [Expiring Assets](monitoring/expiring-assets.md)
* [Unused Tokens or Funds](monitoring/unused-tokens-or-funds.md)
* [Network Congestion](monitoring/network-congestion.md)
* [Spikes in Low Value Transactions](monitoring/spikes-in-low-value-transactions.md)
* [Spikes in Function Calls](monitoring/spikes-in-function-calls.md)
* [Spikes in Failed Transactions](monitoring/spikes-in-failed-transactions.md)
* [Significant Price Changes](monitoring/significant-price-changes.md)
* [Asset Attacks and Issues](monitoring/asset-attacks-and-issues.md)

## Testing

* [Test Contract Upgrades](testing/test-contract-upgrades.md)
* [Assert Revert Reasons](testing/assert-revert-reasons.md)
* [Test Events Emission](testing/test-events-emission.md)
* [Achieve High Smart Contract Test Coverage](testing/achieve-high-smart-contract-test-coverage.md)

## Development

* [Implement Reentrancy Protections](development/implement-reentrancy-protections.md)
* [Recast Variables Safely](development/recast-variables-safely.md)
* [Access Arrays Using Enumeration and Pagination](development/access-arrays-using-enumeration-and-pagination.md)
* [Prevent Replay Attacks When Using Signatures](development/prevent-replay-attacks-when-using-signatures.md)
* [Use the Offer-Accept Pattern for Transferring Admin Role](development/use-the-offer-accept-pattern-for-transferring-admin-role.md)
* [Control Growth of Arrays](development/control-growth-of-arrays.md)
* [Avoid Packed Encoding When Hashing](development/avoid-packed-encoding-when-hashing.md)
* [Do Not Use Solidity's transfer Function](development/do-not-use-soliditys-transfer-function.md)
* [Use Low-Level Calls Carefully](development/use-low-level-calls-carefully.md)
* [Use PullPayment When Sending ETH](development/use-pullpayment-when-sending-eth.md)
* [Declare Constants Explicitly](development/declare-constants-explicitly.md)
* [Include Revert Reasons](development/include-revert-reasons.md)
* [Minimize Division Errors](development/minimize-division-errors.md)
* [Do Not Track Time With Block Numbers](development/do-not-track-time-with-block-numbers.md)
* [Use Indexed Event Parameters](development/use-indexed-event-parameters.md)
* [Emit Events on All State Changes](development/emit-events-on-all-state-changes.md)
* [EIP-1884 Stop Using transfer()](development/eip-1884-stop-using-transfer.md)

## DeFi

* [Overview of Risks](defi/overview-of-risks.md)
* [DeFi Best Practices](defi/defi-best-practices.md)
* [Flashloans](defi/flashloans.md)

## Blockchain Misc

* [Attack Surfaces of Hyperledger Fabric](blockchain-misc/attack-surfaces-of-hyperledger-fabric.md)
* [Blockchain for Cybersecurity Infrastructure](blockchain-misc/blockchain-for-cybersecurity-infrastructure.md)
* [Bitcoin and Ethereum Wallet Cracking](blockchain-misc/bitcoin-and-ethereum-wallet-cracking.md)
