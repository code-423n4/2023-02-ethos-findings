### Unbounded Loop in _getTotalsFromLiquidateTrovesSequence_RecoveryMode and _getTotalsFromLiquidateTrovesSequence_NormalModecan lead to DOS.

If there is a lot of troves to be liquidated and n in the function ```liquidateTroves()``` parameters is large, the transaction’s gas cost could exceed the block gas limit and make it impossible to call this function at all.
```
function _getTotalsFromLiquidateTrovesSequence_RecoveryMode
    (
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        ISortedTroves _sortedTroves,
        address _collateral,
        uint _price,
        uint _LUSDInStabPool,
        uint _n
    )
        internal
        returns(LiquidationTotals memory totals)
    {
        LocalVariables_LiquidationSequence memory vars;
        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);
        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
        LiquidationValues memory singleLiquidation;

        vars.remainingLUSDInStabPool = _LUSDInStabPool;
        vars.backToNormalMode = false;
        vars.entireSystemDebt = getEntireSystemDebt(_collateral);
        vars.entireSystemColl = getEntireSystemColl(_collateral);

        vars.user = _sortedTroves.getLast(_collateral);
        address firstUser = _sortedTroves.getFirst(_collateral);
        for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
            // we need to cache it, because current user is likely going to be deleted
            address nextUser = _sortedTroves.getPrev(_collateral, vars.user);

            vars.ICR = getCurrentICR(vars.user, _collateral, _price);

            if (!vars.backToNormalMode) {
                // Break the loop if ICR is greater than MCR and Stability Pool is empty
                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

                vars.TCR = LiquityMath._computeCR(vars.entireSystemColl, vars.entireSystemDebt, _price, vars.collDecimals);

                singleLiquidation = _liquidateRecoveryMode(
                    _activePool,
                    _defaultPool,
                    _collateral,
                    vars.user,
                    vars.ICR,
                    vars.remainingLUSDInStabPool,
                    vars.TCR,
                    _price,
                    vars.collMCR
                );

                // Update aggregate trackers
                vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);
                vars.entireSystemDebt = vars.entireSystemDebt.sub(singleLiquidation.debtToOffset);
                vars.entireSystemColl = vars.entireSystemColl.
                    sub(singleLiquidation.collToSendToSP).
                    sub(singleLiquidation.collGasCompensation).
                    sub(singleLiquidation.collSurplus);

                // Add liquidation values to their respective running totals
                totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

                vars.backToNormalMode = !_checkPotentialRecoveryMode(
                    vars.entireSystemColl,
                    vars.entireSystemDebt,
                    _price,
                    vars.collDecimals,
                    vars.collCCR
                );
            }
            else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {
                singleLiquidation = _liquidateNormalMode(
                    _activePool,
                    _defaultPool,
                    _collateral,
                    vars.user,
                    vars.remainingLUSDInStabPool
                );

                vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

                // Add liquidation values to their respective running totals
                totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

            }  else break;  // break if the loop reaches a Trove with ICR >= MCR

            vars.user = nextUser;
        }
    }

function _getTotalsFromLiquidateTrovesSequence_NormalMode
    (
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        uint _price,
        uint256 _MCR,
        uint _LUSDInStabPool,
        uint _n
    )
        internal
        returns(LiquidationTotals memory totals)
    {
        LocalVariables_LiquidationSequence memory vars;
        LiquidationValues memory singleLiquidation;
        ISortedTroves sortedTrovesCached = sortedTroves;

        vars.remainingLUSDInStabPool = _LUSDInStabPool;

        for (vars.i = 0; vars.i < _n; vars.i++) {
            vars.user = sortedTrovesCached.getLast(_collateral);
            vars.ICR = getCurrentICR(vars.user, _collateral, _price);

            if (vars.ICR < _MCR) {
                singleLiquidation = _liquidateNormalMode(
                    _activePool,
                    _defaultPool,
                    _collateral,
                    vars.user,
                    vars.remainingLUSDInStabPool
                );

                vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

                // Add liquidation values to their respective running totals
                totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

            } else break;  // break if the loop reaches a Trove with ICR >= MCR
        }
    }
```