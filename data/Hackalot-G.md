File link: https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Array.sol#L9

Array.sol:9

If the array being verified for allUnique becomes very large, it might be more efficient to reduce time complexity to O(n) by using a map 
```
    /// O(n) on average
    /// @return true if the array contains all unique addresses
    function allUnique(address[] memory arr) internal pure returns (bool) {
        mapping(address => bool) seen;
        uint256 arrLen = arr.length;
        for (uint256 i = 0; i < arrLen; ++i) {
            if (seen[arr[i]]) {
                return false;
            }
            seen[arr[i]] = true;
        }
        return true;
    }
```