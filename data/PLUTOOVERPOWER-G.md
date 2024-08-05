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