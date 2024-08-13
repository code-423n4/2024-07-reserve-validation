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

