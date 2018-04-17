<!-- TITLE: Contract Architecture -->
<!-- SUBTITLE: Wyvern Protocol Contract Architecture -->

# Motivation
Wyvern's Solidity contracts are structured to (a) provide the necessary least-authority permissioning and separation of concerns, (b) facilitate user-transparent core protocol upgrades, and (c) maximize code understandability, in that order of precedence.
# Contracts
## Registry
The [registry contracts](https://github.com/ProjectWyvern/wyvern-ethereum/tree/master/contracts/registry) proxy user authentication. Wyvern Protocol users transfer assets to a personal proxy contract and approve token transfers through a token proxy contract. This has two advantages over approving transfers from or transferring assets to the exchange contracts directly: it vastly reduces the attack surface of the complex exchange logic (as the exchange contracts never need to hold assets themselves), and it allows users to use new protocol versions without moving assets to new contracts.
### AuthenticatedProxy
### TokenTransferProxy
### ProxyRegistry
## Exchange
The exchange contracts describe
### Exchange
### ExchangeCore
### SaleKindInterface