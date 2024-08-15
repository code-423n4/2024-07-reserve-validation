## [L-01] The `portions` parameter should be validated to ensure it sums to 1e18 in the `BasketHandler::quoteCustomRedemption()` function.

The `quoteCustomRedemption()` function is called not only by the `RToken::redeemCustom()` function, but it is also an external view function that is called from the offchain side.

So frontend users or other contacts estimate redemption quatities of the given erc20 tokens by calling this function.

If sum of the `portions` is not 1e18, the function will return incorrect estimations.

So I would suggest moving the following code snippet from `RToken::redeemCustom()` to `BasketHandler::quoteCustomRedemption()`.

```solidity
    uint256 portionsSum;
    for (uint256 i = 0; i < portions.length; ++i) {
        portionsSum += portions[i];
    }
    require(portionsSum == FIX_ONE, "portions do not add up to FIX_ONE");
```

## [L-02] A bidder can lose their assets if he transfers an exceeding amount of tokens to dutch trade through the callback.

The callback mode of Dutch trade lets bidders to send their tokens from callback function, and after running callback, the `bidWithCallback()` function checks the income amount of assets with `amountIn` value.

```solidity
    require(
        amountIn <= buy.balanceOf(address(this)) - balanceBefore,
        "insufficient buy tokens"
    );
```

If the bidder transfers more tokens than `amountIn`, the leftovers will be locked in the trade, and will be transferred to backing manager or traders after settlement which result in the loss of user's token.

Therefore, I suggest add the code snippet to refund leftover tokens if an income amount is greater than expected like the below:

```solidity
    const boughtAmt = buy.balanceOf(address(this)) - balanceBefore;
    require(
        amountIn <= boughtAmt,
        "insufficient buy tokens"
    );
    if (boughtAmt > amountIn)
        buy.transfer(bidder, boughtAmt - amountIn);
```
