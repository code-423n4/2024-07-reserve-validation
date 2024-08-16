# [L-01] The protocol uses `_msgSender()` extensively, but not everywhere
The code is using `_msgSender` to allow meta-transactions. It works by using doing a call to _msgSender() instead of querying msg.sender directly, because the method allows those special transactions.
The problem is using of the meta transactions is not consistent everywhere,
like [here in Dutch Auction.sol](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/plugins/trading/DutchTrade.sol#L156) while initiating it passes the caller as `msg.sender` and whereas  [here in Broker.sol](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/Broker.sol#L143)
while creating the Dutch Auction it passes the caller as `_msgSender`
https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/Broker.sol#L130
which breaks this intent and will not allow meta-transactions.
## Recommendation
Change the code to use _msgSender() instead of msg.sender.

# [L-02] No check for sequencer uptime can lead to dutch auctions executing at bad prices
https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/plugins/trading/DutchTrade.sol#L199-226
Since the project is going to be deployed on the Ethereum mainnet and L2s like Arbitrum, we must consider the existence of the Sequencer and the risks it may pose. While using Dutch Auctions on L2s, we notice that the sequencer uptime isn't taken into account
## Vulnerability
When purchasing from dutch auctions on L2s there is no considering of sequencer uptime. If the sequencer goes offline during the the auction period then the auction will continue to decrease in price while the sequencer is offline. Once the sequencer comes back online, users will be able to buy tokens from these auctions at prices much lower than market price.

## Recommendation
Use an external uptime feed such as Chainlink's L2 Sequencer Feeds. This would allow the contract to invalidate an auction if the sequencer was ever offline during its duration. 