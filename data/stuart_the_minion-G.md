## [G-01] Use `=` operator instead of `+=`

Links to affected Code:

https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/StRSR.sol#L733

Update:

```diff
-   stakeRSR += rsrAmount;
+   stakeRSR = newStakeRSR;
```
