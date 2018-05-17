<!-- TITLE: Feature Wishlist -->
<!-- SUBTITLE: Desirable features for possible future versions of the Wyvern Protocol -->

# Feature Wishlist

* Methods of sale
	* English Auction (requires asset escrow to prevent griefing attacks)
	* Second-price Vickrey auction
* Order kinds
  * Transaction to transaction, directly without ERC20 intermediation
  * Fungible to fungible, without necessarily requiring the ERC20 spec
  * Cross-side static calls, can enforce e.g. token proportionality generically (!)
  * Proxy contract restriction - prevent function execution for escrow - upgrade by authorizing double-proxy contract with new registry
  * Completely separate order executor from order maker/taker - signed messages for proxy contracts, fees back to relayer (or market)