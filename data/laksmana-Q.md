When using ``shiftl_toFix``, to round with ``FLOOR``
should only use shiftl_toFix(blah, blah);
Because using ``shiftl_toFix`` with two parameters. Already triggers ``shiftl_toFix`` with ``FLOOR`` rounding
```
function shiftl_toFix(uint256 x, int8 shiftLeft) pure returns (uint192) {
    return shiftl_toFix(x, shiftLeft, FLOOR);
}
```
Example:
https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/plugins/assets/Asset.sol#L180

