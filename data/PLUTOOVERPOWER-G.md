1)

    function pushDraft(address account, uint256 rsrAmount) 
        internal
        returns (uint256 index, uint64 availableAt)
    {
        // draftAmount: how many drafts to create and assign to the user
        // pick draftAmount as big as we can such that (newTotalDrafts <= newDraftRSR * draftRate)
        draftRSR += rsrAmount;
        // newTotalDrafts: {qDrafts} = D18{qDrafts/qRSR} * {qRSR} / D18
        uint256 newTotalDrafts = (draftRate * draftRSR) / FIX_ONE;
        uint256 draftAmount = newTotalDrafts - totalDrafts;
        totalDrafts = newTotalDrafts;

        // Push drafts into account's draft queue
        CumulativeDraft[] storage queue = draftQueues[draftEra][account];
        index = queue.length;

        uint192 oldDrafts = index != 0 ? queue[index - 1].drafts : 0; 
        uint64 lastAvailableAt = index != 0 ? queue[index - 1].availableAt : 0;
        availableAt = uint64(block.timestamp) + unstakingDelay;
        if (lastAvailableAt > availableAt) {
            availableAt = lastAvailableAt; 
        }

        queue.push(CumulativeDraft(uint176(oldDrafts + draftAmount), availableAt));
    }
'''


    function pushDraft(address account, uint256 rsrAmount)

        internal
        returns (uint256 index, uint64 availableAt)
    {
        // draftAmount: how many drafts to create and assign to the user
        // pick draftAmount as big as we can such that (newTotalDrafts <= newDraftRSR * draftRate)
        draftRSR += rsrAmount;
        // newTotalDrafts: {qDrafts} = D18{qDrafts/qRSR} * {qRSR} / D18
        uint256 newTotalDrafts = (draftRate * draftRSR) / FIX_ONE;
        uint256 draftAmount = newTotalDrafts - totalDrafts;
        totalDrafts = newTotalDrafts;


        // Push drafts into account's draft queue
        uint192 oldDrafts = index != 0 ? queue.drafts : 0;
        uint64 lastAvailableAt = index != 0 ? queue.availableAt : 0;


        //-------------------------------------=----------------
        //ISSUE: less gas consumed because of the opcode. it is much more cheeper for a contract
        //to get some contracts before having queue.length gains plus. 
        //------------------------------------------------------


        CumulativeDraft[] storage queue = draftQueues[draftEra][account];
        index = queue.length;

        availableAt = uint64(block.timestamp) + unstakingDelay;
        if (lastAvailableAt > availableAt) {
            availableAt = lastAvailableAt; 
        }

        queue.push(CumulativeDraft(uint176(oldDrafts + draftAmount), availableAt));

    }
'''
2)

    function withdraw(address account, uint256 endId) external {
        _requireNotTradingPausedOrFrozen();

        uint256 firstId = firstRemainingDraft[draftEra][account];
        CumulativeDraft[] storage queue = draftQueues[draftEra][account];
        if (endId == 0 || firstId >= endId) return;

        // == Checks + Effects ==
        require(endId <= queue.length, "index out-of-bounds");
        require(queue[endId - 1].availableAt <= block.timestamp, "withdrawal unavailable");

        // untestable:
        //      firstId will never be zero, due to previous checks against endId
        uint192 oldDrafts = firstId != 0 ? queue[firstId - 1].drafts : 0;
        uint192 draftAmount = queue[endId - 1].drafts - oldDrafts;

        // advance queue past withdrawal
        firstRemainingDraft[draftEra][account] = endId;

        // ==== Compute RSR amount
        uint256 newTotalDrafts = totalDrafts - draftAmount;
        // newDraftRSR: {qRSR} = {qDrafts} * D18 / D18{qDrafts/qRSR}
        uint256 newDraftRSR = (newTotalDrafts * FIX_ONE_256 + (draftRate - 1)) / draftRate;
        uint256 rsrAmount = draftRSR - newDraftRSR;

        if (rsrAmount == 0) return;

        // ==== Transfer RSR from the draft pool
        totalDrafts = newTotalDrafts;
        draftRSR = newDraftRSR;

        // == Interactions ==
        leakyRefresh(rsrAmount);
        IERC20Upgradeable(address(rsr)).safeTransfer(account, rsrAmount);
        emit UnstakingCompleted(firstId, endId, draftEra, account, rsrAmount);

        // == Checks ==
        require(basketHandler.isReady() && basketHandler.fullyCollateralized(), "RToken readying");
    }

    function withdraw(address account, uint256 endId) external {
        _requireNotTradingPausedOrFrozen();


        // == Checks ==
        require(basketHandler.isReady() && basketHandler.fullyCollateralized(), "RToken readying");
//---------------------------------------
check the require code first. if this reverts, the users should pay allthe gas fee all the way through until it reaches this require codes.
//---------------------------------------

        uint256 firstId = firstRemainingDraft[draftEra][account];
        CumulativeDraft[] storage queue = draftQueues[draftEra][account];
        if (endId == 0 || firstId >= endId) return;

        // == Checks + Effects ==
        require(endId <= queue.length, "index out-of-bounds");
        require(queue[endId - 1].availableAt <= block.timestamp, "withdrawal unavailable");

        // untestable:
        //      firstId will never be zero, due to previous checks against endId
        uint192 oldDrafts = firstId != 0 ? queue[firstId - 1].drafts : 0;
        uint192 draftAmount = queue[endId - 1].drafts - oldDrafts;

        // advance queue past withdrawal
        firstRemainingDraft[draftEra][account] = endId;

        // ==== Compute RSR amount
        uint256 newTotalDrafts = totalDrafts - draftAmount;
        // newDraftRSR: {qRSR} = {qDrafts} * D18 / D18{qDrafts/qRSR}
        uint256 newDraftRSR = (newTotalDrafts * FIX_ONE_256 + (draftRate - 1)) / draftRate;
        uint256 rsrAmount = draftRSR - newDraftRSR;

        if (rsrAmount == 0) return;

        // ==== Transfer RSR from the draft pool
        totalDrafts = newTotalDrafts;
        draftRSR = newDraftRSR;

        // == Interactions ==
        leakyRefresh(rsrAmount);
        IERC20Upgradeable(address(rsr)).safeTransfer(account, rsrAmount);
        emit UnstakingCompleted(firstId, endId, draftEra, account, rsrAmount);

    
    }



