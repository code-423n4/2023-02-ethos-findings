

1- in ActivePool.so
 function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
    missing the keyword emit in the last line.

    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
    missing the keyword emit in the last line.


2- It is strongly recommended to use the latest stable version of solc and to use the same version among all 
   contracts in the protocol.
3- Many helper functions with only one-line of code could be replaced with its inner code in-place to minimize 
   code size and enhance readability.
4- In StabilityPool.sol struct SnapShot has a member called "p", at the same time there is a state variable named 
   "p" too. Try to use different & more meaningful names to help increase readability and maintainability.
 