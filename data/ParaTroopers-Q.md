
| Serial No | Title                                  | Contract/Code                 |
|-----------|----------------------------------------|-----------------------------
| Low-1     | No need to check if owner is contract | PermitLib::requireSignature 
| QA-1      | missing `__ReentrancyGuard_init`       | RevenueTraderP1::init          |

# Low-1
The requireSignature makes a decision based on whether the owner is EOA or a smart contract, this is not needed because openzeppelin's `isValidSignatureNow` by default calls `isValidERC1271SignatureNow` which means it checks for both EOA and smart contracts

```solidity
function requireSignature(
        address owner,
        bytes32 hash,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) internal view {
        if (AddressUpgradeable.isContract(owner)) {
            require(
                IERC1271Upgradeable(owner).isValidSignature(hash, abi.encodePacked(r, s, v)) ==
                    0x1626ba7e,
                "ERC1271: Unauthorized"
            );
        } else {
            require(
                SignatureCheckerUpgradeable.isValidSignatureNow(
                    owner,
                    hash,
                    abi.encodePacked(r, s, v)
                ),
                "ERC20Permit: invalid signature"
            );
        }
    }
```

```solidity
function isValidSignatureNow(address signer, bytes32 hash, bytes memory signature) internal view returns (bool) {
        (address recovered, ECDSAUpgradeable.RecoverError error) = ECDSAUpgradeable.tryRecover(hash, signature);
        return
            (error == ECDSAUpgradeable.RecoverError.NoError && recovered == signer) ||
            isValidERC1271SignatureNow(signer, hash, signature);
    }
```
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