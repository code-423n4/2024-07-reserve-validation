AssetRegistry : Incorrect asset check inside [_register](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/AssetRegistry.sol#L201-L208).

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

As shown above, the function incorrectly validate the invariant check.
