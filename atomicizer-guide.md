<!-- TITLE: Atomicizer Guide -->
<!-- SUBTITLE: How to use the Wyvern Atomicizer library contract to combine multiple transactions into one -->

# Wyvern Atomicizer Guide

The Wyvern Atomicizer libary - ([Etherscan](https://etherscan.io/address/wyvernatomicizer.eth), [Solidity code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/WyvernAtomicizer.sol)) - provides a generic way to dynamically combine multiple CALL operations into a single transaction. This may be useful in situations where you want to execute multiple operations atomically, such as selling three CryptoKitties in a single order, deploying a contract and setting an ENS address record, or calling ERC20 `approve` and `transferFrom` with one transaction.