# Gas Optimizations Report

## Gas Optimization 1

File: [Array.sol#L9](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Array.sol#L9)

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

## Gas Optimization 2

File: [Permit.sol#L19](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Permit.sol#L19)

Observation: The abi.encodePacked function is used twice in the function, which involves a certain amount of overhead.

Optimization: Consider storing the encoded value in a memory variable if this reduces the overall gas usage (depending on the compiler's optimization).

```
bytes memory signature = abi.encodePacked(r, s, v);
if (AddressUpgradeable.isContract(owner)) {
    require(
        IERC1271Upgradeable(owner).isValidSignature(hash, signature) == 0x1626ba7e,
        "ERC1271: Unauthorized"
    );
} else {
    require(
        SignatureCheckerUpgradeable.isValidSignatureNow(owner, hash, signature),
        "ERC20Permit: invalid signature"
    );
}
```