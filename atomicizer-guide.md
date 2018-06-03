<!-- TITLE: Atomicizer Guide -->
<!-- SUBTITLE: How to use the Wyvern Atomicizer library contract to combine multiple operations into a single transaction -->

# Wyvern Atomicizer Guide

The Wyvern Atomicizer libary - ([Etherscan](https://etherscan.io/address/wyvernatomicizer.eth), [Solidity code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/WyvernAtomicizer.sol)) - provides a generic way to dynamically combine multiple CALL operations into a single transaction. This may be useful in situations where you want to execute multiple operations atomically, such as selling three CryptoKitties in a single order, deploying a contract and setting an ENS address record, or calling ERC20 `approve` and `transferFrom` with one transaction instead of two. In order to use the Atomicizer library, you must call it from an account capable of executing the `DELEGATECALL` opcode - so standard Ethereum accounts will not work, but Wyvern Registry proxy accounts ([code](https://github.com/ProjectWyvern/wyvern-ethereum/blob/master/contracts/registry/AuthenticatedProxy.sol)) or user-authenticated contracts ([proposal](https://ethereum-magicians.org/t/erc-1077-and-erc-1078-the-magic-of-executable-signed-messages-to-login-and-do-actions/351/9?u=cwgoes)) will. The Atomicizer library was written primarily for the Wyvern Protocol, but it is permissionless and can be used by any Ethereum account for any purpose.

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
  transactions.map(t => (t.calldata.length - 2) / 2), // subtract 2 for '0x', divide by 2 for hex
  transactions.map(t => t.calldata).reduce((x, y) => x + y.slice(2)) // cut off the '0x'
]

const encoded = web3.eth.abi.encodeFunctionCall(atomicize, params)
```

## Examples

Any combination of transactions (within the Ethereum block gas limit) can be atomicized, the following are just a few examples.

### Sending 0.001 Ether to two addresses in the same transaction

Sent through a Wyvern authenticated proxy contract in [0xf74abfddc49b25b6d64e88b49e34573a6c9e5cab65f85206de236f16a89b15c9](https://etherscan.io/tx/0xf74abfddc49b25b6d64e88b49e34573a6c9e5cab65f85206de236f16a89b15c9).

```javascript
const Web3 = require('web3')
const web3 = new Web3()

const atomicizerAddress = '0xC99f70bFD82fb7c8f8191fdfbFB735606b15e5c5' // wyvernatomicizer.eth
const atomicize = {'constant': false, 'inputs': [{'name': 'addrs', 'type': 'address[]'}, {'name': 'values', 'type': 'uint256[]'}, {'name': 'calldataLengths', 'type': 'uint256[]'}, {'name': 'calldatas', 'type': 'bytes'}], 'name': 'atomicize', 'outputs': [], 'payable': false, 'stateMutability': 'nonpayable', 'type': 'function'}

const transactions = [
  {calldata: '0x', value: web3.utils.toWei('0.001'), address: '0x0084a81668b9a978416abeb88bc1572816cc7cac'}, // send 0.001 Ether to 0x0084a81668b9a978416abeb88bc1572816cc7cac
  {calldata: '0x', value: web3.utils.toWei('0.001'), address: '0xa839D4b5A36265795EbA6894651a8aF3d0aE2e68'}  // send 0.001 Ether to 0xa839D4b5A36265795EbA6894651a8aF3d0aE2e68
]

const params = [
  transactions.map(t => t.address),
  transactions.map(t => t.value),
  transactions.map(t => (t.calldata.length - 2) / 2), // subtract 2 for '0x', divide by 2 for hex
  transactions.map(t => t.calldata).reduce((x, y) => x + y.slice(2)) // cut off the '0x'
]

console.log(params)

const encoded = web3.eth.abi.encodeFunctionCall(atomicize, params)

console.log(atomicizerAddress, encoded)
```

### Selling two CryptoKitties with one Wyvern v2 order

Matched in [0xc8b873d08d8c1f4cff43ce430f200a28998b1aebecaa8f6789b08e25c606af47](https://etherscan.io/tx/0xc8b873d08d8c1f4cff43ce430f200a28998b1aebecaa8f6789b08e25c606af47). This example uses the [wyvern-js](https://github.com/ProjectWyvern/wyvern-js) library. Checking the [event logs](https://etherscan.io/tx/0xc8b873d08d8c1f4cff43ce430f200a28998b1aebecaa8f6789b08e25c606af47#eventlog) on Etherscan, you can easily see that both kitties were transferred.

```javascript
const deepcopy = require('deepcopy')
const Web3 = require('web3')
const web3 = new Web3('https://mainnet.infura.io')
const { WyvernProtocol } = require('wyvern-js')
const { schemas } = require('wyvern-schemas')
const protocolInstance = new WyvernProtocol(web3.currentProvider, { network: 'main' })

const CryptoKitties = schemas.main.filter(s => s.name === 'CryptoKitties')[0]

// Predetermined buyer
const buyer = '0x0084a81668B9A978416aBEB88bC1572816cc7cAC'
const seller = '0x0084a81668B9A978416aBEB88bC1572816cc7cAC'

// Two kitties to be sold in one order
const kitties = [
  '591654',
  '570186'
]

const transactions = kitties.map(kitty => {
  const transfer = CryptoKitties.functions.transfer(kitty)
  const encoded = web3.eth.abi.encodeFunctionCall(transfer, [buyer, kitty])
  const calldata = encoded
  const address = transfer.target
  const value = '0'
  return {
    calldata,
    address,
    value
  }
})

const atomicized = protocolInstance.wyvernAtomicizer.atomicize.getABIEncodedTransactionData(
  transactions.map(t => t.address),
  transactions.map(t => t.value),
  transactions.map(t => (t.calldata.length - 2) / 2), // subtract 2 for '0x', divide by 2 for hex
  transactions.map(t => t.calldata).reduce((x, y) => x + y.slice(2)) // cut off the '0x'
)

const calldata = atomicized
const replacementPattern = '0x' // exact match, no replacement

const sellOrder = {
  exchange: WyvernProtocol.getExchangeContractAddress('main'),
  maker: seller,
  taker: buyer,
  makerRelayerFee: '0',
  takerRelayerFee: '0',
  makerProtocolFee: '0',
  takerProtocolFee: '0',
  feeRecipient: seller,
  feeMethod: '0',
  side: '1',
  saleKind: '0',
  target: WyvernProtocol.getAtomicizerContractAddress('main'),
  howToCall: '1', // DELEGATECALL to library
  calldata: calldata,
  replacementPattern: replacementPattern,
  staticTarget: '0x0000000000000000000000000000000000000000',
  staticExtradata: '0x',
  paymentToken: '0x0000000000000000000000000000000000000000',
  basePrice: '0',
  extra: '0',
  listingTime: '0',
  expirationTime: '0',
  salt: WyvernProtocol.generatePseudoRandomSalt().toString()
}

// Create the matching buy-side order
const buyOrder = deepcopy(sellOrder)
buyOrder.side = 0
buyOrder.maker = buyer
buyOrder.taker = seller
buyOrder.feeRecipient = '0x0000000000000000000000000000000000000000'

try {
  (async () => {
    // Check that orders can match
    const ordersCanMatch = await protocolInstance.wyvernExchange.ordersCanMatch_.callAsync(
      [buyOrder.exchange, buyOrder.maker, buyOrder.taker, buyOrder.feeRecipient, buyOrder.target, buyOrder.staticTarget, buyOrder.paymentToken, sellOrder.exchange, sellOrder.maker, sellOrder.taker, sellOrder.feeRecipient, sellOrder.target, sellOrder.staticTarget, sellOrder.paymentToken],
      [buyOrder.makerRelayerFee, buyOrder.takerRelayerFee, buyOrder.makerProtocolFee, buyOrder.takerProtocolFee, buyOrder.basePrice, buyOrder.extra, buyOrder.listingTime, buyOrder.expirationTime, buyOrder.salt, sellOrder.makerRelayerFee, sellOrder.takerRelayerFee, sellOrder.makerProtocolFee, sellOrder.takerProtocolFee, sellOrder.basePrice, sellOrder.extra, sellOrder.listingTime, sellOrder.expirationTime, sellOrder.salt],
      [buyOrder.feeMethod, buyOrder.side, buyOrder.saleKind, buyOrder.howToCall, sellOrder.feeMethod, sellOrder.side, sellOrder.saleKind, sellOrder.howToCall],
      buyOrder.calldata,
      sellOrder.calldata,
      buyOrder.replacementPattern,
      sellOrder.replacementPattern,
      buyOrder.staticExtradata,
      sellOrder.staticExtradata
      )
    console.log('ordersCanMatch: ' + ordersCanMatch)

    // Encode the atomic match call
    const matchEncoded = await protocolInstance.wyvernExchange.atomicMatch_.getABIEncodedTransactionData(
      [buyOrder.exchange, buyOrder.maker, buyOrder.taker, buyOrder.feeRecipient, buyOrder.target, buyOrder.staticTarget, buyOrder.paymentToken, sellOrder.exchange, sellOrder.maker, sellOrder.taker, sellOrder.feeRecipient, sellOrder.target, sellOrder.staticTarget, sellOrder.paymentToken],
      [buyOrder.makerRelayerFee, buyOrder.takerRelayerFee, buyOrder.makerProtocolFee, buyOrder.takerProtocolFee, buyOrder.basePrice, buyOrder.extra, buyOrder.listingTime, buyOrder.expirationTime, buyOrder.salt, sellOrder.makerRelayerFee, sellOrder.takerRelayerFee, sellOrder.makerProtocolFee, sellOrder.takerProtocolFee, sellOrder.basePrice, sellOrder.extra, sellOrder.listingTime, sellOrder.expirationTime, sellOrder.salt],
      [buyOrder.feeMethod, buyOrder.side, buyOrder.saleKind, buyOrder.howToCall, sellOrder.feeMethod, sellOrder.side, sellOrder.saleKind, sellOrder.howToCall],
      buyOrder.calldata,
      sellOrder.calldata,
      buyOrder.replacementPattern,
      sellOrder.replacementPattern,
      buyOrder.staticExtradata,
      sellOrder.staticExtradata,
      [27, 27], // No signatures, order previously approved
      ['0x0000000000000000000000000000000000000000000000000000000000000000', '0x0000000000000000000000000000000000000000000000000000000000000000', '0x0000000000000000000000000000000000000000000000000000000000000000', '0x0000000000000000000000000000000000000000000000000000000000000000', '0x0000000000000000000000000000000000000000000000000000000000000000']
    )
    console.log(matchEncoded)
  })()
} catch (e) {
  console.log(e)
}
```