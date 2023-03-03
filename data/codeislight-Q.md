- internal functions used by a function that doesn't sharing any common logic with any other.

instances in :
LUSDToken._requireCallerIsStabilityPool()
LUSDToken._requireCallerIsTroveMorSP()
LUSDToken._requireMintingNotPaused()
ActivePool._requireCallerIsBorrowerOperationsOrDefaultPool()
BorrowerOperations._getCollChange()
BorrowerOperations._updateTroveFromAdjustment()
BorrowerOperations._moveTokensAndCollateralfromAdjustment()
BorrowerOperations._requireValidCollateralAddress()
... too many to mention :)

- in StabilityPool, _triggerLQTYIssuance and _updateG can be merged in one, since one only calls a variable and pass it to the other.

- unnecessary return in TroveManager.updateTroveRewardSnapshots(), since _updateTroveRewardSnapshots() function doesn't return any value. and _applyPendingRewards doesn't use the returned value.

- TroveManager.getTroveStatus() better return either uint8 or enum Status, instead of rounding it up to uint256, to maintain a consistency.

- LiquityBase._checkRecoveryMode() , it is advised that the recovery mode is inclusive for CCR value:

```solidity
    function _checkRecoveryMode(
        address _collateral,
        uint256 _price,
        uint256 _CCR,
        uint256 _collateralDecimals
    ) internal view returns (bool) {
        uint256 TCR = _getTCR(_collateral, _price, _collateralDecimals);
        // tbd QA got to be inclusive for CCR
        return TCR <= _CCR;
    }
```

- ReaperVaultV2 lacks checking for addresses being contracts in the constructor, lacks checking token, treasury if it's multisig which is advised to be.

- explicitly define ReaperVaultV2 constructor _multisigRoles array variable parameter to a size of 3.

- ReaperVaultV2.updateTreasury() lacks checking for parameter being a contract.