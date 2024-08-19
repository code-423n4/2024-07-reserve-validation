## Summary:

The `seizeRSR` function reverts when the contract’s RSR balance is zero, leading to potential disruptions in operation if the contract attempts to seize RSR under this condition.

## Description:

The `seizeRSR` function includes a check to ensure that the amount of RSR to be seized (rsrAmount) is less than or equal to the current RSR balance (rsrBalance). However, it does not preemptively handle the scenario where the RSR balance is zero. As a result, if the RSR balance is zero, the function will revert with an error message, potentially leading to frequent reverts in environments where the RSR balance is often depleted.

## Impact:

If the RSR balance is zero, any attempt to call the `seizeRSR` function will fail, leading to operational disruptions. This could be particularly problematic in systems where RSR needs to be seized frequently or in systems that automatically trigger such functions based on certain events.

**Root Cause**:

The function lacks an initial check for a zero RSR balance before proceeding to the require statement that compares `rsrAmount` with `rsrBalance`. Consequently, the function does not account for the edge case where `rsrBalance` is zero, leading to an unintended reversion.

## Recommended Mitigation:

To mitigate this issue, introduce an additional check at the beginning of the `seizeRSR` function to explicitly handle the case where `rsrBalance` is zero. This check can gracefully exit the function or trigger an appropriate error message, avoiding unnecessary reverts and improving the function's robustness.