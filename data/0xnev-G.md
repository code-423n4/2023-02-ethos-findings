### Gas Optimization Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [G-01] | Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead | 7 |
| [G-02] | Splitting `require()` statements that uses `&&` saves gas | 5 |
| [G-03] | internal/private functions only called once can be inlined to save gas | 47 |
| [G-04] | Multiple accesses of a mapping/array should use a local variable cache | 59 |
| [G-05] | Use nested `if` statements to avoid multiple check combinations using `&&`| 11 |
| [G-06] | Avoid caching global variables | 2 |
| [G-07] | Check amount before execution of `transfer` functions for possible gas savings | 4 |
| [G-08] | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 6 |
| [G-09] | `<x> -= <y>` costs more gas than `<x> = <x> - <y>` for state variables | 11 |
| [G-10] | Use a more recent version of solidity | - |
| [G-11] | Do not initialize variables with default value | 4 |
| [G-12] | Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate | 8 |
| [G-13] | Unnecessary `return` statements in functions that do not return anything | 4 |
| [G-14] |Unnecessary `return` statements in functions that have already define named return variable(s) | 4 |

| Total Gas-Optimization Issues | 14 |
|:--:|:--:|

### [G-01] Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead
Context: 
```solidity
7 results - 3 files

/TroveManager.sol
  82: uint128 arrayIndex;
1321: uint128 index // dont need to downcast from uint256 to uint128

/StabilityPool.sol
 247: uint128 _epoch, uint128 _scale
 248: uint128 _epoch, uint128 _scale
 249: uint128 _currentEpoch
 250: uint128 _currentScale

 /ReaperStrategyGranarySupplyOnly.sol
  35: uint16 private constant LENDER_REFERRAL_CODE_NONE
```
Additionally, if `index` is declared using `uint256`, the downcasting to `uint128`in line 1329 is not required
```solidity
1321:   function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint256 index) {
1325:        // Push the Troveowner to the array
1326:        TroveOwners[_collateral].push(_borrower);
1327:
1328:        // Record the index of the new Troveowner on their Trove struct
1329:        index = TroveOwners[_collateral].length.sub(1);
1330:        Troves[_borrower][_collateral].arrayIndex = index;
1331:
1332:        return index;
1333:    }
```

The above instance can be refactored to:
```solidity
1321:   function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1325:        // Push the Troveowner to the array
1326:        TroveOwners[_collateral].push(_borrower);
1327:
1328:        // Record the index of the new Troveowner on their Trove struct
1329:        index = uint128(TroveOwners[_collateral].length.sub(1));
1330:        Troves[_borrower][_collateral].arrayIndex = index;
1331:
1332:        return index;
1333:    }
```

Description:
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. 


Recommendation:
Replace `uint` smaller than 32 bytes (256 bits) with `uint256` for instances that does not compromise on additional storage slots.

### [G-02] Splitting `require()` statements that uses `&&` saves gas
Context:
```solidity
5 results - 3 files

/BorrowerOperations.sol
653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:                "Max fee percentage must be between 0.5% and 100%");

/TroveManager.sol
1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

/LUSDToken.sol
347:        require(
348:            _recipient != address(0) && 
349:            _recipient != address(this),
350:            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:        );
352:        require(
353:            !stabilityPools[_recipient] &&
354:            !troveManagers[_recipient] &&
355:            !borrowerOperations[_recipient],
356:            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:        );
```

Description:
Instead of using the `&&` operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save ~8 Gas per `&&`.
The gas difference would only be realized if the revert condition is realized(met).

Recommendation:
Split `require` statements that uses `&&`. For example, the first instance can be refactored to: 
```solidity
require(_maxFeePercentage >= BORROWING_FEE_FLOOR, "Max fee percentage must be between 0.5% and 100%");
require( _maxFeePercentage <= DECIMAL_PRECISION, "Max fee percentage must be between 0.5% and 100%");
```

### [G-03] internal/private functions only called once can be inlined to save gas
Description:
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Context:
Consider inlining less complex functions such as instances of the `_require` helper:

```solidity
24 results - 6 files

/BorrowerOperations.sol
 533: function _requireSingularCollChange
 537: function _requireNonZeroAdjustment
 546: function _requireTroveisNotActive
 551: function _requireNonZeroDebtChange
 555: function _requireNotInRecoveryMode
 567: function _requireNoCollWithdrawal
 571: function _requireValidAdjustmentInCurrentMode
 624: function _requireNewICRisAboveOldICR
 636: function _requireValidLUSDRepayment

/TroveManager.sol
1538: function _requireMoreThanOneTroveInSystem

/ActivePool.sol
 320: function _requireCallerIsBorrowerOperationsOrDefaultPool
 337: function _requireCallerIsBOorTroveM

/StabilityPool.sol
 848: function _requireCallerIsActivePool
 852: function _requireCallerIsTroveManager
 869: function _requireUserHasDeposit
 873: function _requireNonZeroAmount

/LQTYStaking.sol
 261: function _requireCallerIsBorrowerOperations
 265: function _requireUserHasStake
 269: function _requireNonZeroAmount

/LUSDToken.sol
 360: function _requireCallerIsBorrowerOperations
 365: function _requireCallerIsBOorTroveMorSP
 375: function _requireCallerIsStabilityPool
 380: function _requireCallerIsTroveMorSP
 393: function _requireMintingNotPaused
```
For more complex functions listed below, consider inlining if it does not result in core functions being too complex:

```solidity
23 results - 5 files

/BorrowerOperations.sol
 438: function _getCollChange
 455: function _updateTroveFromAdjustment
 476: function _moveTokensAndCollateralfromAdjustment
 661: function _getNewNominalICRFromTroveChange

/TroveManager.sol
 478: function _getCappedOffsetVals 
 582: function _getTotalsFromLiquidateTrovesSequence_RecoveryMode 
 671: function _getTotalsFromLiquidateTrovesSequence_NormalMode 
 785: function _getTotalFromBatchLiquidate_RecoveryMode 
 864: function _getTotalsFromBatchLiquidate_NormalMode
1082: function _applyPendingRewards 
1213: function _computeNewStake
1321: function _addTroveOwnerToArray

/StabilityPool.sol
 421: function _updateG 
 439: function _computeLQTYPerUnitStaked 
 597: function _moveOffsetCollAndDebt
 637: function _getCollateralGainFromSnapshots
 693: function _getLQTYGainFromSnapshots
 729: function _getCompoundedDepositFromSnapshots
 776: function _sendLUSDtoStabilityPool
 794: function _sendLUSDToDepositor
 856: function _requireNoUnderCollateralizedTroves

/ReaperVaultV2.sol
 462: function _chargeFees

/ReaperStrategyGranarySupplyOnly.sol
 206: function _claimRewards
```

### [G-04] Multiple accesses of a mapping/array should use a local variable cache
Description:
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory

Context:

#### TroveManager.sol
Cache ` Troves[_borrower][_collateral]` in local storage
```solidity
14 results

/TroveManager.sol
 962:   function updateDebtAndCollAndStakesPostRedemption
 969:        Troves[_borrower][_collateral].debt = _newDebt;
 970:        Troves[_borrower][_collateral].coll = _newColl;
 977:             Troves[_borrower][_collateral].stake,

1066:    function _getCurrentTroveAmounts
1070:        uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);
1071:        uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

1082:    function _applyPendingRewards
1091:            Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);
1092:            Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);
1102:                Troves[_borrower][_collateral].debt,
1103:                Troves[_borrower][_collateral].coll,
1104:                Troves[_borrower][_collateral].stake,

1123:    function getPendingCollateralReward
1127:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }        
1129:        uint stake = Troves[_borrower][_collateral].stake;

1138:    function getPendingLUSDDebtReward
1142:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
1144:        uint stake = Troves[_borrower][_collateral].stake;

1164:    function getEntireDebtAndColl
1173:        debt = Troves[_borrower][_collateral].debt;
1174:        coll = Troves[_borrower][_collateral].coll;

1189:    function _removeStake
1190:        uint stake = Troves[_borrower][_collateral].stake;
1192:        Troves[_borrower][_collateral].stake = 0;

1201:    function _updateStakeAndTotalStakes
1202:        uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);
1203:        uint oldStake = Troves[_borrower][_collateral].stake;
1204:        Troves[_borrower][_collateral].stake = newStake;

1230:    function _redistributeDebtAndColl
1284:        Troves[_borrower][_collateral].status = closedStatus;
1285:        Troves[_borrower][_collateral].coll = 0;
1286:        Troves[_borrower][_collateral].debt = 0;

1339:    function _removeTroveOwner
1340:        Status troveStatus = Troves[_borrower][_collateral].status;
1344:        uint128 index = Troves[_borrower][_collateral].arrayIndex;

1567:    function increaseTroveColl
1569        uint newColl = Troves[_borrower][_collateral].coll.add(_collIncrease);
1570:       Troves[_borrower][_collateral].coll = newColl;

1574:    function decreaseTroveColl
1576:        uint newColl = Troves[_borrower][_collateral].coll.sub(_collDecrease);
1577:        Troves[_borrower][_collateral].coll = newColl;

1581:    function increaseTroveDebt
1583:        uint newDebt = Troves[_borrower][_collateral].debt.add(_debtIncrease);
1584:        Troves[_borrower][_collateral].debt = newDebt;

1588:    function decreaseTroveDebt
1590:        uint newDebt = Troves[_borrower][_collateral].debt.sub(_debtDecrease);
1591:        Troves[_borrower][_collateral].debt = newDebt;
```
Cache `totalStakes[_collateral]` in local storage:

```solidity
3 results

/TroveManager.sol
1189:    function _removeStake
1191:        totalStakes[_collateral] = totalStakes[_collateral].sub(stake);

1201:    function _updateStakeAndTotalStakes
1206:        totalStakes[_collateral] = totalStakes[_collateral].sub(oldStake).add(newStake);
1207:        emit TotalStakesUpdated(_collateral, totalStakes[_collateral]);

1230:     function _redistributeDebtAndColl
1255:        uint collateralRewardPerUnitStaked = collateralNumerator.div(totalStakes[_collateral]);
1256:        uint LUSDDebtRewardPerUnitStaked = LUSDDebtNumerator.div(totalStakes[_collateral]);
1257:
1258:        lastCollateralError_Redistribution[_collateral] = collateralNumerator.sub(collateralRewardPerUnitStaked.mul(totalStakes[_collateral]));
1259:        lastLUSDDebtError_Redistribution[_collateral] = LUSDDebtNumerator.sub(LUSDDebtRewardPerUnitStaked.mul(totalStakes[_collateral]));
```

Cache `totalStakesSnapshot[_collateral]` in local storage:

```solidity
2 results

/TroveManager.sol
1213:    function _computeNewStake
1224:            assert(totalStakesSnapshot[_collateral] > 0);
1225:            stake = _coll.mul(totalStakesSnapshot[_collateral]).div(totalCollateralSnapshot[_collateral]);

1305:    function _updateSystemSnapshots_excludeCollRemainder
1306:        totalStakesSnapshot[_collateral] = totalStakes[_collateral];
1312:        emit SystemSnapshotsUpdated(_collateral, totalStakesSnapshot[_collateral], totalCollateralSnapshot[_collateral]);
```

Cache `L_Collateral[_collateral]` in local storage

```solidity
2 results

/TroveManager.sol
1116:    function _updateTroveRewardSnapshots
1117:        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];
1119:        emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1230:    function _redistributeDebtAndColl
1262:        L_Collateral[_collateral] = L_Collateral[_collateral].add(collateralRewardPerUnitStaked);
1265:        emit LTermsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);
```

Cache `L_LUSDDebt[_collateral]` in local storage

```solidity
2 results

/TroveManager.sol
1116:    function _updateTroveRewardSnapshots
1118:        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];
1119:        emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1230:     function _redistributeDebtAndColl
1262:        L_Collateral[_collateral] = L_Collateral[_collateral].add(collateralRewardPerUnitStaked);
1263:        L_LUSDDebt[_collateral] = L_LUSDDebt[_collateral].add(LUSDDebtRewardPerUnitStaked);
1264:
1265:        emit LTermsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);
```

Cache `rewardSnapshots[_borrower][_collateral]` in local storage
```solidity
2 results

/TroveManager.sol
1116:    function _updateTroveRewardSnapshots
1117:        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];
1118:        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];

1278:    function _closeTrove
1288:        rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289        rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
```

Cache `TroveOwners[_collateral]` in local storage

```solidity
2 results

/TroveManager.sol
1321:    function _addTroveOwnerToArray
1326:        TroveOwners[_collateral].push(_borrower);
1329:        index = uint128(TroveOwners[_collateral].length.sub(1));

1339:    function _removeTroveOwner
1350:        address addressToMove = TroveOwners[_collateral][idxLast];
1352:        TroveOwners[_collateral][index] = addressToMove;
1356:        TroveOwners[_collateral].pop();
```

Cache `lastCollateralError_Redistribution[_collateral]` in local storage

```solidity
1 result

/TroveManager.sol
1230:    function _redistributeDebtAndColl
1251:        uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);
1258:        lastCollateralError_Redistribution[_collateral] = collateralNumerator.sub(collateralRewardPerUnitStaked.mul(totalStakes[_collateral]));
```

Cache `lastLUSDDebtError_Redistribution[_collateral]` in local storage

```solidity
1 result

/TroveManager.sol
1230:    function _redistributeDebtAndColl
1252:        uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);
1259:        lastLUSDDebtError_Redistribution[_collateral] = LUSDDebtNumerator.sub(LUSDDebtRewardPerUnitStaked.mul(totalStakes[_collateral]));
```
#### ActivePool.sol

Cache `collAmount[_collateral]` in local storage

```solidity
2 results

/ActivePool.sol
171:    function sendCollateral
175:        collAmount[_collateral] = collAmount[_collateral].sub(_amount);
176:        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);

204:    function pullCollateralFromBorrowerOperationsOrDefaultPool
207:        collAmount[_collateral] = collAmount[_collateral].add(_amount);
208:        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
```

Cache `LUSDDebt[_collateral]` in local storage

```solidity
2 results

/ActivePool.sol
190:    function increaseLUSDDebt
193:        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
194:        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);

197:    function decreaseLUSDDebt
200:        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
201:        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
```

Cache `yieldingAmount[_collateral]` in local storage

```solidity
1 result

/ActivePool.sol
239:    function _rebalance
243:        vars.currentAllocated = yieldingAmount[_collateral];
263:        vars.yieldingAmount = yieldingAmount[_collateral];
268:            yieldingAmount[_collateral] = vars.yieldingAmount;
273:            yieldingAmount[_collateral] = vars.yieldingAmount;
```

Cache `yieldGenerator[_collateral]` in local storage

```solidity
1 result

/ActivePool.sol
239:    function _rebalance
246:        vars.yieldGenerator = IERC4626(yieldGenerator[_collateral]);
279:            IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));
280:            IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));
282:            IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));
```
#### StabilityPool.sol

Cache `collAmounts[_collateral]` in local storage

```solidity
3 results

/StabilityPool.sol
486:    function updateRewardSum
497:        uint sum = collAmounts[_collateral].add(_collToAdd);
498:        collAmounts[_collateral] = sum;

597:    function _moveOffsetCollAndDebt
608:        uint sum = collAmounts[_collateral].add(_collToAdd);
609:        collAmounts[_collateral] = sum;

783:    function _sendCollateralGainToDepositor
785:        uint newCollAmount = collAmounts[_collateral].sub(_amount);
786:        collAmounts[_collateral] = newCollAmount;
```

Cache `depositSnapshots[_depositor]` in local storage

```solidity
1 result

/StabilityPool.sol
803:    function _updateDepositAndSnapshots
810:            for (uint i = 0; i < collaterals.length; i++) {
811:                delete depositSnapshots[_depositor].S[collaterals[i]];
812:            }
813:            delete depositSnapshots[_depositor];
825:        depositSnapshots[_depositor].P = currentP;
826:        depositSnapshots[_depositor].G = currentG;
827:        depositSnapshots[_depositor].scale = currentScaleCached;
828:        depositSnapshots[_depositor].epoch = currentEpochCached;
831:        for (uint i = 0; i < collaterals.length; i++) {
832:            address collateral = collaterals[i];
833:            uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
834:            depositSnapshots[_depositor].S[collateral] = currentS;
835:            amounts[i] = currentS;
836:        }
```
Cache `epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral]` in local storage

```solidity
1 result

/StabilityPool.sol
547:    function _updateRewardSumAndProduct
560:        uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral];
571:        epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral] = newS;
```

Cache `epochToScaleToSum[vars.epochSnapshot]` in local storage

```solidity
1 result

/StabilityPool.sol
657:    function _getSingularCollateralGain
670:        vars.firstPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot][_collateral].sub(vars.S_Snapshot);
671:        vars.secondPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot.add(1)][_collateral].div(SCALE_FACTOR);
```

Cache `epochToScaleToG[currentEpoch][currentScale]` in local storage

```solidity
1 result

/StabilityPool.sol
421:    function _updateG
434:        epochToScaleToG[currentEpoch][currentScale] = epochToScaleToG[currentEpoch][currentScale].add(marginalLQTYGain);
435:
436:        emit G_Updated(epochToScaleToG[currentEpoch][currentScale], currentEpoch, currentScale);
```

Cache `epochToScaleToG[epochSnapshot]` in local storage

```solidity
1 result

/StabilityPool.sol
693:    function _getLQTYGainFromSnapshots
704:        uint firstPortion = epochToScaleToG[epochSnapshot][scaleSnapshot].sub(G_Snapshot);
705:        uint secondPortion = epochToScaleToG[epochSnapshot][scaleSnapshot.add(1)].div(SCALE_FACTOR);
```

Cache `lastCollateralError_Offset[_collateral]` in local storage
```solidity
1 result

/StabilityPool.sol
504:    function _computeRewardsPerUnitStaked
524:        uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);
541:        lastCollateralError_Offset[_collateral] = collNumerator.sub(collGainPerUnitStaked.mul(_totalLUSDDeposits));
```
#### LQTYStaking.sol

Cache `stakes[msg.sender]` in local storage

```solidity
2 results

/LQTYStaking.sol
104:    function stake
107:        uint currentStake = stakes[msg.sender];
123:        stakes[msg.sender] = newStake;

142:    function unstake
143:        uint currentStake = stakes[msg.sender];
158:            stakes[msg.sender] = newStake;
```
Cache `F_Collateral[_collateral]` in local storage

```solidity
2 results

/LQTYStaking.sol
177:    function increaseF_Collateral
183:        F_Collateral[_collateral] = F_Collateral[_collateral].add(collFeePerLQTYStaked);
184:        emit F_CollateralUpdated(_collateral, F_Collateral[_collateral]);

225:    function _updateUserSnapshots
228:        for (uint i = 0; i < collaterals.length; i++) {
229:            address collateral = collaterals[i];
230:            snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];
231:            amounts[i] = F_Collateral[collateral];
232:        }
```

Cache `snapshots[_user]` in local storage

```solidity
1 result

/LQTYStaking.sol
225:    function _updateUserSnapshots
228:        for (uint i = 0; i < collaterals.length; i++) {
229:            address collateral = collaterals[i];
230:            snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];
231:            amounts[i] = F_Collateral[collateral];
232:        }
234:        snapshots[_user].F_LUSD_Snapshot = fLUSD;
```
#### LUSDToken.sol

Cache ` _balances[sender]` in local storage

```solidity
1 result

/LUSDToken.sol
311:    function _transfer
315:         _balances[sender] = _balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");
```

Cache `_balances[recipient]` in local storage

```solidity
1 result

/LUSDToken.sol
311:    function _transfer
316:        _balances[recipient] = _balances[recipient].add(amount);
```
Cache `_balances[account]` in local storage
```solidity
2 results

/LUSDToken.sol
320:    function _mint
324:        _balances[account] = _balances[account].add(amount);

328:    function _burn
331:        _balances[account] = _balances[account].sub(amount, "ERC20: burn amount exceeds balance");
```

#### ReaperVaultV2.sol

Cache `strategies[_strategy]` in local storage

```solidity
4 results

/ReaperVaultV2.sol
144:    function addStrategy
152:        require(strategies[_strategy].activation == 0, "Strategy already added");
158:        strategies[_strategy] = StrategyParams({

178:    function updateStrategyFeeBPS
180:        require(strategies[_strategy].activation != 0, "Invalid strategy address");
182:        strategies[_strategy].feeBPS = _feeBPS;

191:    function updateStrategyAllocBPS
193:        require(strategies[_strategy].activation != 0, "Invalid strategy address");
194:        totalAllocBPS -= strategies[_strategy].allocBPS;
195:        strategies[_strategy].allocBPS = _allocBPS;

205:     function revokeStrategy
210:        if (strategies[_strategy].allocBPS == 0) {
214:        totalAllocBPS -= strategies[_strategy].allocBPS;
215:        strategies[_strategy].allocBPS = 0;
```

Cache `strategies[stratAddr]` in local storage. To further optimize code, avoid caching the global variable `msg.sender` in `stratAddr` and use `msg.sender` directly. In that case, cache `strategies[msg.sender]` instead:

```solidity
2 results

/ReaperVaultV2.sol
225:    function availableCapital() public view returns (int256) {
226:        address stratAddr = msg.sender;
228:            return -int256(strategies[stratAddr].allocated);
231:        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
232:        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;

359:     function _withdraw
379:                uint256 strategyBal = strategies[stratAddr].allocated;
395:                strategies[stratAddr].allocated -= actualWithdrawn;
```

### [G-05] Use nested `if` statements to avoid multiple check combinations using `&&`
Context:
```solidity
11 results - 3 files

/BorrowerOperations.sol
282:    function _adjustTrove
311:        if (_isDebtIncrease && !isRecoveryMode) { 
337:        if (!_isDebtIncrease && _LUSDChange > 0) {

/TroveManager.sol
357:    function _liquidateRecoveryMode
397:        } else if ((_ICR > _100pct) && (_ICR < _MCR)) {
415:        } else if ((_ICR >= _MCR) && (_ICR < _TCR) && (singleLiquidation.entireTroveDebt <= _LUSDInStabPool)) {

582:    function _getTotalsFromLiquidateTrovesSequence_RecoveryMode
616:                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }
651:             else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {

785:    function _getTotalFromBatchLiquidate_RecoveryMode
817:                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }
853:            else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {

/ActivePool.sol
239:    function _rebalance
264:        if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
269:        } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
```

Description:
Using nested `if` statements is cheaper than using `&& ` for multiple check combinations. Additionally, it improves readability of code and better coverage reports.

Recommendation:
Replace `&&` used within `if` and `else if` statements with nested `if` statements

### [G-06] Avoid caching global variables
Context:
```solidity
2 results - 1 file

/ReaperVaultV2.sol
225:    function availableCapital() public view returns (int256) {
226:        address stratAddr = msg.sender;

493:    function report(int256 _roi, uint256 _repayment) external returns (uint256) {
494:        LocalVariables_report memory vars;
495:        vars.stratAddr = msg.sender;
```

Description: 
Caching global variables is more expensive than using the actual variable itself

Recommendation:
use `msg.sender` directly instead of caching it

### [G-07] Check amount before execution of `transfer` functions for possible gas savings
```solidity
4 results - 4 files

/CommunityIssuance.sol
124:    function sendOath
127:        OathToken.transfer(_account, _OathAmount);

/LQTYStaking.sol
104:    function stake
135:             lusdToken.transfer(msg.sender, LUSDGain);

142:    function unstake
171:        lusdToken.transfer(msg.sender, LUSDGain);

/LUSDToken.sol
311:    function _transfer(address sender, address recipient, uint256 amount) internal {
```

Description:
Before execution of `transfer` functions, we should check if amount is zero to prevent wastage of gas when `transfer` functions do not do anything upon execution

Recommendation:
Add checks such as `require(amount != 0, "cannot transfer 0")` statement to check amount transferred using `transfer` functions like the one done in the function below:

```solidity
/CommunityIssuance.sol
101:    function fund(uint amount) external onlyOwner {
102:        require(amount != 0, "cannot fund 0");
103:        OathToken.transferFrom(msg.sender, address(this), amount);
```

### [G-08] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables 
Context:

```solidity
6 results - 1 file

/ReaperVaultV2.sol
168: totalAllocBPS += _allocBPS;
196: totalAllocBPS += _allocBPS;
450: stratParams.losses += loss;
505: strategy.gains += vars.gain;
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;
```
Recommendation:
Replace `<x> += <y>` with `<x> = <x> + <y>` for state variables 

### [G-09] `<x> -= <y>` costs more gas than `<x> = <x> - <y>` for state variables
Context:
```solidity
11 results - 1 file

/ReaperVaultV2.sol
194: totalAllocBPS -= strategies[_strategy].allocBPS;
214: totalAllocBPS -= strategies[_strategy].allocBPS;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment; // tracked for return value
```
Recommendation:
Replace `<x> -= <y>` with `<x> = <x> - <y>` for state variables 

### [G-10] Use a more recent version of solidity
Description:

- Use a solidity version of at least 0.8 to get default underflow/overflow checks.
- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining 
- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads 
- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings 
- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

We can avoid using SafeMath library in the following contracts by using version 0.8.0+:
```solidity
/ActivePool.sol
/StabilityPool.sol
/CommunityIssuance.sol
/LQTYStaking.sol
/LUSDToken.sol
```

Additionally, the use of version 0.8.0+ allows the use of `unchecked` keyword which can help further save gas in `for` loops. In the following `for ` loops, `++i/i++ ` can be `unchecked{++i}/unchecked{i++}` as it is not possible for them to overflow, only if more recent version of solidity is used:

```solidity

/CollateralConfig.sol
56: for(uint256 i = 0; i < _collaterals.length; i++)

/TroveManager.sol
608: for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++)
690: for (vars.i = 0; vars.i < _n; vars.i++)
808: for (vars.i = 0; vars.i < _troveArray.length; vars.i++)
882: for (vars.i = 0; vars.i < _troveArray.length; vars.i++)

/ActivePool.sol
108: for(uint256 i = 0; i < numCollaterals; i++)

/StabilityPool.sol
351: for (uint i = 0; i < numCollaterals; i++)
397: for (uint i = 0; i < numCollaterals; i++)
640: for (uint i = 0; i < assets.length; i++)
810: for (uint i = 0; i < collaterals.length; i++)
831: for (uint i = 0; i < collaterals.length; i++)
859: for (uint i = 0; i < numCollaterals; i++)

/LQTYStaking.sol
206: for (uint i = 0; i < assets.length; i++)
228: for (uint i = 0; i < collaterals.length; i++)
240: for (uint i = 0; i < numCollaterals; i++)
```

### [G-11] Do not initialize variables with default value
Context:
```solidity
4 results - 4 files

/CollateralConfig.sol
18: bool public initialized = false;

/ActivePool.sol
32: bool public addressesSet = false;

/CommunityIssuance.sol
21: bool public initialized = false;

/LUSDToken.sol
37: bool public mintingPaused = false;
```

Description:
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value wastes gas.

Recommendation:
Remove explicit initializations for default values

### [G-12] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
Context:

```solidity
8 results - 4 files

/TroveManager.sol
 88: mapping (address => uint) public totalStakes;
 91: mapping (address => uint) public totalStakesSnapshot;
 94: mapping (address => uint) public totalCollateralSnapshot;

104: mapping (address => uint) public L_Collateral;
105: mapping (address => uint) public L_LUSDDebt;

119: mapping (address => uint) public lastCollateralError_Redistribution;
120: mapping (address => uint) public lastLUSDDebtError_Redistribution;

/ActivePool.sol
 41: mapping (address => uint256) internal collAmount;  // collateral => amount tracker
 42: mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

 44: mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
 43: mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
 45: mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
 46: mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute

/LQTYStaking.sol
 25: mapping( address => uint) public stakes;
 28: mapping (address => uint) public F_Collateral;

/LUSDToken.sol
 54: mapping (address => uint256) private _nonces;
 57: mapping (address => uint256) private _balances;
 58: mapping (address => mapping (address => uint256)) private _allowances;

 62: mapping (address => bool) public troveManagers;
 63: mapping (address => bool) public stabilityPools;
 64: mapping (address => bool) public borrowerOperations;
```

Description:
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot

Recommendation:
Combine mappings multiple address mappings to a single mapping of an address to a struct, where appropriate

### [G-13] Unnecessary `return` statements in functions that do not return anything
Context:
```solidity
4 results - 1 file

/TroveManager.sol
1076:    function applyPendingRewards(address _borrower, address _collateral) external override {
1077:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1078:        return _applyPendingRewards(activePool, defaultPool, _borrower, _collateral);
1079:    }

1111:    function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {
1112:        _requireCallerIsBorrowerOperations();
1113:       return _updateTroveRewardSnapshots(_borrower, _collateral);
1114:    }

1183:    function removeStake(address _borrower, address _collateral) external override {
1184:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1185:        return _removeStake(_borrower, _collateral);
1186:    }

1273:    function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {
1274:        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
1275:        return _closeTrove(_borrower, _collateral, Status(_closedStatusNum));
1276:    }
```

Description:
Some functions use the return keyword after checks. However, the corresponding functions invoked does not return anything. As such, `return` keyword is not required

Recommendation:
Remove return keyword in functions that do not return anything

### [G-14] Unnecessary return statements in functions that have already define named return variable(s)
Context:
```solidity
4 results - 2 files

/TroveManager.sol
357:    function _liquidateRecoveryMode(
358:        IActivePool _activePool,
359:        IDefaultPool _defaultPool,
360:        address _collateral,
361:        address _borrower,
362:        uint _ICR,
363:        uint _LUSDInStabPool,
364:        uint _TCR,
365:        uint _price,
366:        uint256 _MCR
367:    )
368:        internal
369:        returns (LiquidationValues memory singleLiquidation)

432:            LiquidationValues memory zeroVals;
433:            return zeroVals;
434:        }
435:
436:        return singleLiquidation;
437:    }

1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1329:        index = uint128(TroveOwners[_collateral].length.sub(1));
1330:        Troves[_borrower][_collateral].arrayIndex = index;
1331:
1332:        return index;
1333:    }

/StabilityPool.sol
504:    function _computeRewardsPerUnitStaked(
505:        address _collateral,
506:        uint _collToAdd,
507:        uint _debtToOffset,
508:        uint _totalLUSDDeposits
509:    )
510:        internal
511:        returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)

543:        return (collGainPerUnitStaked, LUSDLossPerUnitStaked);
```
Recommendation:
Remove `return` statements for functions that have already define named return variable(s)