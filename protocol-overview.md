<!-- TITLE: Protocol Overview -->
<!-- SUBTITLE: An overview of the Wyvern Protocol -->

# Summary

Wyvern is a protocol for the decentralized exchange of nonfungible digital assets. Like decentralized fungible token exchange protocols, Etherdelta[0] and 0x[1], Wyvern uses a hybrid model: signed orders are transmitted and stored off-chain, while all state transitions are settled on-chain, meaning that protocol users need not trust any counterparty with custody of their assets. Unlike prior protocols, Wyvern is representation-agnostic: the protocol uses a proxy account system to abstract over the space of Ethereum transactions, allowing arbitrary state transitions to be bought and sold without the deployment of any additional smart contracts. Wyvern supports buy- and sell-side orders, fixed price and Dutch auction pricing, and asset criteria specification — orders may be placed for specific assets, or for any assets with specific properties.

The protocol is deployed as [a set of Solidity smart contracts](https://github.com/ProjectWyvern/wyvern-ethereum) on the Ethereum mainnet, with provision for automatic upgrades enacted by the [Wyvern DAO](https://dao.projectwyvern.com). All deployed mainnet versions have undergone third-party audit, and Wyvern is used in production by the [Wyvern Exchange](https://exchange.projectwyvern.com).
# Protocol Reference
- [Contract Architecture](/contract-architecture)
- [Order Schema](/order-schema)
# Examples / Tutorials
## Marketplaces
- [Wyvern Exchange sourcecode](https://github.com/ProjectWyvern/exchange.projectwyvern.com)
## Automation
- [CryptoKitties resale script](https://github.com/ProjectWyvern/automation/tree/master/cryptokitties-resale)
## External Integrations
- [Discord chatbot](https://github.com/ProjectWyvern/integrations/tree/master/discord)

[0]: https://etherdelta.com
[1]: https://0xproject.com
