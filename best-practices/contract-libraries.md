# Contract Libraries

## Contract Libraries

## Contract Libraries

There is a lot of existing code available for reuse, both deployed on-chain as callable libraries and off-chain as code template libraries. On-platform libraries, having been deployed, exist as bytecode smart contracts, so great care should be taken before using them in production. However, using well-established existing on-platform libraries comes with many advantages, such as being able to benefit from the latest upgrades, and saves you money and benefits the Ethereum ecosystem by reducing the total number of live contracts in Ethereum.

In Ethereum, the most widely used resource is the [OpenZeppelin suite](https://openzeppelin.org/), an ample library of contracts ranging from implementations of ERC20 and ERC721 tokens, to many flavors of crowdsale models, to simple behaviors commonly found in contracts, such as Ownable, Pausable, or LimitBalance. The contracts in this repository have been extensively tested and in some cases even function as de facto standard implementations. They are free to use, and are built and maintained by Zeppelin together with an ever-growing list of external contributors.

Also from Zeppelin is [ZeppelinOS](https://openzeppelin.com/sdk/) \(SDK\), an open source platform of services and tools to develop and manage smart contract applications securely. ZeppelinOS provides a layer on top of the EVM that makes it easy for developers to launch upgradeable DApps linked to an on-chain library of well-tested contracts that are themselves upgradeable. Different versions of these libraries can coexist on the Ethereum platform, and a vouching system allows users to propose or push improvements in different directions. A set of off-chain tools to debug, test, deploy, and monitor decentralized applications is also provided by the platform.

The project ethpm aims to organize the various resources that are developing in the ecosystem by providing a package management system. As such, their registry provides more examples for you to browse:

Website: [https://www.ethpm.com/](https://www.ethpm.com/)

Repository link: [https://www.ethpm.com/registry](https://www.ethpm.com/registry)

GitHub link: [https://github.com/ethpm](https://github.com/ethpm)

Documentation: [https://www.ethpm.com/docs/integration-guide](https://www.ethpm.com/docs/integration-guide)

