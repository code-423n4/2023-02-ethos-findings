## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// struct coming after functions, should come before.
646:    struct LocalVariables_getSingularCollateralGain {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol

```solidity
// struct coming after a function, should come before them.
70:    struct LocalVariables_getRedemptionHints {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```solidity
// modifier coming after all the functions
128:    modifier checkCollateral(address _collateral) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
// struct coming after functions, should come before.
220:    struct LocalVariables_rebalance {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// struct coming after functions, should come before.
643:    struct LocalVariables_getSingularCollateralGain {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```solidity
// internal function coming before external one
203:    function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```solidity
  External and Public functions are all mixed.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```solidity
// internal function coming before some external ones
282:    function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

// external function coming after all the internal ones
748:    function getCompositeDebt(uint _debt) external pure override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
  The whole file has mixed visibility functions, the dev should consider ordering them acording to specifications.
```

---

### natSpec missing [3]

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/TroveManagerScript.sol

```solidity
9:    contract TroveManagerScript is CheckContract {

14:    constructor(ITroveManager _troveManager) public {

19:    function redeemCollateral(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/TokenScript.sol

```solidity
9:    contract TokenScript is CheckContract {

14:    constructor(address _tokenAddress) public {

19:    function transfer(address recipient, uint256 amount) external returns (bool) {

23:    function allowance(address owner, address spender) external view returns (uint256) {

27:    function approve(address spender, uint256 amount) external returns (bool) {

31:    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool) {

35:    function increaseAllowance(address spender, uint256 addedValue) external returns (bool) {

39:    function decreaseAllowance(address spender, uint256 subtractedValue) external returns (bool) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/StabilityPoolScript.sol

```solidity
9:    contract StabilityPoolScript is CheckContract {

14:    constructor(IStabilityPool _stabilityPool) public {

19:    function provideToSP(uint _amount) external {

23:    function withdrawFromSP(uint _amount) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/LQTYStakingScript.sol

```solidity
9:    contract LQTYStakingScript is CheckContract {

12:    constructor(address _lqtyStakingAddress) public {

17:    function stake(uint _LQTYamount) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/ERC20TransferScript.sol

```solidity
7:    contract ERC20TransferScript {

8:    function transferERC20(address _token, address _recipient, uint256 _amount) external returns (bool) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol

```solidity
21: contract BorrowerWrappersScript is BorrowerOperationsScript, ERC20TransferScript, LQTYStakingScript {

35:    constructor(

74:    function claimCollateralAndOpenTrove(address _collateral, uint _collAmount, uint _maxFee, uint _LUSDAmount, address _upperHint, address _lowerHint) external {

93:    function claimSPRewardsAndRecycle(address _collateral, uint _maxFee, address _upperHint, address _lowerHint) external {

124:    function claimStakingGainsAndRecycle(address _collateral, uint _maxFee, address _upperHint, address _lowerHint) external {

159:    function _getNetLUSDAmount(address _collateral, uint _collAmount) internal returns (uint) {

171:    function _requireUserHasTrove(address _depositor, address _collateral) internal view {

175:    function _getScaledCollAmount(uint256 _collAmount, uint256 _collDecimals) internal pure returns (uint256 scaledColl) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerOperationsScript.sol

```solidity
10:   contract BorrowerOperationsScript is CheckContract {

15:    constructor(IBorrowerOperations _borrowerOperations) public {

20:    function openTrove(address _collateral, uint _collAmount, uint _maxFee, uint _LUSDAmount, address _upperHint, address _lowerHint) external {

26:    function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external {

32:    function withdrawColl(address _collateral, uint _amount, address _upperHint, address _lowerHint) external {

36:    function withdrawLUSD(address _collateral, uint _maxFee, uint _amount, address _upperHint, address _lowerHint) external {

40:    function repayLUSD(address _collateral, uint _amount, address _upperHint, address _lowerHint) external {

44:    function closeTrove(address _collateral) external {

48:    function adjustTrove(address _collateral, uint _maxFee, uint _collTopUp, uint _collWithdrawal, uint _debtChange, bool isDebtIncrease, address _upperHint, address _lowerHint) external {

58:    function claimCollateral(address _collateral) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```solidity
18:   contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {

66:    function setAddresses

104:    function stake(uint _LQTYamount) external override {

142:    function unstake(uint _LQTYamount) external override {

177:    function increaseF_Collateral(address _collateral, uint _collFee) external override {

187:    function increaseF_LUSD(uint _LUSDFee) external override {

199:    function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory) {

203:    function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {

213:    function getPendingLUSDGain(address _user) external view override returns (uint) {

217:    function _getPendingLUSDGain(address _user) internal view returns (uint) {

225:    function _updateUserSnapshots(address _user) internal {

238:    function _sendCollGainToUser(address[] memory assets, uint[] memory amounts) internal {

251:    function _requireCallerIsTroveManagerOrActivePool() internal view {

261:    function _requireCallerIsBorrowerOperations() internal view {

265:    function _requireUserHasStake(uint currentStake) internal pure {  

269:    function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```solidity
14:  contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

61:    function setAddresses

// @return missing
84:    function issueOath() external override returns (uint issuance) {

120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

124:    function sendOath(address _account, uint _OathAmount) external override {

132:    function _requireCallerIsStabilityPool() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
25:    struct StrategyParams {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```solidity
16:    constructor(

29:    function asset() external view override returns (address assetTokenAddress) {

37:    function totalAssets() external view override returns (uint256 totalManagedAssets) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```solidity
// natspec incomplete
62:    function initialize(

74:    function _adjustPosition(uint256 _debt) internal override {

86:    function _liquidatePosition(uint256 _amountNeeded)

104:    function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

// incomplete
114:    function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
147:    function updateVeloSwapPath(
160:    function setHarvestSteps(address[2 ][] calldata _newSteps) external {
176:    function _deposit(uint256 toReinvest) internal {
187:    function _withdraw(uint256 _amount) internal {
197:    function _withdrawUnderlying(uint256 _withdrawAmount) internal {
206:    function _claimRewards() internal {
213:    function authorizedWithdrawUnderlying(uint256 _amount) external {
222:    function balanceOf() public view override returns (uint256) {

226:    function balanceOfWant() public view returns (uint256) {

230:    function balanceOfPool() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```solidity
68:    enum Status {

77:    struct Trove {

129:    struct LocalVariables_OuterLiquidationFunction {

140:    struct LocalVariables_InnerSingleLiquidateFunction {

146:    struct LocalVariables_LiquidationSequence {

160:    struct LiquidationValues {

172:    struct LiquidationTotals {

210:    event Redemption(

218:     enum TroveManagerOperation {

232:    function setAddresses(

299:    function getTroveOwnersCount(address _collateral) external view override returns (uint) {

303:    function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {

310:    function liquidate(address _borrower, address _collateral) external override {

321:    function _liquidateNormalMode(

357:    function _liquidateRecoveryMode(

442:    function _getOffsetAndRedistributionVals

478:    function _getCappedOffsetVals

513:    function liquidateTroves(address _collateral, uint _n) external override {

582:    function _getTotalsFromLiquidateTrovesSequence_RecoveryMode

671:    function _getTotalsFromLiquidateTrovesSequence_NormalMode

715:    function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {

785:    function _getTotalFromBatchLiquidate_RecoveryMode

864:    function _getTotalsFromBatchLiquidate_NormalMode

898:    function _addLiquidationValuesToTotals(LiquidationTotals memory oldTotals, LiquidationValues memory singleLiquidation)

915:    function _sendGasCompensation(IActivePool _activePool, address _collateral, address _liquidator, uint _LUSD, uint _collAmount) internal {

926:    function _movePendingTroveRewardsToActivePool(IActivePool _activePool, IDefaultPool _defaultPool, address _collateral, uint _LUSD, uint _collAmount) internal {

957:    function reInsert(address _id, address _collateral, uint256 _newNICR, address _prevId, address _nextId) external override {

962:    function updateDebtAndCollAndStakesPostRedemption(

982:    function burnLUSDAndEmitRedemptionEvent(

1045:    function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1054:    function getCurrentICR(

1066:    function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {

1076:    function applyPendingRewards(address _borrower, address _collateral) external override {

1082:    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

1111:    function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {

1116:    function _updateTroveRewardSnapshots(address _borrower, address _collateral) internal {

1123:    function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1138:    function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1152:    function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {

1164:    function getEntireDebtAndColl(

1183:    function removeStake(address _borrower, address _collateral) external override {

1189:    function _removeStake(address _borrower, address _collateral) internal {

1195:    function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {

1201:    function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {

1213:    function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1230:    function _redistributeDebtAndColl(

1273:    function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {

1278:    function _closeTrove(address _borrower, address _collateral, Status closedStatus) internal {

1316:    function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {

1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

1339:    function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {

1361:    function getTCR(address _collateral, uint _price) external view override returns (uint) {

1366:    function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {

1373:    function _checkPotentialRecoveryMode(

1397:    function updateBaseRateFromRedemption(

1425:    function getRedemptionRate() public view override returns (uint) {

1429:    function getRedemptionRateWithDecay() public view override returns (uint) {

1433:    function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {

1440:    function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1444:    function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1448:    function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {

1456:    function getBorrowingRate() public view override returns (uint) {

1460:    function getBorrowingRateWithDecay() public view override returns (uint) {

1464:    function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {

1471:    function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {

1475:    function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {

1479:    function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {

1485:    function decayBaseRateFromBorrowing() external override {

1500:    function _updateLastFeeOpTime() internal {

1509:    function _calcDecayedBaseRate() internal view returns (uint) {

1516:    function _minutesPassedSinceLastFeeOp() internal view returns (uint) {

1522:    function _requireCallerIsBorrowerOperations() internal view {

1526:    function _requireCallerIsRedemptionHelper() internal view {

1530:    function _requireCallerIsBorrowerOperationsOrRedemptionHelper() internal view {

1534:    function _requireTroveIsActive(address _borrower, address _collateral) internal view {

1538:    function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {

1544:    function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {

1548:    function getTroveStake(address _borrower, address _collateral) external view override returns (uint) {

1552:    function getTroveDebt(address _borrower, address _collateral) external view override returns (uint) {

1556:    function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {

1562:    function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

1567:    function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {

1574:    function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

1581:    function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {

1588:    function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {
```

https://    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
261:    function setAddresses(

307:    function getCollateral(address _collateral) external view override returns (uint) {

311:    function getTotalLUSDDeposits() external view override returns (uint) {

416:    function _triggerLQTYIssuance(ICommunityIssuance _communityIssuance) internal {

421:    function _updateG(uint _LQTYIssuance) internal {

439:    function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

504:    function _computeRewardsPerUnitStaked(

547:    function _updateRewardSumAndProduct(address _collateral, uint _collGainPerUnitStaked, uint _LUSDLossPerUnitStaked) internal {

597:    function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

613:    function _decreaseLUSD(uint _amount) internal {

637:    function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

646:    struct LocalVariables_getSingularCollateralGain {

657:    function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {

693:    function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

729:    function _getCompoundedDepositFromSnapshots(

776:    function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

783:    function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {

794:    function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

803:    function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {

840:    function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {

848:    function _requireCallerIsActivePool() internal view {

852:    function _requireCallerIsTroveManager() internal view {

856:    function _requireNoUnderCollateralizedTroves() internal {

869:    function _requireUserHasDeposit(uint _initialDeposit) internal pure {

873:    function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol

```solidity
80:    function setParams(address _troveManagerAddress, address _borrowerOperationsAddress) external override onlyOwner {

109:    function _insert(ITroveManager _troveManager, address _collateral, address _id, uint256 _NICR, address _prevId, address _nextId) internal {

154:    function remove(address _collateral, address _id) external override {

// natspec incomplete
227:    function contains(address _collateral, address _id) public view override returns (bool) {
234:    function isEmpty(address _collateral) public view override returns (bool) {
241:    function getSize(address _collateral) external view override returns (uint256) {
248:    function getFirst(address _collateral) external view override returns (address) {
255:    function getLast(address _collateral) external view override returns (address) {

288:    function _validInsertPosition(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _prevId, address _nextId) internal view returns (bool) {

367:    function _findInsertPosition(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _prevId, address _nextId) internal view returns (address, address) {

402:    function _requireCallerIsTroveManager() internal view {

406:    function _requireCallerIsBOorTroveM(ITroveManager _troveManager) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol

```solidity
26:    struct RedemptionTotals {

40:    struct SingleRedemptionValues {

47:    struct LocalVariables_redeemCollateralFromTrove {

54:    function setAddresses(

216:    function _isValidFirstRedemptionHint(

235:    function _redeemCollateralFromTrove(

295:    function _requireCallerIsTroveManager() internal view {

299:    function _requireValidCollateralAddress(address _collateral) internal view {

303:    function _requireValidMaxFeePercentage(uint _maxFeePercentage) internal view {

307:    function _requireAfterBootstrapPeriod() internal view {

312:    function _requireTCRoverMCR(address _collateral, uint _price, uint256 _collDecimals, uint256 _MCR) internal view {

316:    function _requireAmountGreaterThanZero(uint _amount) internal pure {

320:    function _requireLUSDBalanceCoversRedemption(ILUSDToken _lusdToken, address _redeemer, uint _amount) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol

```solidity
90:    function setAddresses(

143:    function updateChainlinkAggregator(

167:    function updateTellorCaller(

424:    function _chainlinkIsFrozen(ChainlinkResponse memory _response) internal view returns (bool) {

428:    function _chainlinkPriceChangeAboveMax(ChainlinkResponse memory _currentResponse, ChainlinkResponse memory _prevResponse) internal pure returns (bool) {

446:    function _tellorIsBroken(TellorResponse memory _response) internal view returns (bool) {

457:     function _tellorIsFrozen(TellorResponse  memory _tellorResponse) internal view returns (bool) {

461:    function _bothOraclesLiveAndUnbrokenAndSimilarPrice

486:    function _bothOraclesSimilarPrice( ChainlinkResponse memory _chainlinkResponse, TellorResponse memory _tellorResponse) internal pure returns (bool) {

502:    function _scaleChainlinkPriceByDigits(uint _price, uint _answerDigits) internal pure returns (uint) {

521:    function _scaleTellorPriceByDigits(uint _price) internal pure returns (uint) {

531:    function _changeStatus(address _collateral, Status _status) internal {

536:    function _storePrice(address _collateral, uint _currentPrice) internal {

541:     function _storeTellorPrice(address _collateral, TellorResponse memory _tellorResponse) internal returns (uint) {

548:    function _storeChainlinkPrice(address _collateral, ChainlinkResponse memory _chainlinkResponse) internal returns (uint) {

557:    function _getCurrentTellorResponse(address _collateral) internal view returns (TellorResponse memory tellorResponse) {

578:    function _getCurrentChainlinkResponse(address _collateral) internal view returns (ChainlinkResponse memory chainlinkResponse) {

610:    function _getPrevChainlinkResponse(address _collateral, uint80 _currentRoundId, uint8 _currentDecimals) internal view returns (ChainlinkResponse memory prevChainlinkResponse) {

641:    function _requireValidCollateralAddress(address _collateral) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol

```solidity
58:    struct ChainlinkResponse {

66:    struct TellorResponse {

73:    enum Status {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol

```solidity
11:  contract MultiTroveGetter {

12:    struct CombinedTroveData {

27:    constructor(ICollateralConfig _collateralConfig, TroveManager _troveManager, ISortedTroves _sortedTroves) public {

33:    function getMultipleSortedTroves(address _collateral, int _startIdx, uint _count)

67:    function _getMultipleSortedTrovesFromHead(address _collateral, uint _startIdx, uint _count)

96:    function _getMultipleSortedTrovesFromTail(address _collateral, uint _startIdx, uint _count)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Migrations.sol

```solidity
17:  function setCompleted(uint completed) public restricted {

21:  function upgrade(address new_address) public restricted {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
84:    constructor

133:    function pauseMinting() external {

141:    function unpauseMinting() external {

146:    function updateGovernance(address _newGovernanceAddress) external {

153:    function updateGuardian(address _newGuardianAddress) external {

160:    function upgradeProtocol(

185:    function mint(address _account, uint256 _amount) external override {

191:    function burn(address _account, uint256 _amount) external override {

196:    function sendToPool(address _sender,  address _poolAddress, uint256 _amount) external override {

201:    function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override {

206:    function getDeploymentStartTime() external view override returns (uint256) {

212:    function totalSupply() external view override returns (uint256) {

216:    function balanceOf(address account) external view override returns (uint256) {

220:    function transfer(address recipient, uint256 amount) external override returns (bool) {

226:    function allowance(address owner, address spender) external view override returns (uint256) {

230:    function approve(address spender, uint256 amount) external override returns (bool) {

235:    function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {

242:    function increaseAllowance(address spender, uint256 addedValue) external override returns (bool) {

247:    function decreaseAllowance(address spender, uint256 subtractedValue) external override returns (bool) {

254:    function domainSeparator() public view override returns (bytes32) {    

262:    function permit

292:    function nonces(address owner) external view override returns (uint256) { // FOR EIP 2612

298:    function _chainID() private pure returns (uint256 chainID) {

304:    function _buildDomainSeparator(bytes32 typeHash, bytes32 name, bytes32 version) private view returns (bytes32) {

311:    function _transfer(address sender, address recipient, uint256 amount) internal {

320:    function _mint(address account, uint256 amount) internal {

328:    function _burn(address account, uint256 amount) internal {

336:    function _approve(address owner, address spender, uint256 amount) internal {

346:    function _requireValidRecipient(address _recipient) internal view {

360:    function _requireCallerIsBorrowerOperations() internal view {

365:    function _requireCallerIsBOorTroveMorSP() internal view {

375:    function _requireCallerIsStabilityPool() internal view {

380:    function _requireCallerIsTroveMorSP() internal view {

389:    function _requireCallerIsGovernance() internal view {

393:    function _requireMintingNotPaused() internal view {

399:    function name() external view override returns (string memory) {

403:    function symbol() external view override returns (string memory) {

407:    function decimals() external view override returns (uint8) {

411:    function version() external view override returns (string memory) {

415:    function permitTypeHash() external view override returns (bytes32) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol

```solidity
27:    function setAddresses(

84:    function getRedemptionHints(

159:    function getApproxHint(address _collateral, uint _CR, uint _numTrials, uint _inputRandomSeed)

195:    function computeNominalCR(uint _coll, uint _debt, uint8 _collDecimals) external pure returns (uint) {

199:    function computeCR(uint _coll, uint _debt, uint _price, uint8 _collDecimals) external pure returns (uint) {

205:    function _requireValidCollateralAddress(address _collateral) internal view {

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol

```solidity
40:    function setAddresses(

70:    function getCollateral(address _collateral) external view override returns (uint) {

75:    function getLUSDDebt(address _collateral) external view override returns (uint) {

82:    function sendCollateralToActivePool(address _collateral, uint _amount) external override {

94:    function increaseLUSDDebt(address _collateral, uint _amount) external override {

101:    function decreaseLUSDDebt(address _collateral, uint _amount) external override {

108:    function pullCollateralFromActivePool(address _collateral, uint _amount) external override {

119:    function _requireValidCollateralAddress(address _collateral) internal view {

126:    function _requireCallerIsActivePool() internal view {

130:    function _requireCallerIsTroveManager() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```solidity
85:    function updateCollateralRatios(

102:    function getAllowedCollaterals() external override view returns (address[] memory) {

106:    function isCollateralAllowed(address _collateral) external override view returns (bool) {

110:    function getCollateralDecimals(

116:    function getCollateralMCR(

122:    function getCollateralCCR(
```

https://github.com/BronzeDisc/arena/edit/main/QA-ethos-reserve.md

```solidity
41:    function setAddresses(

71:    function getCollateral(address _collateral) external view override returns (uint) {

76:    function getUserCollateral(address _account, address _collateral) external view override returns (uint) {

83:    function accountSurplus(address _account, address _collateral, uint _amount) external override {

93:    function claimColl(address _account, address _collateral) external override {

108:    function pullCollateralFromActivePool(address _collateral, uint _amount) external override {

118:    function _requireValidCollateralAddress(address _collateral) internal view {

125:    function _requireCallerIsBorrowerOperations() internal view {

131:    function _requireCallerIsTroveManager() internal view {

137:    function _requireCallerIsActivePool() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```solidity
47:     struct LocalVariables_adjustTrove {

66:    struct LocalVariables_openTrove {

80:    struct ContractsCache {

86:    enum BorrowerOperation {

110:    function setAddresses(

172:    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

242:    function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

248:    function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

252:    function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override {

259:    function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

264:    function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

268:    function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {

282:    function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

375:    function closeTrove(address _collateral) external override {

412:    function claimCollateral(address _collateral) external override {

419:    function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {

432:    function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

438:    function _getCollChange(

455:    function _updateTroveFromAdjustment

476:    function _moveTokensAndCollateralfromAdjustment

505:    function _activePoolAddColl(IActivePool _activePool, address _collateral, uint _amount) internal {

511:    function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {

517:    function _repayLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSD) internal {

524:    function _requireValidCollateralAddress(address _collateral) internal view {

528:    function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {

533:    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

537:    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

541:    function _requireTroveisActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

546:    function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

551:    function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

555:    function _requireNotInRecoveryMode(

567:    function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {

571:    function _requireValidAdjustmentInCurrentMode 

616:    function _requireICRisAboveMCR(uint _newICR, uint256 _MCR) internal pure {

620:    function _requireICRisAboveMCR(uint _newICR, uint256 _MCR) internal pure {

624:    function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

628:    function _requireNewTCRisAboveCCR(uint _newTCR, uint256 _CCR) internal pure {

632:    function _requireAtLeastMinNetDebt(uint _netDebt) internal pure {

636:    function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

640:    function _requireCallerIsStabilityPool() internal view {

644:     function _requireSufficientLUSDBalance(ILUSDToken _lusdToken, address _borrower, uint _debtRepayment) internal view {

648:    function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {

661:    function _getNewNominalICRFromTroveChange

682:    function _getNewICRFromTroveChange

703:    function _getNewTroveAmounts(

724:    function _getNewTCRFromTroveChange

748:    function getCompositeDebt(uint _debt) external pure override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
261:    function setAddresses(

307:    function getCollateral(address _collateral) external view override returns (uint) {

311:    function getTotalLUSDDeposits() external view override returns (uint) {

// add right natspec
323:    function provideToSP(uint _amount) external override {

// add right natspec
367:    function withdrawFromSP(uint _amount) external override {

410:    function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {

416:    function _triggerLQTYIssuance(ICommunityIssuance _communityIssuance) internal {

421:    function _updateG(uint _LQTYIssuance) internal {

439:    function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

466:    function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {

486:    function updateRewardSum(address _collateral, uint _collToAdd) external override {

504:    function _computeRewardsPerUnitStaked(

547:    function _updateRewardSumAndProduct(address _collateral, uint _collGainPerUnitStaked, uint _LUSDLossPerUnitStaked) internal {

597:    function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

613:    function _decreaseLUSD(uint _amount) internal {

626:    function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {

637:    function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

657:    function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {

683:    function getDepositorLQTYGain(address _depositor) public view override returns (uint) {

693:    function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

718:    function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {

729:    function _getCompoundedDepositFromSnapshots(

776:    function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

783:    function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {

794:    function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

803:    function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {

840:    function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {

848:    function _requireCallerIsActivePool() internal view {

852:    function _requireCallerIsTroveManager() internal view {

856:    function _requireNoUnderCollateralizedTroves() internal {

869:    function _requireUserHasDeposit(uint _initialDeposit) internal pure {

873:    function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
71:    function setAddresses(

125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

159:    function getCollateral(address _collateral) external view override returns (uint) {

164:    function getLUSDDebt(address _collateral) external view override returns (uint) {

171:    function sendCollateral(address _collateral, address _account, uint _amount) external override {

190:    function increaseLUSDDebt(address _collateral, uint _amount) external override {

197:    function decreaseLUSDDebt(address _collateral, uint _amount) external override {

204:    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

214:    function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

239:    function _rebalance(address _collateral, uint256 _amountLeavingPool) internal {

313:    function _requireValidCollateralAddress(address _collateral) internal view {

320:    function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {

327:    function _requireCallerIsBOorTroveMorSP() internal view {

337:    function _requireCallerIsBOorTroveM() internal view {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// private and internal `variables` should preppend with `underline`
167:    mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
170:    uint256 internal totalLUSDDeposits;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
// private and internal `variables` should preppend with `underline`
75:    uint internal immutable deploymentStartTime;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol

```solidity
// private and internal `variables` should preppend with `underline`
30:    mapping (address => uint256) internal collAmount;  // collateral => amount tracker
31:    mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol

```solidity
// private and internal `variables` should preppend with `underline`
25:    mapping (address => uint256) internal collAmount;
27:    mapping (address => mapping (address => uint)) internal balances;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
// private and internal `variables` should be preppended with `underline`
41:    mapping (address => uint256) internal collAmount;  
42:    mapping (address => uint256) internal LUSDDebt;  
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// private and internal `variables` should be preppended with `underline`
167:    mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
170:    uint256 internal totalLUSDDeposits;
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/TroveManagerScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/TokenScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/StabilityPoolScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/LQTYStakingScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/ERC20TransferScript.sol

```solidity
// consider updating to a newer version
3:    pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerWrappersScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Proxy/BorrowerOperationsScript.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```solidity
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```solidity
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol

```solidity
// consider updating to a newer version
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Migrations.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/GasPool.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
// consider updating to a newer version
3: pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```solidity
// consider updating to a newer version
3:  pragma solidity 0.6.11;
```


---

### Initialized variables [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```solidity
// redundant and also more gas costly, remove the `false`.
21:    bool public initialized = false;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```solidity
// redundant and also more gas costly, remove the `0`.
35:    uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol

```solidity
// redundant and also more gas costly, remove the `false`.
29:    bool public initialized = false;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
// redundant and also more gas costly, remove the `false`.
37:    bool public mintingPaused = false;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```solidity
// redundant and also more gas costly, remove the `false`.
18:    bool public initialized = false;
```


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
// redundant and also more gas costly, remove the `false`.
32:     bool public addressesSet = false;
```

---

### Observations [7]

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration
49:    event LQTYTokenAddressSet(address _lqtyTokenAddress);
50:    event LUSDTokenAddressSet(address _lusdTokenAddress);
51:    event TroveManagerAddressSet(address _troveManager);
52:    event BorrowerOperationsAddressSet(address _borrowerOperationsAddress);
53:    event ActivePoolAddressSet(address _activePoolAddress);
54:    event CollateralConfigAddressSet(address _collateralConfigAddress);
56:    event StakeChanged(address indexed staker, uint newStake);
57:    event StakingGainsWithdrawn(address indexed staker, uint LUSDGain, address[] _assets, uint[] _amounts);
58:    event F_CollateralUpdated(address _collateral, uint _F_Collateral);
59:    event F_LUSDUpdated(uint _F_LUSD);
60:    event TotalLQTYStakedUpdated(uint _totalLQTYStaked);
61:    event CollateralSent(address _account, address _collateral, uint _amount);
62:    event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration
50:    event OathTokenAddressSet(address _oathTokenAddress);
51:    event LogRewardPerSecond(uint _rewardPerSecond);
52:    event StabilityPoolAddressSet(address _stabilityPoolAddress);
53:    event TotalOATHIssuedUpdated(uint _totalOATHIssued);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration
80:    event StrategyAdded(address indexed strategy, uint256 feeBPS, uint256 allocBPS);
81:    event StrategyFeeBPSUpdated(address indexed strategy, uint256 feeBPS);
82:    event StrategyAllocBPSUpdated(address indexed strategy, uint256 allocBPS);
83:    event StrategyRevoked(address indexed strategy);
84:    event UpdateWithdrawalQueue(address[] withdrawalQueue);
85:    event WithdrawMaxLossUpdated(uint256 withdrawMaxLoss);
86:    event EmergencyShutdown(bool active);
87:    event InCaseTokensGetStuckCalled(address token, uint256 amount);
88:    event TvlCapUpdated(uint256 newTvlCap);
89:    event LockedProfitDegradationUpdated(uint256 degradation);
90:    event StrategyReported(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
186:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
187:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
188:    event PriceFeedAddressChanged(address _newPriceFeedAddress);
189:    event LUSDTokenAddressChanged(address _newLUSDTokenAddress);
190:    event ActivePoolAddressChanged(address _activePoolAddress);
191:    event DefaultPoolAddressChanged(address _defaultPoolAddress);
192:    event StabilityPoolAddressChanged(address _stabilityPoolAddress);
193:    event GasPoolAddressChanged(address _gasPoolAddress);
194:    event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
195:    event SortedTrovesAddressChanged(address _sortedTrovesAddress);
196:    event LQTYTokenAddressChanged(address _lqtyTokenAddress);
197:    event LQTYStakingAddressChanged(address _lqtyStakingAddress);
198:    event RedemptionHelperAddressChanged(address _redemptionHelperAddress);
200:    event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);
201:    event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint _stake, TroveManagerOperation _operation);
202:    event TroveLiquidated(address indexed _borrower, address _collateral, uint _debt, uint _coll, TroveManagerOperation _operation);
203:    event BaseRateUpdated(uint _baseRate);
204:    event LastFeeOpTimeUpdated(uint _lastFeeOpTime);
205:    event TotalStakesUpdated(address _collateral, uint _newTotalStakes);
206:    event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);
207:    event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
208:    event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
209:    event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);
210:    event Redemption(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
233:    event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);
234:    event StabilityPoolLUSDBalanceUpdated(uint _newBalance);
236:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
237:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
238:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
239:    event ActivePoolAddressChanged(address _newActivePoolAddress);
240:    event DefaultPoolAddressChanged(address _newDefaultPoolAddress);
241:    event LUSDTokenAddressChanged(address _newLUSDTokenAddress);
242:    event SortedTrovesAddressChanged(address _newSortedTrovesAddress);
243:    event PriceFeedAddressChanged(address _newPriceFeedAddress);
244:    event CommunityIssuanceAddressChanged(address _newCommunityIssuanceAddress);
246:    event P_Updated(uint _P);
247:    event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);
248:    event G_Updated(uint _G, uint128 _epoch, uint128 _scale);
249:    event EpochUpdated(uint128 _currentEpoch);
250:    event ScaleUpdated(uint128 _currentScale);
257:    event CollateralSent(address _collateral, address _to, uint _amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
51:    event TroveManagerAddressChanged(address _troveManagerAddress);
52:    event BorrowerOperationsAddressChanged(address _borrowerOperationsAddress);
53:    event NodeAdded(address _collateral, address _id, uint _NICR);
54:    event NodeRemoved(address _collateral, address _id);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
84:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
85:    event LastGoodPriceUpdated(address _collateral, uint _lastGoodPrice);
86:    event PriceFeedStatusChanged(address _collateral, Status newStatus);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
78:    event TroveManagerAddressChanged(address _troveManagerAddress);
79:    event StabilityPoolAddressChanged(address _newStabilityPoolAddress);
80:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
81:    event GovernanceAddressChanged(address _governanceAddress);
82:    event GuardianAddressChanged(address _guardianAddress);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
21:    event CollateralConfigAddressChanged(address _collateralConfigAddress);
22:    event SortedTrovesAddressChanged(address _sortedTrovesAddress);
23:    event TroveManagerAddressChanged(address _troveManagerAddress);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
33:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
34:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
35:    event DefaultPoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);
36:    event DefaultPoolCollateralBalanceUpdated(address _collateral, uint _amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
37:    event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
38:    event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
31:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
32:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
33:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
34:    event ActivePoolAddressChanged(address _newActivePoolAddress);
36:    event CollBalanceUpdated(address indexed _account, address _collateral, uint _newBalance);
37:    event CollateralSent(address _collateral, address _to, uint _amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
92:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
93:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
94:    event ActivePoolAddressChanged(address _activePoolAddress);
95:    event DefaultPoolAddressChanged(address _defaultPoolAddress);
96:    event StabilityPoolAddressChanged(address _stabilityPoolAddress);
97:    event GasPoolAddressChanged(address _gasPoolAddress);
98:    event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
99:    event PriceFeedAddressChanged(address  _newPriceFeedAddress);
100:    event SortedTrovesAddressChanged(address _sortedTrovesAddress);
101:    event LUSDTokenAddressChanged(address _lusdTokenAddress);
102:    event LQTYStakingAddressChanged(address _lqtyStakingAddress);
104:    event TroveCreated(address indexed _borrower, address _collateral, uint arrayIndex);
105:    event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);
106:    event LUSDBorrowingFeePaid(address indexed _borrower, address _collateral, uint _LUSDFee);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
58:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
59:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
60:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
61:    event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
62:    event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);
63:    event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);
64:    event YieldingPercentageUpdated(address _collateral, uint256 _bps);
65:    event YieldingPercentageDriftUpdated(uint256 _driftBps);
66:    event YieldClaimThresholdUpdated(address _collateral, uint256 _threshold);
67:    event YieldDistributionParamsUpdated(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```solidity
// consider adding the `indexed` keyword for better searching & integration 
233:    event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);
234:    event StabilityPoolLUSDBalanceUpdated(uint _newBalance);
236:    event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
237:    event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
238:    event TroveManagerAddressChanged(address _newTroveManagerAddress);
239:    event ActivePoolAddressChanged(address _newActivePoolAddress);
240:    event DefaultPoolAddressChanged(address _newDefaultPoolAddress);
241:    event LUSDTokenAddressChanged(address _newLUSDTokenAddress);
242:    event SortedTrovesAddressChanged(address _newSortedTrovesAddress);
243:    event PriceFeedAddressChanged(address _newPriceFeedAddress);
244:    event CommunityIssuanceAddressChanged(address _newCommunityIssuanceAddress);
246:    event P_Updated(uint _P);
247:    event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);
248:    event G_Updated(uint _G, uint128 _epoch, uint128 _scale);
249:    event EpochUpdated(uint128 _currentEpoch);
250:    event ScaleUpdated(uint128 _currentScale);
252:    event DepositSnapshotUpdated(address indexed _depositor, uint _P, address[] _assets, uint[] _amounts, uint _G);
253:    event UserDepositChanged(address indexed _depositor, uint _newDeposit);
255:    event CollateralGainWithdrawn(address indexed _depositor, address _collateral, uint _collAmount);
256:    event LQTYPaidToDepositor(address indexed _depositor, uint _LQTY);
257:    event CollateralSent(address _collateral, address _to, uint _amount);
```