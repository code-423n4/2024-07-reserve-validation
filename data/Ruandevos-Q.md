solc frequently releases new compiler versions. Using an old version prevents access to new Solidity security checks. We also recommend avoiding complex pragma statement.
Location:

Pragma version0.8.19 (contracts/libraries/Allowance.sol#2) necessitates a version too recent to be trusted. Consider deploying with 0.8.18.

// SPDX-License-Identifier: BlueOak-1.0.0
pragma solidity 0.8.19;

interface IERC20ApproveOnly {
    function approve(address spender, uint256 value) external;

    function allowance(address owner, address spender) external view returns (uint256);
}
