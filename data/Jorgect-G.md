# LOW REPORT

## wrong check in `setUnstakingDelay` in the `StRSR.sol`

The `setUnstakingDelay` have some limit to respects `MIN_UNSTAKING_DELAY` and 
`MAX_UNSTAKING_DELAY`

```solidity


    function setUnstakingDelay(uint48 val) public {
        _requireGovernanceOnly();
        require(val > MIN_UNSTAKING_DELAY && val <= MAX_UNSTAKING_DELAY,  "invalid unstakingDelay"); <------
        emit UnstakingDelaySet(unstakingDelay, val);
        unstakingDelay = val; //@audit-issue see for front run oportunitties before and after this call
    }

```
[[Link](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/StRSR.sol#L970)

As you can see the arrow below the min value is not been permited.

### Impact
The `MIN_UNSTAKING_DELAY` can not be set up because an incorrect check


### Recommendation

Consider change `>` to `>=`:

```solidity


    function setUnstakingDelay(uint48 val) public {
        _requireGovernanceOnly();
        require(val >= MIN_UNSTAKING_DELAY && val <= MAX_UNSTAKING_DELAY,  "invalid unstakingDelay"); <------
        emit UnstakingDelaySet(unstakingDelay, val);
        unstakingDelay = val; //@audit-issue see for front run oportunitties before and after this call
    }

```

## [L-02]  not FROZEN IN `stake` can be a problem `StRSR.sol`

`Stake` function does not have any frozen or paused functionality as `unstake` do. this is intended design but it could be a problem too because if the project is in a frozen or paused state and a person or a smart contract build on top  `stake` that money could be loss for ever.

### Impact
User can loss his money is he stake in the project in a frozen state. normals users will use the front end so probably they will be safe, But smart contract build on top of The protocol could be in a lot danger.

### Proof of concept
See That `stake` does not have a frozen functionality:

```Solidity

  function stake(uint256 rsrAmount) public {
        _notZero(rsrAmount);

        _payoutRewards();

        // Mint new stakes
        address caller = _msgSender();
        mintStakes(caller, rsrAmount);

        // == Interactions ==
        IERC20Upgradeable(address(rsr)).safeTransferFrom(caller, address(this), rsrAmount);
    }

```
[[Link]](https://github.com/code-423n4/2024-07-reserve/blob/3f133997e186465f4904553b0f8e86ecb7bbacbf/contracts/p1/StRSR.sol#L227)

### Recommendation 

Consider add a frozen functionality to the stake functions, this way users or smart contract does not stake accidentally.



