In `DeployerP1::deploy`, the `StringLib::toLower` is called to convert strings to lowercase.
According to the [docs gist](https://gist.github.com/ottodevs/c43d0a8b4b891ac2da675f825b1d1dbf) 

This gives unexpected behavior when the string that was passed in _toLowerCase is referenced later in the code, because _toLowerCase doesn't only transform the output, but also modifies the original string and its bytes in the memory.

This means that the input `symbol`, will not be the same as user input but lower cased.
```solidity
    function deploy(
        string memory name,
        string memory symbol,
        string calldata mandate,
        address owner,
        DeploymentParams memory params
    ) external returns (address) {
    -- SNIP --
        // Init StRSR
        {
            string memory stRSRSymbol = string(abi.encodePacked(StringLib.toLower(symbol), "RSR"));
            string memory stRSRName = string(abi.encodePacked(stRSRSymbol, " Token"));
            main.stRSR().init(
                main,
                stRSRName,
                stRSRSymbol,
                params.unstakingDelay,
                params.rewardRatio,
                params.withdrawalLeak
            );
        }

        // Init RToken
        components.rToken.init(
            main,
            name,
            symbol, //original string is modified here too
            mandate,
            params.issuanceThrottle,
            params.redemptionThrottle
        );
    -- SNIP --
```
To fix the issue:
So here's a fixed "transform string to lower case" version for Solidity above ^0.5.0 (that won't modify the original string in the memory):

```solidity
function copyBytes(bytes memory _bytes) private pure returns (bytes memory) {
        bytes memory copy = new bytes(_bytes.length);
        uint256 max = _bytes.length + 31;
        for (uint256 i=32; i<=max; i+=32)
        {
            assembly { mstore(add(copy, i), mload(add(_bytes, i))) }
        }
        return copy;
    }

    function _toLowercase(string memory inputNonModifiable) public pure returns (string memory) {
        bytes memory bytesInput = copyBytes(bytes(inputNonModifiable));

        for (uint i = 0; i < bytesInput.length; i++) {
            // checks for valid ascii characters // will allow unicode after building a string library
            require (uint8(bytesInput[i]) > 31 && uint8(bytesInput[i]) < 127, "Only ASCII characters");
            // Uppercase character...
            if (uint8(bytesInput[i]) > 64 && uint8(bytesInput[i]) < 91) {
                // add 32 to make it lowercase
                bytesInput[i] = bytes1(uint8(bytesInput[i]) + 32);
            }
        }
        return string(bytesInput);
    }
```