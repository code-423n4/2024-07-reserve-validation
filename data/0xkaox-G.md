https://github.com/reserve-protocol/protocol/blob/c8752bcdc5acc291941149881199e18ffadde5aa/contracts/libraries/Array.sol#L7-L16

This function might accomplish same ressult O(n) instead of O(n²)

    function allUnique(IERC20[] memory arr) internal pure returns (bool) {
        mapping(address => bool) memory seen;
        
        for (uint i = 0; i < arr.length; i++) {
            address tokenAddress = address(arr[i]);
            if (seen[tokenAddress]) {
                // If has already been seen, not unique
                return false;
            }
            
            seen[tokenAddress] = true;
        }
        
        return true;
    }
