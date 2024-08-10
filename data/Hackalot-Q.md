# QA Report

## QA1
File: [Allowance.sol#L30](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Allowance.sol#L30)

The condition should be placed at the very top of the function to avoid unnecessary operations if the value is zero. This check should precede any external calls or operations and be executed first for efficiency. 

## QA2: Potential Underflow in abs() Function

File: [Fixed.sol#L148](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Fixed.sol#L148)
The abs function calculates the absolute value of an integer. While Solidity 0.8+ includes checked arithmetic by default, it's essential to explicitly state that this is intentional to prevent underflows.
Suggestion: Ensure the underflow is avoided or explicitly handle it.

```
function abs(int256 x) pure returns (uint256) {
    require(x != type(int256).min, "Int256 underflow");
    return x < 0 ? uint256(-x) : uint256(x);
}
```

## QA3 Additional Error Handling for ERC1271

File: [Permit.sol#L19](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Permit.sol#L19)

Issue: When verifying signatures via the ERC1271 interface, the current implementation checks for a specific return value (0x1626ba7e). However, it's important to handle cases where the contract might revert or return unexpected values. A failure to do so could result in unexpected behavior.

Suggestion: Add error handling to capture reverts and unexpected return values from the isValidSignature function.

```
try IERC1271Upgradeable(owner).isValidSignature(hash, abi.encodePacked(r, s, v)) returns (bytes4 result) {
    require(result == 0x1626ba7e, "ERC1271: Unauthorized");
} catch {
    revert("ERC1271: Unauthorized");
}
```

## QA4 Type and Range Safety

File: [String.sol#L16](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/String.sol#L16)

Current State: The function checks if the ASCII value is between 65 and 90 using uint8(bStr[i]).

Optimization: Solidity allows direct comparison of bytes1 values using hexadecimal constants, which is more intuitive and avoids unnecessary type conversions.

```
if (bStr[i] >= 0x41 && bStr[i] <= 0x5A) {
    bStr[i] = bytes1(uint8(bStr[i]) + 32);
}
```
This approach reduces the need for converting bytes1 to uint8, slightly improving gas efficiency.


## QA5 Potential improvement for hourlyLimit function

File: [Throttle.sol#L80](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/libraries/Throttle.sol#L88)

Optimization: Consider adding an if check to avoid unnecessary calculations if pctRate is zero
```
if (params.pctRate > 0) {
    ...
}
```

## QA6 Code Clarity in freezeLong function

File: [Auth.sol#L139](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/mixins/Auth.sol#L139)

Issue: The function freezeLong decreases the longFreezes counter without checking if the current value is greater than zero before decrementing. Although Solidity 0.8.x has built-in underflow checks, it's better to put this explicitly in the code.

Suggestion: Add a check to ensure that longFreezes[_msgSender()] is greater than zero before decrementing to avoid a revert that might not be immediately obvious to someone reading the 

```
require(longFreezes[_msgSender()] > 0, "no long freeze charges left");
longFreezes[_msgSender()] -= 1;
```

## QA7 Unchecked Low-Level Call in rebalance function in BackingManager.sol

File: [BackingManager.sol:82](https://github.com/code-423n4/2024-07-reserve/blob/154db90db4f870d147a149e62ffc5e4e6380af00/contracts/p0/BackingManager.sol#L82)

Issue: In the rebalance function, there is a low-level call to ```main.stRSR().seizeRSR(req.sellAmount - bal);``` if the balance is insufficient. If this call fails, there is no handling for potential issues like insufficient gas or reentrancy attacks.

Impact: If the call to seize RSR fails due to unexpected reasons (e.g., a reentrancy attack or a gas limit issue), it could leave the contract in an undercollateralized state or prevent the system from functioning as intended.

Mitigation: Consider implementing error handling around this call to log failures or revert the transaction with a specific reason.

## QA8: Potential risks with External Calls in BackingManager.sol

File: [BackingManager.sol](https://github.com/code-423n4/2024-07-reserve/blob/154db90db4f870d147a149e62ffc5e4e6380af00/contracts/p0/BackingManager.sol)

Issue: The contract interacts with external contracts like ```main.stRSR()``` and ```main.rToken()``` in functions like ```rebalance```, ```forwardRevenue```, and ```settleTrade```. If these external contracts are untrusted, there is a potential reentrancy risk.

Impact: If one of these external calls triggers a reentrant call back into the BackingManager, it could manipulate the state in unintended ways, such as altering the balances or causing double spending.

Mitigation: Use the nonReentrant modifier on critical functions that interact with external contracts, especially those that modify state and consider minimizing external calls after state changes. 

## QA9: Potential for Stale Data in forwardRevenue in BackingManager.sol

File: [BackingManager.sol:157](https://github.com/code-423n4/2024-07-reserve/blob/154db90db4f870d147a149e62ffc5e4e6380af00/contracts/p0/BackingManager.sol#L157)

Issue: The forwardRevenue function relies on a snapshot of basket quantities and balances. If the state of the basket changes after this snapshot but before the revenue is forwarded, the system may operate on stale data.

Impact: This could result in incorrect revenue distribution, potentially leading to over- or under-distribution of tokens.

Mitigation: Refresh the state immediately before performing calculations that depend on basket quantities and balances or minimize the time before snapshot is taken and used. (eg. refresh ```basketsHeld``` right before usage a few lines down)
