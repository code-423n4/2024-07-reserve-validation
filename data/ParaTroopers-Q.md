
| Serial No | Title                                  | Contract/Code                 |
|-----------|----------------------------------------|-------------------------------|
| QA-1      | missing `__ReentrancyGuard_init`       | RevenueTraderP1::init          |


**[QA-1]** missing `__ReentrancyGuard_init` implementation in `RevenueTraderP1::init`

Status storage wont be updated from 0 to 1 if [__ReentrancyGuard_init](https://github.com/moonwell-fi/contracts-open-source/blob/c7da88a3fe3f0062d8a83ba808b648f1da369fec/contracts/safety-module/OpenZeppelin/ReentrancyGuardUpgradeable.sol#L32-L64) is not called internally

[__ReentrancyGuard_init](https://github.com/moonwell-fi/contracts-open-source/blob/c7da88a3fe3f0062d8a83ba808b648f1da369fec/contracts/safety-module/OpenZeppelin/ReentrancyGuardUpgradeable.sol#L32-L64)
```solidity
    uint256 private constant _NOT_ENTERED = 1;
    uint256 private constant _ENTERED = 2;

    uint256 private _status;

    function __ReentrancyGuard_init() internal onlyInitializing {
        __ReentrancyGuard_init_unchained();
    }

    function __ReentrancyGuard_init_unchained() internal onlyInitializing {
        _status = _NOT_ENTERED;
    }
```
[RevenueTraderP1::init](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/RevenueTrader.sol#L27-L38)
```diff
    function init(
        IMain main_,
        IERC20 tokenToBuy_,
        uint192 maxTradeSlippage_,
        uint192 minTradeVolume_
    ) external initializer {
        require(address(tokenToBuy_) != address(0), "invalid token address");
        __Component_init(main_);
+       __ReentrancyGuard_init();
        __Trading_init(main_, maxTradeSlippage_, minTradeVolume_);
        tokenToBuy = tokenToBuy_;
        cacheComponents();
    }
```