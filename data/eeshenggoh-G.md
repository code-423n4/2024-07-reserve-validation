In `PermitLib::requireSignature`, the function checks if owner address is either EOA or contract address. If it not an EOA, it calls `SignatureCheckerUpgradeable.isValidSignatureNow`. Within that function it calls `isValidERC1271SignatureNow(signer, hash, signature)` again to check if it's a contract address. This should be removed:
```
    function isValidSignatureNow(address signer, bytes32 hash, bytes memory signature) internal view returns (bool) {
        (address recovered, ECDSAUpgradeable.RecoverError error) = ECDSAUpgradeable.tryRecover(hash, signature);
        // checks correctly for eoa signatures recover == signer@audit-ok
        return
            (error == ECDSAUpgradeable.RecoverError.NoError && recovered == signer)
    }

```