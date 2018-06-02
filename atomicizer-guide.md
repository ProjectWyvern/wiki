<!-- TITLE: Atomicizer Guide -->
<!-- SUBTITLE: How to use the Wyvern Atomicizer library contract to combine multiple operations into a single transaction -->

# Wyvern Atomicizer Guide

The Wyvern Atomicizer libary - ([Etherscan](https://etherscan.io/address/wyvernatomicizer.eth), [Solidity code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/WyvernAtomicizer.sol)) - provides a generic way to dynamically combine multiple CALL operations into a single transaction. This may be useful in situations where you want to execute multiple operations atomically, such as selling three CryptoKitties in a single order, deploying a contract and setting an ENS address record, or calling ERC20 `approve` and `transferFrom` with one transaction instead of two. In order to use the Atomicizer library, you must call it from an account capable of executing the `DELEGATECALL` opcode - so standard Ethereum accounts will not work, but Wyvern Registry proxy accounts ([code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/registry/AuthenticatedProxy.sol)) or user-authenticated contracts ([proposal](https://ethereum-magicians.org/t/erc-1077-and-erc-1078-the-magic-of-executable-signed-messages-to-login-and-do-actions/351/9?u=cwgoes)) will.

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

This means that execution is "all-or-nothing". If any of the operations fail, the Atomicizer library will stop execution, refund remaining gas, and throw an error, reverting all previous state changes - so, if you atomicize three `transfer` calls of individual CryptoKitties, either all of the CryptoKitties will be transferred or none of them will.

## Usage

Using a standard web3 API library, construct the transactions as you would normally, but instead of calling `send()`, call `encodeABI()` and then pass them to the Atomicizer, as follows:

```javascript
const atomicizerAddress = '0xC99f70bFD82fb7c8f8191fdfbFB735606b15e5c5' // wyvernatomicizer.eth
const atomicize = {'constant': false, 'inputs': [{'name': 'addrs', 'type': 'address[]'}, {'name': 'values', 'type': 'uint256[]'}, {'name': 'calldataLengths', 'type': 'uint256[]'}, {'name': 'calldatas', 'type': 'bytes'}], 'name': 'atomicize', 'outputs': [], 'payable': false, 'stateMutability': 'nonpayable', 'type': 'function'}

const transactions = [
   {calldata: '...', value: '...', address: '...'},
	 ...
]

const params = [
  transactions.map(t => t.address),
  transactions.map(t => t.value),
  transactions.map(t => t.calldata.length - 2), // subtract 2 for '0x'
  transactions.map(t => t.calldata).reduce((x, y) => x + y.slice(2)) // cut off the '0x'
]

const encoded = web3.eth.abi.encodeFunctionCall(atomicize, params)
```

## Examples

### Sending 0.001 Ether to two addresses in the same transaction

```javascript
const Web3 = require('web3')
const web3 = new Web3()

const atomicizerAddress = '0xC99f70bFD82fb7c8f8191fdfbFB735606b15e5c5' // wyvernatomicizer.eth
const atomicize = {'constant': false, 'inputs': [{'name': 'addrs', 'type': 'address[]'}, {'name': 'values', 'type': 'uint256[]'}, {'name': 'calldataLengths', 'type': 'uint256[]'}, {'name': 'calldatas', 'type': 'bytes'}], 'name': 'atomicize', 'outputs': [], 'payable': false, 'stateMutability': 'nonpayable', 'type': 'function'}

const transactions = [
  {calldata: '0x', value: web3.utils.toWei(0.001), address: '0x0084a81668b9a978416abeb88bc1572816cc7cac'}, // send 0.001 Ether to 0x0084a81668b9a978416abeb88bc1572816cc7cac
  {calldata: '0x', value: web3.utils.toWei(0.001), address: '0xa839D4b5A36265795EbA6894651a8aF3d0aE2e68'}  // send 0.001 Ether to 0xa839D4b5A36265795EbA6894651a8aF3d0aE2e68
]

const params = [
  transactions.map(t => t.address),
  transactions.map(t => t.value),
  transactions.map(t => t.calldata.length - 2), // subtract 2 for '0x'
  transactions.map(t => t.calldata).reduce((x, y) => x + y.slice(2)) // cut off the '0x'
]

console.log(params)

const encoded = web3.eth.abi.encodeFunctionCall(atomicize, params)

console.log(atomicizerAddress, encoded)
```

Sent through a Wyvern authenticated proxy contract in [0xf74abfddc49b25b6d64e88b49e34573a6c9e5cab65f85206de236f16a89b15c9](https://etherscan.io/tx/0xf74abfddc49b25b6d64e88b49e34573a6c9e5cab65f85206de236f16a89b15c9).