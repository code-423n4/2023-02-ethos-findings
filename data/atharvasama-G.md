# Gas Optimizations list

| Number | Details                                                                                                      | Instances |
| ------ | ------------------------------------------------------------------------------------------------------------ | --------- |
| 1      | ```x += y``` COSTS MORE GAS THAN ```x = x + y``` FOR STATE VARIABLES                                         | 15        |
| 2      | CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS                                                                | 23        |
| 3      | ++i/i++ CAN BE UNCHECKED                                                                                     | 13        |
| 4      | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMNBINED INTO A SINGLE MAPPING TO A STRUCT                              | 21        |
| 5      | USE A MORE RECENT VERSION OF SOLIDITY                                                                        | 12        |
| 6      | AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS                                                     | 6         |
| 7      | USE ```require``` INSTEAD OF ```assert```                                                                    | 20        |
| 8      | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                              | 4         |
| 9      | USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE                                                  | 11        |
| 10     | OPTIMIZE NAMES TO SAVE GAS                                                                                   | 12        |
| 11     | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS                                               | 24        |
| 12     | STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS                                                       |  2         |
| 13     | ```require()``` OR ```revert()``` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION |           |
| 14     | SPLITTING ```require()``` STATEMENTS THAT USE && SAVES GAS                                                   |       7    |
| 15     | EXPRESSION CAN BE UNCHECKED WHEN OVERFLOW IS NOT POSSIBLE                                                    |  1         |
| 16     | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                  |     2      |

# Gas Optimizations
## [G-01] ```x += y``` COSTS MORE GAS THAN ```x = x + y``` FOR STATE VARIABLES
Using the addition operator instead of plus-equals saves some gas for state variables.

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
168:         totalAllocBPS += _allocBPS;
194:         totalAllocBPS -= strategies[_strategy].allocBPS;
214:         totalAllocBPS -= strategies[_strategy].allocBPS;
395:                 strategies[stratAddr].allocated -= actualWithdrawn;
396:                 totalAllocated -= actualWithdrawn;
444:                 stratParams.allocBPS -= bpsChange;
445:                 totalAllocBPS -= bpsChange;
450:         stratParams.losses += loss;
451:         stratParams.allocated -= loss;
452:         totalAllocated -= loss;
505:             strategy.gains += vars.gain;
514:                 strategy.allocated -= vars.debtPayment;
515:                 totalAllocated -= vars.debtPayment;
520:             strategy.allocated += vars.credit;
521:             totalAllocated += vars.credit;
```

## [G-02] CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS       
[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
```js
352:             address collateral = assets[i];
353:             uint amount = amounts[i];
398:             address collateral = assets[i];
399:             uint amount = amounts[i];
832:             address collateral = collaterals[i];
833:             uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
860:             address collateral = collaterals[i];
861:             uint price = priceFeed.fetchPrice(collateral);
862:             address lowestTrove = sortedTroves.getLast(collateral);
863:             uint256 collMCR = collateralConfig.getCollateralMCR(collateral);
864:             uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
```js
610:             address nextUser = _sortedTroves.getPrev(_collateral, vars.user);
```

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
```js
207:             address collateral = assets[i];
208:             uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];
229:             address collateral = collaterals[i];
242:                 address collateral = assets[i];
```

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
```js
120:             uint256 amount = startToken.balanceOf(address(this));
```

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
265:             address strategy = _withdrawalQueue[i];
378:                 address stratAddr = withdrawalQueue[i];
379:                 uint256 strategyBal = strategies[stratAddr].allocated;
384:                 uint256 remaining = value - vaultBalance;
385:                 uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
386:                 uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
```

## [G-03] ++i/i++ CAN BE UNCHECKED 
++i/i++ can be done in an unchecked block in for loops where the operation would not cause an overflow

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol):
```js
351:         for (uint i = 0; i < numCollaterals; i++) {
397:         for (uint i = 0; i < numCollaterals; i++) {
640:         for (uint i = 0; i < assets.length; i++) {
810:             for (uint i = 0; i < collaterals.length; i++) {
831:         for (uint i = 0; i < collaterals.length; i++) {
859:         for (uint i = 0; i < numCollaterals; i++) {
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol):
```js
608:         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
690:         for (vars.i = 0; vars.i < _n; vars.i++) {
808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol):
```js
206:         for (uint i = 0; i < assets.length; i++) {
228:         for (uint i = 0; i < collaterals.length; i++) {
240:         for (uint i = 0; i < numCollaterals; i++) {
```

## [G-04] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMNBINED INTO A SINGLE MAPPING TO A STRUCT
If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Depending on the circumstances and sizes of types, can avoid a Gsset per mapping combined.

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol):
```js
41:     mapping (address => uint256) internal collAmount;  // collateral => amount tracker
42:     mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker 
44:     mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
45:     mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
46:     mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
47:     mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
```

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
```js
56:     // User data for LUSD token
57:     mapping (address => uint256) private _balances;
58:     mapping (address => mapping (address => uint256)) private _allowances;  
     
60:     // --- Addresses ---
61:     // mappings store addresses of old versions so they can still burn (close troves)
62:     mapping (address => bool) public troveManagers;
63:     mapping (address => bool) public stabilityPools;
64:     mapping (address => bool) public borrowerOperations;
```

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
```js
186:     mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct
187:     mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
```js
85:     // user => (collateral type => trove)
86:     mapping (address => mapping (address => Trove)) public Troves;
87: 
88:     mapping (address => uint) public totalStakes;

90:     // Snapshot of the value of totalStakes for each collateral, taken immediately after the latest liquidation
91:     mapping (address => uint) public totalStakesSnapshot;
92: 
93:     // Snapshot of the total collateral across the ActivePool and DefaultPool, immediately after the latest liquidation.
94:     mapping (address => uint) public totalCollateralSnapshot;

104:     mapping (address => uint) public L_Collateral;
105:     mapping (address => uint) public L_LUSDDebt;

118      // Error trackers for the trove redistribution calculation
119:     mapping (address => uint) public lastCollateralError_Redistribution;
120:     mapping (address => uint) public lastLUSDDebtError_Redistribution;
```

## [G-05] USE A MORE RECENT VERSION OF SOLIDITY
Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

>[Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
>
>[Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
>
>[Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
>
>[Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
>
>[Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts//StabilityPool.sol)
>
>[Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
>
>[Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
>
>[Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
>
>[Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
>
>[Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
>
>[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperBaseStrategyV4.sol)
>
>[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

## [G-06] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS
Before Solidity 0.8.10 the compiler inserted extra code to check for contract existence for external function calls ```EXTCODESIZE``` (100 gas). In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
```js
179:             IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest);
```

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
328:         token.safeTransferFrom(msg.sender, address(this), _amount);

410:         token.safeTransfer(_receiver, value);

526:             token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);

528:             token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);

642:         IERC20Metadata(_token).safeTransfer(msg.sender, amount);
```

## [G-07] USE ```require``` INSTEAD OF ```assert```  
```assert()``` function when false, uses up all the remaining gas and reverts all the changes made.

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol):
```js
128:         assert(MIN_NET_DEBT > 0);

197:         assert(vars.compositeDebt > 0);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331:         assert(_collWithdrawal <= vars.coll); 
```

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol):
```js
312:         assert(sender != address(0));

313:         assert(recipient != address(0));

321:         assert(account != address(0));

329:         assert(account != address(0));

337:         assert(owner != address(0));

338:         assert(spender != address(0));
```

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol): 
```js
526:         assert(_debtToOffset <= _totalLUSDDeposits);

551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
 
591:         assert(newP > 0);
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol):
```js
418:             assert(_LUSDInStabPool != 0);

1226:             assert(totalStakesSnapshot[_collateral] > 0);

1281:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1344:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1350:         assert(index <= idxLast);

1416:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1491:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```

## [G-08] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from public to external and can save gas by doing so.

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
```js
1442:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1047:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {
```

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
```js
62:     function initialize(
63:         address _vault,
64:         address[] memory _strategists,
65:         address[] memory _multisigRoles,
66:         IAToken _gWant
67:     ) public initializer {
```

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
295:     function getPricePerFullShare() public view returns (uint256) {
```


## [G-09] USE ```bytes32``` INSTEAD OF ```string``` WHEREVER POSSIBLE
[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol): 
```js
30:     string constant public NAME = "ActivePool";
```

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol):
```js
21:     string constant public NAME = "BorrowerOperations";
```

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol):
```js
150:     string constant public NAME = "StabilityPool";
```

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol): 
```js
19:     string constant public NAME = "CommunityIssuance";
```

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol):
```js
23:     string constant public NAME = "LQTYStaking"; 
```

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol):
```js
32:     string constant internal _NAME = "LUSD Stablecoin";
33:     string constant internal _SYMBOL = "LUSD";
34:     string constant internal _VERSION = "1";

399:     function name() external view override returns (string memory) {
 
403:     function symbol() external view override returns (string memory) {

411:     function version() external view override returns (string memory) {
```

## [G-10] OPTIMIZE NAMES TO SAVE GAS
Function hashes that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted. Prioritize the most called functions and sort and rename them according to the function hashes/method IDs.
The method IDs in the TroveManager.sol will be used the most. A lower method ID may be given to the most frequently used functions. 

```json
Sighash   |   Function Signature
========================
45978366  =>  increaseTroveColl(address,address,uint256)
db5e7321  =>  setAddresses(address,address,address,address,address,address,address,address,address,address,address,address,address)
40c315d2  =>  getTroveOwnersCount(address)
8424ce77  =>  getTroveFromTroveOwnersArray(address,uint256)
86b9d81f  =>  liquidate(address,address)
5d1cdbca  =>  _liquidateNormalMode(IActivePool,IDefaultPool,address,address,uint256)
37954acb  =>  _liquidateRecoveryMode(IActivePool,IDefaultPool,address,address,uint256,uint256,uint256,uint256,uint256)
7f551157  =>  liquidateTroves(address,uint256)
7c92590f  =>  batchLiquidateTroves(address,address[])
bef708ca  =>  _addLiquidationValuesToTotals(LiquidationTotals,LiquidationValues)
e0d64107  =>  _sendGasCompensation(IActivePool,address,address,uint256,uint256)
a444f0ad  =>  _movePendingTroveRewardsToActivePool(IActivePool,IDefaultPool,address,uint256,uint256)
bed8854b  =>  redeemCloseTrove(address,address,uint256,uint256)
bc9b5bd5  =>  reInsert(address,address,uint256,address,address)
aad64613  =>  updateDebtAndCollAndStakesPostRedemption(address,address,uint256,uint256)
40a44bd1  =>  redeemCollateral(address,uint256,address,address,address,uint256,uint256,uint256)
90ec2301  =>  getNominalICR(address,address)
b1eafaab  =>  getCurrentICR(address,address,uint256)
8d06f773  =>  _getCurrentTroveAmounts(address,address)
df4fb9ef  =>  applyPendingRewards(address,address)
dd61b2d3  =>  _applyPendingRewards(IActivePool,IDefaultPool,address,address)
c7a1bbdb  =>  updateTroveRewardSnapshots(address,address)
4321cdb5  =>  _updateTroveRewardSnapshots(address,address)
752fd9a1  =>  getPendingCollateralReward(address,address)
738009e6  =>  getPendingLUSDDebtReward(address,address)
2dcfca60  =>  hasPendingRewards(address,address)
26f7a0d4  =>  getEntireDebtAndColl(address,address)
fb4e863c  =>  removeStake(address,address)
89a8fa1c  =>  _removeStake(address,address)
dbe9f919  =>  updateStakeAndTotalStakes(address,address)
e73e5699  =>  _updateStakeAndTotalStakes(address,address)
621117b6  =>  closeTrove(address,address,uint256)
f924be84  =>  _closeTrove(address,address,Status)
417b7831  =>  _updateSystemSnapshots_excludeCollRemainder(IActivePool,address,uint256)
31bd9300  =>  _addTroveOwnerToArray(address,address)
788515eb  =>  _removeTroveOwner(address,address,uint256)
49fe8858  =>  getTCR(address,uint256)
8a490ebf  =>  checkRecoveryMode(address,uint256)
b83c7774  =>  _checkPotentialRecoveryMode(uint256,uint256,uint256,uint256,uint256)
a1a79bea  =>  updateBaseRateFromRedemption(uint256,uint256,uint256,uint256)
2b11551a  =>  getRedemptionRate()
c52861f2  =>  getRedemptionRateWithDecay()
b2c77626  =>  _calcRedemptionRate(uint256)
e85b603c  =>  getRedemptionFee(uint256)
d5b35635  =>  getRedemptionFeeWithDecay(uint256)
f36b2425  =>  getBorrowingRate()
66ca4a21  =>  getBorrowingRateWithDecay()
3d43ce81  =>  _calcBorrowingRate(uint256)
631203b0  =>  getBorrowingFee(uint256)
477d66cf  =>  getBorrowingFeeWithDecay(uint256)
181f90a4  =>  _calcBorrowingFee(uint256,uint256)
5dba4c4a  =>  decayBaseRateFromBorrowing()
ad7cc904  =>  _updateLastFeeOpTime()
2ee832aa  =>  _calcDecayedBaseRate()
d5eab66b  =>  _minutesPassedSinceLastFeeOp()
3a2f2848  =>  _requireCallerIsBorrowerOperations()
cf3a4303  =>  _requireCallerIsRedemptionHelper()
92a948f0  =>  _requireCallerIsBorrowerOperationsOrRedemptionHelper()
3a988cee  =>  _requireTroveIsActive(address,address)
141190dd  =>  _requireMoreThanOneTroveInSystem(uint256,address)
614d1209  =>  getTroveStatus(address,address)
dc3fe76f  =>  getTroveStake(address,address)
f3e2c608  =>  getTroveDebt(address,address)
305a5369  =>  getTroveColl(address,address)
8a25363c  =>  setTroveStatus(address,address,uint256)
2fae7700  =>  decreaseTroveColl(address,address,uint256)
fe4b0df6  =>  increaseTroveDebt(address,address,uint256)
8463b16a  =>  decreaseTroveDebt(address,address,uint256)
066bbf49  =>  _calcRedemptionFee(uint256,uint256)
080eeb65  =>  _computeNewStake(address,uint256)
0903c1a5  =>  _redistributeDebtAndColl(IActivePool,IDefaultPool,address,uint256,uint256,uint256)
099a676e  =>  burnLUSDAndEmitRedemptionEvent(address,address,uint256,uint256,uint256,uint256)
0e0d7bfe  =>  addTroveOwnerToArray(address,address)
```

## [G-11] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
```js
438:     function _getCollChange(
439:         uint _collReceived,
440:         uint _requestedCollWithdrawal
441:     )
442:         internal
443:         pure
444:         returns(uint collChange, bool isCollIncrease)

455:     function _updateTroveFromAdjustment
456:     (
457:         ITroveManager _troveManager,
458:         address _borrower,
459:         address _collateral,
460:         uint _collChange,
461:         bool _isCollIncrease,
462:         uint _debtChange,
463:         bool _isDebtIncrease
464:     )
465:         internal
466:         returns (uint, uint)


476:     function _moveTokensAndCollateralfromAdjustment
477:     (
478:         IActivePool _activePool,
479:         ILUSDToken _lusdToken,
480:         address _borrower,
481:         address _collateral,
482:         uint _collChange,
483:         bool _isCollIncrease,
484:         uint _LUSDChange,
485:         bool _isDebtIncrease,
486:         uint _netDebtChange
487:     )
488:         internal

533:     function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

537:     function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

551:     function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

555:     function _requireNotInRecoveryMode(
556:         address _collateral,
557:         uint _price,
558:         uint256 _CCR,
559:         uint256 _collateralDecimals
560:     ) internal view {

567:     function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {

571:     function _requireValidAdjustmentInCurrentMode 
572:     (
573:         bool _isRecoveryMode,
574:         address _collateral,
575:         uint _collWithdrawal,
576:         bool _isDebtIncrease, 
577:         LocalVariables_adjustTrove memory _vars
578:     ) 
579:         internal 
580:         view 

624:     function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

636:     function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

661:     function _getNewNominalICRFromTroveChange
662:     (
663:         uint _coll,
664:         uint _debt,
665:         uint _collChange,
666:         bool _isCollIncrease,
667:         uint _debtChange,
668:         bool _isDebtIncrease,
669:         uint256 _collateralDecimals
670:     )
671:         pure
672:         internal
673:         returns (uint)

421:     function _updateG(uint _LQTYIssuance) internal {

439:     function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) { 

637:     function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

693:     function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

729:     function _getCompoundedDepositFromSnapshots(
730:         uint initialDeposit,
731:         Snapshots memory snapshots
732:     )
733:         internal
734:         view
735:         returns (uint)

776:     function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

783:     function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {

794:     function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

856:     function _requireNoUnderCollateralizedTroves() internal {

869:     function _requireUserHasDeposit(uint _initialDeposit) internal pure {
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
```js
479:     function _getCappedOffsetVals
480:     (
481:         uint _entireTroveDebt,
482:         uint _entireTroveColl,
483:         uint _price,
484:         uint256 _MCR,
485:         uint _collDecimals
486:     )
487:         internal
488:         pure
489:         returns (LiquidationValues memory singleLiquidation)
```

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
462:     function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
463:         uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;
```

## [G-12] STATE VARIABLES CAN BE PACKED INTO FEWER STORAGE SLOTS
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
```js
37:     bool public mintingPaused = false;
38:     
39:     // --- Data for EIP2612 ---
40:     
41:     // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
42:     bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
43:     // keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");
44:     bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;
45: 
46:     // Cache the domain separator as an immutable value, but also store the chain id that it corresponds to, in order to
47:     // invalidate the cached domain separator if the chain id changes.
48:     bytes32 private immutable _CACHED_DOMAIN_SEPARATOR;
49:     uint256 private immutable _CACHED_CHAIN_ID;
50: 
51:     bytes32 private immutable _HASHED_NAME;
52:     bytes32 private immutable _HASHED_VERSION;
53:     
54:     mapping (address => uint256) private _nonces;
55:     
56:     // User data for LUSD token
57:     mapping (address => uint256) private _balances;
58:     mapping (address => mapping (address => uint256)) private _allowances;  
59:     
60:     // --- Addresses ---
61:     // mappings store addresses of old versions so they can still burn (close troves)
62:     mapping (address => bool) public troveManagers;
63:     mapping (address => bool) public stabilityPools;
64:     mapping (address => bool) public borrowerOperations;
65:     // simple address variables track current version that can mint (in addition to burning)
66:     // this design makes it so that only the latest version can open troves
67:     address public troveManagerAddress;
68:     address public stabilityPoolAddress;
69:     address public borrowerOperationsAddress;
70:     address public governanceAddress; // can pause/unpause minting and upgrade addresses
71:     address public guardianAddress; // can pause minting during emergency
```
```bool mintingPaused``` can be moved down before line 67

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
38:     address[] public withdrawalQueue;
39: 
40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
41:     uint256 public constant PERCENT_DIVISOR = 10000;
42:     uint256 public tvlCap;
43: 
44:     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
45:     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
46:     uint256 public lastReport; // block.timestamp of last report from any strategy
47: 
48:     uint256 public immutable constructionTime;
49:     bool public emergencyShutdown;
```

```bool public emergencyShutdown``` can be moved up before line 38

## [G-13] ```require()``` OR ```revert()``` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION
By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
```js
93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

127:         require(_bps <= 10_000, "Invalid BPS value");
```

[Ethos-Core\contracts\CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
```js
66:             require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

69:             require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

94:         require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

97:         require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
```

## [G-14] SPLITTING ```require()``` STATEMENTS THAT USE && SAVES GAS 
There is a larger deployment gas cost when doing this, but with enough runtime calls, the change ends up being cheaper by 3 gas.

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol):
```js
301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0)); 

653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
```

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol):
```js
347:         require(
348:             _recipient != address(0) && 
349:             _recipient != address(this),
350:             "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:         );
352:         require(
353:             !stabilityPools[_recipient] &&
354:             !troveManagers[_recipient] &&
355:             !borrowerOperations[_recipient],
356:             "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:         );
```

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
```js
1281:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1344:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1541:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

## [G-15] EXPRESSION CAN BE UNCHECKED WHEN OVERFLOW/UNDERFLOW IS NOT POSSIBLE
[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
```js
156:         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
```

## [G-16] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.
[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
```js
103:         OathToken.transferFrom(msg.sender, address(this), amount);

127:         OathToken.transfer(_account, _OathAmount);
```