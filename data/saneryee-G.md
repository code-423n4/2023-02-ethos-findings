# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |    10     |
| [G-002] | Splitting require() statements that use `&&` saves gas                                                                             |     3     |
| [G-003] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |    10     |
| [G-004] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |     9     |
| [G-005] | Using `calldata` instead of memory for read-only arguments in external functions saves gas                                         |     1     |
| [G-006] | internal functions only called once can be inlined to save gas                                                                     |    46     |
| [G-007] | Use a more recent version of solidity                                                                                              |     8     |
| [G-008] | Use elementary types or a user-defined type instead of a struct that has only one member                                           |     1     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:10

[Ethos-Vault/contracts/ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L168)

```solidity
168:    totalAllocBPS += _allocBPS;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L196)

```solidity
196:    totalAllocBPS += _allocBPS;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L391](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L391)

```solidity
391:    totalLoss += loss;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L452](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L452)

```solidity
452:    totalAllocated -= loss;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L396)

```solidity
396:    totalAllocated -= actualWithdrawn;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L445](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L445)

```solidity
445:    totalAllocBPS -= bpsChange;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L521](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L521)

```solidity
521:    totalAllocated += vars.credit;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L515](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L515)

```solidity
515:    totalAllocated -= vars.debtPayment;
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L390](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L390)

```solidity
390:    value -= loss;
```

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133)

```solidity
133:    toFree += profit;
```

## [G-002] Splitting require() statements that use `&&` saves gas

### Impact

When using `&&` in a `require()` statement, the entire statement will be evaluated before the transaction reverts. If the first condition is false, the second condition will not be evaluated. This means that if the first condition is false, the second condition will not be evaluated, which is a waste of gas. Splitting the `require()` statement into two separate statements will save gas.

### Findings

Total:3

[Ethos-Core/contracts/BorrowerOperations.sol#L653-L654](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)

```solidity
653:    require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:                    "Max fee percentage must be between 0.5% and 100%");
```

[Ethos-Core/contracts/LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L352-L357)

```solidity
352:    require(
353:                !stabilityPools[_recipient] &&
354:                !troveManagers[_recipient] &&
355:                !borrowerOperations[_recipient],
356:                "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:            );
```

[Ethos-Core/contracts/LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L347-L351)

```solidity
347:    require(
348:                _recipient != address(0) &&
349:                _recipient != address(this),
350:                "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:            );
```

## [G-003] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:10

[Ethos-Core/contracts/TroveManager.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L313)

```solidity
313:    address[] memory borrowers = new address[](1);
```

[Ethos-Core/contracts/ActivePool.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L105)

```solidity
105:    address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
```

[Ethos-Core/contracts/StabilityPool.sol#L807](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L807)

```solidity
807:    uint[] memory amounts = new uint[](collaterals.length);
```

[Ethos-Core/contracts/StabilityPool.sol#L806](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L806)

```solidity
806:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

[Ethos-Core/contracts/StabilityPool.sol#L857](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L857)

```solidity
857:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L227)

```solidity
227:    uint[] memory amounts = new uint[](collaterals.length);
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L226](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L226)

```solidity
226:    address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L660](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L660)

```solidity
660:    bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```

[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L204](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L204)

```solidity
204:    bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166)

```solidity
166:    address[2] memory step = _newSteps[i];
```

### Recommendation

Using `storage` instead of `memory`

## [G-004] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:9

[Ethos-Core/contracts/CollateralConfig.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/CollateralConfig.sol#L50)

```solidity
50:    ) external override onlyOwner {
```

[Ethos-Core/contracts/CollateralConfig.sol#L89](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/CollateralConfig.sol#L89)

```solidity
89:    ) external onlyOwner checkCollateral(_collateral) {
```

[Ethos-Core/contracts/ActivePool.sol#L144](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L144)

```solidity
144:    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
```

[Ethos-Core/contracts/ActivePool.sol#L138](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L138)

```solidity
138:    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
```

[Ethos-Core/contracts/ActivePool.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L125)

```solidity
125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
```

[Ethos-Core/contracts/ActivePool.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L214)

```solidity
214:    function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
```

[Ethos-Core/contracts/ActivePool.sol#L132](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L132)

```solidity
132:    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
```

[Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101)

```solidity
101:    function fund(uint amount) external onlyOwner {
```

[Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120)

```solidity
120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
```

### Recommendation

Mark the function as payable.

## [G-005] Using `calldata` instead of memory for read-only arguments in external functions saves gas

### Impact

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a `for-loop` to copy each index of the `calldata` to the `memory` index. Each iteration of this `for-loop` costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead.<br><br> If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

### Findings

Total:1

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L65](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L65)

```solidity
62:    function initialize(
63:            address _vault,
64:            address[] memory _strategists,
65:            address[] memory _multisigRoles,
```

## [G-006] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:46

[Ethos-Core/contracts/BorrowerOperations.sol#L537](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L537)

```solidity
537:    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L524](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L524)

```solidity
524:    function _requireValidCollateralAddress(address _collateral) internal view {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L636](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L636)

```solidity
636:    function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L624](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L624)

```solidity
624:    function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L567](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L567)

```solidity
567:    function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L533](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L533)

```solidity
533:    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L551)

```solidity
551:    function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L546](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L546)

```solidity
546:    function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
```

[Ethos-Core/contracts/BorrowerOperations.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L640)

```solidity
640:    function _requireCallerIsStabilityPool() internal view {
```

[Ethos-Core/contracts/TroveManager.sol#L1538](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1538)

```solidity
1538:    function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
```

[Ethos-Core/contracts/TroveManager.sol#L1082](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1082)

```solidity
1082:    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
```

[Ethos-Core/contracts/TroveManager.sol#L1516](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1516)

```solidity
1516:    function _minutesPassedSinceLastFeeOp() internal view returns (uint) {
```

[Ethos-Core/contracts/TroveManager.sol#L1213](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1213)

```solidity
1213:    function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
```

[Ethos-Core/contracts/TroveManager.sol#L1321](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1321)

```solidity
1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
```

[Ethos-Core/contracts/TroveManager.sol#L1339](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L1339)

```solidity
1339:    function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
```

[Ethos-Core/contracts/ActivePool.sol#L337](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L337)

```solidity
337:    function _requireCallerIsBOorTroveM() internal view {
```

[Ethos-Core/contracts/ActivePool.sol#L320](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L320)

```solidity
320:    function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
```

[Ethos-Core/contracts/StabilityPool.sol#L856](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L856)

```solidity
856:    function _requireNoUnderCollateralizedTroves() internal {
```

[Ethos-Core/contracts/StabilityPool.sol#L848](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L848)

```solidity
848:    function _requireCallerIsActivePool() internal view {
```

[Ethos-Core/contracts/StabilityPool.sol#L597](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L597)

```solidity
597:    function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {
```

[Ethos-Core/contracts/StabilityPool.sol#L852](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L852)

```solidity
852:    function _requireCallerIsTroveManager() internal view {
```

[Ethos-Core/contracts/StabilityPool.sol#L873](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L873)

```solidity
873:    function _requireNonZeroAmount(uint _amount) internal pure {
```

[Ethos-Core/contracts/StabilityPool.sol#L693](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L693)

```solidity
693:    function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
```

[Ethos-Core/contracts/StabilityPool.sol#L776](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L776)

```solidity
776:    function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {
```

[Ethos-Core/contracts/StabilityPool.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L421)

```solidity
421:    function _updateG(uint _LQTYIssuance) internal {
```

[Ethos-Core/contracts/StabilityPool.sol#L794](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L794)

```solidity
794:    function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {
```

[Ethos-Core/contracts/StabilityPool.sol#L657](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L657)

```solidity
657:    function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {
```

[Ethos-Core/contracts/StabilityPool.sol#L439](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L439)

```solidity
439:    function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {
```

[Ethos-Core/contracts/StabilityPool.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L637)

```solidity
637:    function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
```

[Ethos-Core/contracts/StabilityPool.sol#L869](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L869)

```solidity
869:    function _requireUserHasDeposit(uint _initialDeposit) internal pure {
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L269](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L269)

```solidity
269:    function _requireNonZeroAmount(uint _amount) internal pure {
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251)

```solidity
251:    function _requireCallerIsTroveManagerOrActivePool() internal view {
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265)

```solidity
265:    function _requireUserHasStake(uint currentStake) internal pure {
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L261](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L261)

```solidity
261:    function _requireCallerIsBorrowerOperations() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L328](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L328)

```solidity
328:    function _burn(address account, uint256 amount) internal {
```

[Ethos-Core/contracts/LUSDToken.sol#L380](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L380)

```solidity
380:    function _requireCallerIsTroveMorSP() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L360](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L360)

```solidity
360:    function _requireCallerIsBorrowerOperations() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L375](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L375)

```solidity
375:    function _requireCallerIsStabilityPool() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L393](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L393)

```solidity
393:    function _requireMintingNotPaused() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L365](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L365)

```solidity
365:    function _requireCallerIsBOorTroveMorSP() internal view {
```

[Ethos-Core/contracts/LUSDToken.sol#L320](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L320)

```solidity
320:    function _mint(address account, uint256 amount) internal {
```

[Ethos-Vault/contracts/ReaperVaultV2.sol#L462](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperVaultV2.sol#L462)

```solidity
462:    function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
```

[Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L250](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol#L250)

```solidity
250:    function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);
```

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176)

```solidity
176:    function _deposit(uint256 toReinvest) internal {
```

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L187](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L187)

```solidity
187:    function _withdraw(uint256 _amount) internal {
```

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206)

```solidity
206:    function _claimRewards() internal {
```

## [G-007] Use a more recent version of solidity

### Impact

- Use a solidity version of at least `0.8.2` to get simple compiler automatic inlining
- Use a solidity version of at least `0.8.3` to get better struct packing and cheaper multiple storage reads
- Use a solidity version of at least `0.8.4` to get custom errors, which are cheaper at deployment than `revert()/require()` strings
- Use a solidity version of at least `0.8.10` to have external calls skip contract existence checks if the external call has a return value

### Findings

Total:8

[Ethos-Core/contracts/CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/CollateralConfig.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/TroveManager.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/ActivePool.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

[Ethos-Core/contracts/LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L3)

```solidity
3:    pragma solidity 0.6.11;
```

### Recommendation

Current version: 0.8.17 (2022-11)

## [G-008] Use elementary types or a user-defined type instead of a struct that has only one member

### Impact

Use elementary types or a user-defined type instead of a struct that has only one member.

### Findings

Total:1

[Ethos-Core/contracts/StabilityPool.sol#L174-L176](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L174-L176)

```solidity
174:    struct Deposit {
175:            uint initialValue;
176:        }
```
