1. AssetRegistry : Incorrect asset check inside [_register](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/AssetRegistry.sol#L201-L208).

[(AssetRegistry.sol#L201-L208)](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/AssetRegistry.sol#L201-L208)

```solidity
    function _register(IAsset asset) internal returns (bool registered) {
        require(
            !_erc20s.contains(address(asset.erc20())) || assets[asset.erc20()] == asset, -- incorrect check
            "duplicate ERC20 detected"
        );

        registered = _registerIgnoringCollisions(asset);
    }
```

The above logic is incorrect, there should not be asset in the assets map.

require(
            !(_erc20s.contains(address(asset.erc20())) || assets[asset.erc20()] == asset), -- updated check
            "duplicate ERC20 detected"
        );

As shown above, the function incorrectly validate the invariant check. The assets[erc20] = asset; will be overwritten unknowingly.

2. RToken : Lack of slippage check for [redeemTo](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/RToken.sol#L183-L186).

Redeem function used to redeem the RToken for collateral tokens. This function transfers the collateral from the backingManager to the caller or redeemer. The issue here is, if the basket is manipulated, there will be loss of collateral to user in the form of slippage.

https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/RToken.sol#L203-L224

```solidity
        uint192 baskets = _scaleDown(caller, amount);
        emit Redemption(caller, recipient, amount, baskets);

        (address[] memory erc20s, uint256[] memory amounts) = basketHandler.quote(
            baskets,
            false,
            FLOOR
        );

        // === Interactions ===

        for (uint256 i = 0; i < erc20s.length; ++i) {
            if (amounts[i] == 0) continue;

            // Send withdrawal
            // slither-disable-next-line arbitrary-send-erc20
            IERC20Upgradeable(erc20s[i]).safeTransferFrom(
                address(backingManager),
                recipient,
                amounts[i]
            );
        }
```

Fix - provide the slippage input to prevent the users from unexpected loss.
