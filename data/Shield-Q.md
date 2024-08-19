## L-01 Discrepency in code & comments

### Lines of Code
https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/plugins/trading/DutchTrade.sol#L37 

https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/plugins/trading/DutchTrade.sol#L165 

### Description
In `DutchTrade.sol` it is mentioned that `30-minutes is the recommended length of auction for a chain with 12-second blocktimes`

but in `init` method this check
```
assert(address(sell_) != address(0) && address(buy_) != address(0) && auctionLength >= 60);
```
allows the auction length to be a min of 60 secs which contradicts the comment


### Recommended Mitigation Steps
```
assert(address(sell_) != address(0) && address(buy_) != address(0) && auctionLength >= 60);
``` 
should be updated to
```
assert(address(sell_) != address(0) && address(buy_) != address(0) && auctionLength >= 1700 && auctionLength <= 1900);
```
so the min auction length is always close to 30 mins


## L-02 Missing check in `BasketHandler.quantityUnsafe`

### Lines of Code
https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/BasketHandler.sol#L359

### Description
In the `BasketHandler.quantityUnsafe` function it is assumed that the asset is correct. But there is no check in place to ensure that the erc20 is registered in the AssetRegistry.

Prior to the other calls to the `BasketHandler._quantity` function it is verified that the erc20 is registered in the AssetRegistry but not in the case of quantityUnsafe function.

### Recommended Mitigation Steps 
Add the `toColl` check in `quantityUnsafe` too like in `quantity`


## L-03 Discrepency in code & comments

### Lines of Code
https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/BasketHandler.sol#L214

https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/BasketHandler.sol#L237

### Description
In `_setPrimeBasket` there is a comment 
`/// @param disableTargetAmountCheck If true, skips the `requireConstantConfigTargets()` check`

but actually even if `disableTargetAmountCheck` is true & `reweightable` is false then too `requireConstantConfigTargets` is called


### Recommended Mitigation Steps
Update the comment to mention both scenario's where `requireConstantConfigTargets` is called/skipped

## L-04 missing check in `setBackupConfig` affects 100 % utilisation of backup collateral assets


### Lines of Code
https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/BasketHandler.sol#L284

https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/mixins/BasketLib.sol#L252

### Description
In `setBackupConfig` we set the `BackupConfig` struct which has 2 params 
```
struct BackupConfig {
    uint256 max; // Maximum number of backup collateral erc20s to use in a basket
    IERC20[] erc20s; // Ordered list of backup collateral ERC20s
}
```

so `max` should always be `=> erc20s.length`  but there is no such check in `setBackupConfig`

now this can also affect the `nextBasket` backup weight logic as there is a for loop with multiple conditions to handle backup weights

```
            for (uint256 j = 0; j < backup.erc20s.length && size < backup.max; ++j) {
                if (goodCollateral(_targetName, backup.erc20s[j], assetRegistry)) size++;
            }
```

now let's assume that `size > backup.erc20s.length` & all backup collaterals are good collaterals so therefore once the loop ends size will be `max - 1` but ideally it should be `backup.erc20s.length` and since the weights for backup tokens are divided equally therefore some backup assets can remain un-utilised


### Recommended Mitigation Steps
Add the following check in `setBackupConfig`

```
require(erc20s.length <= max, "Invalid backup config");
```