https://github.com/code-423n4/2024-07-reserve/blob/main/contracts/mixins/Auth.sol#L198-L209

    function setShortFreeze(uint48 shortFreeze_) public onlyRole(OWNER) {
        require(shortFreeze_ != 0 && shortFreeze_ <= MAX_SHORT_FREEZE, "short freeze out of range");
        emit ShortFreezeDurationSet(shortFreeze, shortFreeze_);
        shortFreeze = shortFreeze_;
    }

    /// @custom:governance
    function setLongFreeze(uint48 longFreeze_) public onlyRole(OWNER) {
        require(longFreeze_ != 0 && longFreeze_ <= MAX_LONG_FREEZE, "long freeze out of range");
        emit LongFreezeDurationSet(longFreeze, longFreeze_);
        longFreeze = longFreeze_;
    }

There should be a check, longFreeze > shortFreeze . 