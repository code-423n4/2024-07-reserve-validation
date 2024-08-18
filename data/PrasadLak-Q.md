## Approval race condition 

## Lines of code
https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/StRSR.sol#L781C4-L784C6



## Vulnerability details
The race condition occurs when changing the approval amount. There is a well-known attack vector described [here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.b32yfk54vyg9).

## Proof of Concept

```solidity
 function approve(address spender, uint256 amount) public returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

   function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal {
        _notZero(owner);
        _notZero(spender);

        _allowances[era][owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
```



## Impact
Attacker able to get more tokens when changing the approval amount by the user if user use `approve` function to change the approval amount.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Instead of `approve` function use [increaseAllowance](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/StRSR.sol#L800C14-L800C31) and [decreaseAllowance](https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/p1/StRSR.sol#L806) function for changing the approve  token amounts.
