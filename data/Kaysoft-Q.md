## Wrong Number of reserved storage gap in Furnace.sol can cause storage collision

By convention an upgradable contract reserves storage gap based on total number of 50 storage variables. 
The Furnace.sol has 4 storage variable gaps so the reserved storage gap is supposed to by `46` instead of the current `47`. This can cause storage collision when new variable is added during upgrade.

```solidity
File: Furnace.sol
contract FurnaceP1 is ComponentP1, IFurnace {
    using FixLib for uint192;

    uint192 public constant MAX_RATIO = 1e14; // {1} 0.01%

    IRToken private rToken;

    // === Governance params ===
    uint192 public ratio; // {1} What fraction of balance to melt each period

    // === Cached ===
    uint48 public lastPayout; // {seconds} The last time we did a payout
    uint256 public lastPayoutBal; 

    ...
    uint256[47] private __gap;
}
```
The `rToken`, `ratio`, `lastPayout` and `lastPayoutBal` state variables take up 4 storage slots so `50 - 4 = 46`.

Recommendation: Consider changing the `47` and `46` this way:
```diff
contract FurnaceP1 is ComponentP1, IFurnace {
    using FixLib for uint192;

    uint192 public constant MAX_RATIO = 1e14; // {1} 0.01%

    IRToken private rToken;

    // === Governance params ===
    uint192 public ratio; // {1} What fraction of balance to melt each period

    // === Cached ===
    uint48 public lastPayout; // {seconds} The last time we did a payout
    uint256 public lastPayoutBal; 

    ...
--  uint256[47] private __gap;
++  uint256[46] private __gap;
}
```