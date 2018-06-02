<!-- TITLE: Atomicizer Guide -->
<!-- SUBTITLE: How to use the Wyvern Atomicizer library contract to combine multiple operations into a single transaction -->

# Wyvern Atomicizer Guide

The Wyvern Atomicizer libary - ([Etherscan](https://etherscan.io/address/wyvernatomicizer.eth), [Solidity code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/WyvernAtomicizer.sol)) - provides a generic way to dynamically combine multiple CALL operations into a single transaction. This may be useful in situations where you want to execute multiple operations atomically, such as selling three CryptoKitties in a single order, deploying a contract and setting an ENS address record, or calling ERC20 `approve` and `transferFrom` with one transaction.

## Basics

The Atomicizer contract is a library - so you can only DELEGATECALL to it - exposing just one function:

```solidity
function atomicize (address[] addrs, uint[] values, uint[] calldataLengths, bytes calldatas)
```

These four parameters allow you to pass a list of CALL operations (standard Ethereum transactions), to be executed in series. `addrs`, `values`, and `calldataLengths` must all have the same number of elements, and the length of `calldatas` must be equal to the sum of `calldataLengths` (this is just a way to pass a two-dimensional byte array, which Solidity does not natively support).

For each index, in series, the library parses out the calldata for that index then executes:

```solidity
require(addrs[i].call.value(values[i])(calldata));
```

This means that execution is "all-or-nothing": if any of the operations fail, the Atomicizer library will stop execution, refund remaining gas, and throw an error, reverting all previous state changes - so, if you atomicize three `transfer` calls of individual CryptoKitties, either all of the CryptoKitties will be transferred or none of them will.