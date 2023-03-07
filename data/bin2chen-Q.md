L-1:
CollateralConfig.updateCollateralRatios() will modify the MCR and CCR, these two parameters will affect whether the user is liquidated, related to the user's collateral.
Although the administrator is a multi-signature wallet, but still need to have an effective time, to give the user sufficient time to choose whether to keep trove or repayment


L-2:
increaseLUSDDebt()/decreaseLUSDDebt()
Event missing emit
```solidity
    function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
-       ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
+       emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
```

L-3:
TroveManager._removeTroveOwner()
Forgot settings `Troves[_borrower][_collateral].arrayIndex = 0` when clearing Trove Owner

    function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
        Status troveStatus = Troves[_borrower][_collateral].status;
        // It’s set in caller function `_closeTrove`
        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

        uint128 index = Troves[_borrower][_collateral].arrayIndex;
        uint length = TroveOwnersArrayLength;
        uint idxLast = length.sub(1);

        assert(index <= idxLast);

        address addressToMove = TroveOwners[_collateral][idxLast];

        TroveOwners[_collateral][index] = addressToMove;
        Troves[addressToMove][_collateral].arrayIndex = index;
        emit TroveIndexUpdated(addressToMove, _collateral, index);

+       Troves[_borrower][_collateral].arrayIndex = 0;
        TroveOwners[_collateral].pop();
    }
