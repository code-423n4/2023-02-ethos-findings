
----------

| S No. | Issue | Instances | Gas Savings (from provided tests) |
|-----|-----|-----|-----|
| [G-01] | x += y costs more gas than x = x + y for state variables (same applies for x -= y) | 22 | 2486 
| [G-02] | Internal functions only called once can be inlined to save gas | 46 | 1840
| [G-03] | Using both named returns and a return statement isn't necessary | 18 | 54
| [G-04] | Increments can be unchecked | 15 | 600
| [G-05] | Require() or revert() strings longer than 32 bytes cost extra gas | 49 | 147
| [G-06] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 8 | - 
| [G-07] | Splitting require() statements that use && saves gas | 4 | 12
| [G-08] | Use scientific notation rather than exponentiation | 1 | -
| [G-09] | Use a more recent version of solidity | 12 | -

-------------

## G-01 x += y costs more gas than x = x + y for state variables (same applies for x -= y)

Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**.

_There are 22 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

133: toFree += profit;
141: roi -= int256(loss);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

168: totalAllocBPS += _allocBPS;
194: totalAllocBPS -= strategies[_strategy].allocBPS;
196: totalAllocBPS += _allocBPS;
214: totalAllocBPS -= strategies[_strategy].allocBPS;
390: value -= loss;
391: totalLoss += loss;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
450: stratParams.losses += loss;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;
505: strategy.gains += vars.gain;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment;
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

128: repayment -= uint256(-roi);
```

----------

## G-02 Internal functions only called once can be inlined to save gas

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 46 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
File: Ethos-Core/contracts/ActivePool.sol

320: function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
337: function _requireCallerIsBOorTroveM() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
File: Ethos-Core/contracts/BorrowerOperations.sol

438: function _getCollChange(
455: function _updateTroveFromAdjustment
476: function _moveTokensAndCollateralfromAdjustment
533: function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
537: function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {
546: function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
551: function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
555: function _requireNotInRecoveryMode(
567: function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {
571: function _requireValidAdjustmentInCurrentMode
624: function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {
636: function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {
661: function _getNewNominalICRFromTroveChange
682: function _getNewICRFromTroveChange
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
File: Ethos-Core/contracts/LUSDToken.sol

380: function _requireCallerIsTroveMorSP() internal view {
393: function _requireMintingNotPaused() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

421: function _updateG(uint _LQTYIssuance) internal {
439: function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {
776: function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {
794: function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {
848: function _requireCallerIsActivePool() internal view {
852: function _requireCallerIsTroveManager() internal view{
856: function _requireNoUnderCollateralizedTroves() internal {
869: function _requireUserHasDeposit(uint _initialDeposit) internal pure {
873: function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

478: function _getCappedOffsetVals
582: function _getTotalsFromLiquidateTrovesSequence_RecoveryMode
671: function _getTotalsFromLiquidateTrovesSequence_NormalMode
785: function _getTotalFromBatchLiquidate_RecoveryMode
864: function _getTotalsFromBatchLiquidate_NormalMode
1082: function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
1213: function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
1321: function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1339: function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
1516: function _minutesPassedSinceLastFeeOp() internal view returns (uint) {
1538: function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

251: function _requireCallerIsTroveManagerOrActivePool() internal view {
265: function _requireUserHasStake(uint currentStake) internal pure {
269: function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

86: function _liquidatePosition(uint256 _amountNeeded)
206: function _claimRewards() internal {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

462: function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

240: function _liquidatePosition(uint256 _amountNeeded)
250: function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);
```

-----

## G-03 Using both named returns and a return statement isn't necessary 

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.

Each MLOAD and MSTORE costs **3 additional gas**

_There are 18 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

504: function _computeRewardsPerUnitStaked(
626: function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

321: function _liquidateNormalMode(
357: function _liquidateRecoveryMode(
898: function _addLiquidationValuesToTotals(LiquidationTotals memory oldTotals, LiquidationValues memory singleLiquidation)
1316: function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
1321: function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

29: function asset() external view override returns (address assetTokenAddress) {
37: function totalAssets() external view override returns (uint256 totalManagedAssets) {
51: function convertToShares(uint256 assets) public view override returns (uint256 shares) {
66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
79: function maxDeposit(address) external view override returns (uint256 maxAssets) {
96: function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
122: function maxMint(address) external view override returns (uint256 maxShares) {
165: function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {
220: function maxRedeem(address owner) external view override returns (uint256 maxShares) {
240: function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
```

-------

## G-04 Increments can be unchecked

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used. It saves **30-40 gas** per loop.

_There are 15 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
File: Ethos-Core/contracts/ActivePool.sol

108: for(uint256 i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```
File: Ethos-Core/contracts/CollateralConfig.sol

56: for(uint256 i = 0; i < _collaterals.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

351: for (uint i = 0; i < numCollaterals; i++) {
397: for (uint i = 0; i < numCollaterals; i++) {
640: for (uint i = 0; i < assets.length; i++) {
810: for (uint i = 0; i < collaterals.length; i++) {
831: for (uint i = 0; i < collaterals.length; i++) {
859: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

608: for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
690: for (vars.i = 0; vars.i < _n; vars.i++) {
808: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
882: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

206: for (uint i = 0; i < assets.length; i++) {
228: for (uint i = 0; i < collaterals.length; i++) {
240: for (uint i = 0; i < numCollaterals; i++) {
```

-------

## G-05 Require() or revert() strings longer than 32 bytes cost extra gas

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

Each extra memory word of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs **3 gas**

I suggest shortening the revert strings to fit in 32 bytes, or using custom errors.

_There are 49 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
File: Ethos-Core/contracts/ActivePool.sol

107: require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
133: require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
321-324: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == defaultPoolAddress,
            "ActivePool: Caller is neither BO nor Default Pool");
329-334: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
338-341: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
File: Ethos-Core/contracts/BorrowerOperations.sol

525: require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
529: require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
530: require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
534: require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
538: require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
543: require(status == 1, "BorrowerOps: Trove does not exist or is closed");
552: require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
561-563: require(
            !_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),
            "BorrowerOps: Operation not permitted during Recovery Mode"
568: require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
617: require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");
621: require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");
625: require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
629: require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");
633: require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
637: require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
641: require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
645: require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
650-651: require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must less than or equal to 100%");
653-654: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
File: Ethos-Core/contracts/LUSDToken.sol

134-136: require(
            msg.sender == guardianAddress || msg.sender == governanceAddress,
            "LUSD: Caller is not guardian or governance"
347-350: require(
            _recipient != address(0) &&
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
352-356: require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
362: require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");
367-371: require(
            troveManagers[msg.sender] ||
            stabilityPools[msg.sender] ||
            borrowerOperations[msg.sender],
            "LUSD: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"
377: require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");
384-386: require(
            troveManagers[msg.sender] || stabilityPools[msg.sender],
            "LUSD: Caller is neither TroveManager nor StabilityPool");
390: require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");
394: require(!mintingPaused, "LUSDToken: Minting is currently paused");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

849: require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
853: require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");
865: require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");
870: require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
874: require(_amount > 0, 'StabilityPool: Amount must be non-zero');
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

133: require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

253-257: require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool"
262: require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
266: require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
270: require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

150: require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
321: require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
436: require(loss <= allocation, "Strategy loss cannot be greater than allocation");
619: require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

98: require(_amount <= balanceOf(), "Ammount must be less than balance");
191-193: require(
            upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
            "Upgrade cooldown not initiated or still ongoing"
```

----------

## G-06 Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

See [this](https://github.com/ethereum/solidity/issues/9232) issue for a detail description of the issue.

_There are 8 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73: bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76: bytes32 public constant ADMIN = keccak256("ADMIN");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

49: bytes32 public constant KEEPER = keccak256("KEEPER");
50: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52: bytes32 public constant ADMIN = keccak256("ADMIN");
```

----------

## G-07 Splitting require() statements that use && saves gas

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by **3 gas

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
File: Ethos-Core/contracts/BorrowerOperations.sol

653: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
File: Ethos-Core/contracts/LUSDToken.sol

347-348: require(
            _recipient != address(0) &&
352-354: require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

-----------

## G-08 Use scientific notation rather than exponentiation

Use scientific notation (eg. `1e18`) rather than exponentiation (eg. `10**18`)

Scientific notation should be used for better code readability

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18;
```

------

## G-09 Use a more recent version of solidity

_This issue exists in all the In-scope contracts_. _There are 12 instances of this issue

[CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3) , [BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3) , [TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)] , [ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3) , [StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3) , [CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3) , [LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3) , [LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3) , [ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3) , [ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3) , [ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3) , [ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)

-----
