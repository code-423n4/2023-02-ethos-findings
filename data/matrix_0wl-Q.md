# Summary

## Low Risk and Non-Critical Issues

## Non-Critical Issues

|       | Issue                                                                                                  |
| ----- | :----------------------------------------------------------------------------------------------------- |
| NC-1  | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                                                          |
| NC-2  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                                   |
| NC-3  | BE EXPLICIT DECLARING TYPES                                                                            |
| NC-4  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                               |
| NC-5  | SAME CONSTANT REDEFINED ELSEWHERE                                                                      |
| NC-6  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                                                        |
| NC-7  | SOLIDITY COMPILER VERSIONS MISMATCH                                                                    |
| NC-8  | SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`                                                          |
| NC-9  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                                                |
| NC-10 | MARK VISIBILITY OF `INITIALIZE(…)` FUNCTIONS AS EXTERNAL                                               |
| NC-11 | CONSTANT VALUES SUCH AS A CALL `TO KECCAK256()`, SHOULD USE IMMUTABLE RATHER THAN CONSTANT             |
| NC-12 | LACK OF EVENT EMISSION AFTER CRITICAL `INITIALIZE()` FUNCTIONS                                         |
| NC-13 | LACK OF CHECKS SUPPORTSINTERFACE                                                                       |
| NC-14 | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION                                                  |
| NC-15 | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                                          |
| NC-16 | USE A MORE RECENT VERSION OF SOLIDITY                                                                  |
| NC-17 | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                                                            |
| NC-18 | FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE                                                      |
| NC-19 | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS                                                      |
| NC-20 | INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS                                                          |
| NC-21 | NO SAME VALUE INPUT CONTROL                                                                            |
| NC-22 | OMISSIONS IN EVENTS                                                                                    |
| NC-23 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                                                     |
| NC-24 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE                                    |
| NC-25 | PROJECT UPGRADE AND STOP SCENARIO                                                                      |
| NC-26 | DON’T USE PERIODS WITH FRAGMENTS                                                                       |
| NC-27 | UNUSED IMPORTS                                                                                         |
| NC-28 | DUPLICATED IMPORTS                                                                                     |
| NC-29 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS                                |
| NC-30 | LONG LINES ARE NOT SUITABLE FOR THE SOLIDITY STYLE GUIDE                                               |
| NC-31 | USE UNDERSCORES FOR NUMBER LITERALS                                                                    |
| NC-32 | VARIABLE NAMES THAT CONSIST OF ALL CAPITAL LETTERS SHOULD BE RESERVED FOR CONSTANT/IMMUTABLE VARIABLES |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

62:     event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);

```

#### Recommendation:

Add Event-Emit

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

71:     function setAddresses(

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

110:     function setAddresses(

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61:     function setAddresses

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:     function setAddresses

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:     function setAddresses(

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

232:     function setAddresses(

1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160:     function setHarvestSteps(address[2][] calldata _newSteps) external {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

258:     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {

603:     function setEmergencyShutdown(bool _active) external {

617:     function setLockedProfitDegradation(uint256 degradation) external {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

156:     function setEmergencyExit() external {

```

### [NC-3] BE EXPLICIT DECLARING TYPES

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

62:     event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);

63:     event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);

159:     function getCollateral(address _collateral) external view override returns (uint) {

164:     function getLUSDDebt(address _collateral) external view override returns (uint) {

171:     function sendCollateral(address _collateral, address _account, uint _amount) external override {

190:     function increaseLUSDDebt(address _collateral, uint _amount) external override {

197:     function decreaseLUSDDebt(address _collateral, uint _amount) external override {

204:     function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

51:         uint price;

52:         uint collChange;

53:         uint netDebtChange;

55:         uint debt;

56:         uint coll;

57:         uint oldICR;

58:         uint newICR;

59:         uint newTCR;

60:         uint LUSDFee;

61:         uint newDebt;

62:         uint newColl;

63:         uint stake;

70:         uint price;

71:         uint LUSDFee;

72:         uint netDebt;

73:         uint compositeDebt;

74:         uint ICR;

75:         uint NICR;

76:         uint stake;

77:         uint arrayIndex;

104:     event TroveCreated(address indexed _borrower, address _collateral, uint arrayIndex);

105:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);

105:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);

106:     event LUSDBorrowingFeePaid(address indexed _borrower, address _collateral, uint _LUSDFee);

172:     function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

206:             uint newTCR = _getNewTCRFromTroveChange(

242:     function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

248:     function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

254:     function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override {

259:     function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

264:     function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

268:     function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {

282:     function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

347:         uint newNICR = _getNewNominalICRFromTroveChange(

383:         uint price = priceFeed.fetchPrice(_collateral);

389:         uint debt = troveManagerCached.getTroveDebt(msg.sender, _collateral);

393:         uint newTCR = _getNewTCRFromTroveChange(_collateral, coll, false, debt, false, price, collDecimals);

419:     function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {

421:         uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);

432:     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

433:         uint usdValue = _price.mul(_coll).div(10**_collDecimals);

439:         uint _collReceived,

440:         uint _requestedCollWithdrawal

444:         returns(uint collChange, bool isCollIncrease)

460:         uint _collChange,

462:         uint _debtChange,

466:         returns (uint, uint)

468:         uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)

470:         uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)

482:         uint _collChange,

484:         uint _LUSDChange,

486:         uint _netDebtChange

505:     function _activePoolAddColl(IActivePool _activePool, address _collateral, uint _amount) internal {

511:     function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {

517:     function _repayLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSD) internal {

528:     function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {

533:     function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

537:     function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

542:         uint status = _troveManager.getTroveStatus(_borrower, _collateral);

547:         uint status = _troveManager.getTroveStatus(_borrower, _collateral);

551:     function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

557:         uint _price,

567:     function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {

575:         uint _collWithdrawal,

616:     function _requireICRisAboveMCR(uint _newICR, uint256 _MCR) internal pure {

620:     function _requireICRisAboveCCR(uint _newICR, uint256 _CCR) internal pure {

624:     function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

628:     function _requireNewTCRisAboveCCR(uint _newTCR, uint256 _CCR) internal pure {

632:     function _requireAtLeastMinNetDebt(uint _netDebt) internal pure {

636:     function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

644:      function _requireSufficientLUSDBalance(ILUSDToken _lusdToken, address _borrower, uint _debtRepayment) internal view {

648:     function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {

663:         uint _coll,

664:         uint _debt,

665:         uint _collChange,

667:         uint _debtChange,

673:         returns (uint)

675:         (uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);

677:         uint newNICR = LiquityMath._computeNominalCR(newColl, newDebt, _collateralDecimals);

684:         uint _coll,

685:         uint _debt,

686:         uint _collChange,

688:         uint _debtChange,

690:         uint _price,

695:         returns (uint)

697:         (uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);

699:         uint newICR = LiquityMath._computeCR(newColl, newDebt, _price, _collateralDecimals);

704:         uint _coll,

705:         uint _debt,

706:         uint _collChange,

708:         uint _debtChange,

713:         returns (uint, uint)

715:         uint newColl = _coll;

716:         uint newDebt = _debt;

727:         uint _collChange,

729:         uint _debtChange,

731:         uint _price,

736:         returns (uint)

738:         uint totalColl = getEntireSystemColl(_collateral);

739:         uint totalDebt = getEntireSystemDebt(_collateral);

744:         uint newTCR = LiquityMath._computeCR(totalColl, totalDebt, _price, _collateralDecimals);

748:     function getCompositeDebt(uint _debt) external pure override returns (uint) {

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

15:     using SafeMath for uint;

41:     uint public totalOATHIssued;

42:     uint public lastDistributionTime;

43:     uint public distributionPeriod;

44:     uint public rewardPerSecond;

46:     uint public lastIssuanceTimestamp;

51:     event LogRewardPerSecond(uint _rewardPerSecond);

53:     event TotalOATHIssuedUpdated(uint _totalOATHIssued);

84:     function issueOath() external override returns (uint issuance) {

101:     function fund(uint amount) external onlyOwner {

107:             uint timeLeft = lastDistributionTime.sub(lastIssuanceTimestamp);

108:             uint notIssued = timeLeft.mul(rewardPerSecond);

124:     function sendOath(address _account, uint _OathAmount) external override {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

20:     using SafeMath for uint;

25:     mapping( address => uint) public stakes;

26:     uint public totalLQTYStaked;

28:     mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked

29:     uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked

35:         mapping (address => uint) F_Collateral_Snapshot;

36:         uint F_LUSD_Snapshot;

56:     event StakeChanged(address indexed staker, uint newStake);

57:     event StakingGainsWithdrawn(address indexed staker, uint LUSDGain, address[] _assets, uint[] _amounts);

57:     event StakingGainsWithdrawn(address indexed staker, uint LUSDGain, address[] _assets, uint[] _amounts);

58:     event F_CollateralUpdated(address _collateral, uint _F_Collateral);

59:     event F_LUSDUpdated(uint _F_LUSD);

60:     event TotalLQTYStakedUpdated(uint _totalLQTYStaked);

61:     event CollateralSent(address _account, address _collateral, uint _amount);

62:     event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);

62:     event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);

104:     function stake(uint _LQTYamount) external override {

107:         uint currentStake = stakes[msg.sender];

110:         uint[] memory collGainAmounts;

111:         uint LUSDGain;

120:         uint newStake = currentStake.add(_LQTYamount);

142:     function unstake(uint _LQTYamount) external override {

143:         uint currentStake = stakes[msg.sender];

147:         (address[] memory collGainAssets, uint[] memory collGainAmounts) = _getPendingCollateralGain(msg.sender);

148:         uint LUSDGain = _getPendingLUSDGain(msg.sender);

153:             uint LQTYToWithdraw = LiquityMath._min(_LQTYamount, currentStake);

155:             uint newStake = currentStake.sub(LQTYToWithdraw);

177:     function increaseF_Collateral(address _collateral, uint _collFee) external override {

179:         uint collFeePerLQTYStaked;

187:     function increaseF_LUSD(uint _LUSDFee) external override {

189:         uint LUSDFeePerLQTYStaked;

199:     function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory) {

203:     function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {

205:         amounts = new uint[](assets.length);

206:         for (uint i = 0; i < assets.length; i++) {

208:             uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];

213:     function getPendingLUSDGain(address _user) external view override returns (uint) {

217:     function _getPendingLUSDGain(address _user) internal view returns (uint) {

218:         uint F_LUSD_Snapshot = snapshots[_user].F_LUSD_Snapshot;

219:         uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);

227:         uint[] memory amounts = new uint[](collaterals.length);

228:         for (uint i = 0; i < collaterals.length; i++) {

233:         uint fLUSD = F_LUSD;

238:     function _sendCollGainToUser(address[] memory assets, uint[] memory amounts) internal {

239:         uint numCollaterals = assets.length;

240:         for (uint i = 0; i < numCollaterals; i++) {

265:     function _requireUserHasStake(uint currentStake) internal pure {

269:     function _requireNonZeroAmount(uint _amount) internal pure {

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

75:     uint internal immutable deploymentStartTime;

266:         uint amount,

267:         uint deadline,

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

175:         uint initialValue;

179:         mapping (address => uint) S;

180:         uint P;

181:         uint G;

195:     uint public P = DECIMAL_PRECISION;

197:     uint public constant SCALE_FACTOR = 1e9;

214:     mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;

223:     mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;

226:     uint public lastLQTYError;

228:     mapping (address => uint) public lastCollateralError_Offset;

229:     uint public lastLUSDLossError_Offset;

233:     event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);

234:     event StabilityPoolLUSDBalanceUpdated(uint _newBalance);

246:     event P_Updated(uint _P);

247:     event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);

248:     event G_Updated(uint _G, uint128 _epoch, uint128 _scale);

252:     event DepositSnapshotUpdated(address indexed _depositor, uint _P, address[] _assets, uint[] _amounts, uint _G);

253:     event UserDepositChanged(address indexed _depositor, uint _newDeposit);

255:     event CollateralGainWithdrawn(address indexed _depositor, address _collateral, uint _collAmount);

256:     event LQTYPaidToDepositor(address indexed _depositor, uint _LQTY);

257:     event CollateralSent(address _collateral, address _to, uint _amount);

307:     function getCollateral(address _collateral) external view override returns (uint) {

311:     function getTotalLUSDDeposits() external view override returns (uint) {

323:     function provideToSP(uint _amount) external override {

326:         uint initialDeposit = deposits[msg.sender].initialValue;

332:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

333:         uint compoundedLUSDDeposit = getCompoundedLUSDDeposit(msg.sender);

339:         uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log

346:         uint newDeposit = compoundedLUSDDeposit.add(_amount);

350:         uint numCollaterals = assets.length;

351:         for (uint i = 0; i < numCollaterals; i++) {

353:             uint amount = amounts[i];

367:     function withdrawFromSP(uint _amount) external override {

369:         uint initialDeposit = deposits[msg.sender].initialValue;

376:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

377:         uint compoundedLUSDDeposit = getCompoundedLUSDDeposit(msg.sender);

378:         uint LUSDtoWithdraw = LiquityMath._min(_amount, compoundedLUSDDeposit);

384:         uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log

392:         uint newDeposit = compoundedLUSDDeposit.sub(LUSDtoWithdraw);

396:         uint numCollaterals = assets.length;

397:         for (uint i = 0; i < numCollaterals; i++) {

399:             uint amount = amounts[i];

410:     function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {

417:         uint LQTYIssuance = _communityIssuance.issueOath();

421:     function _updateG(uint _LQTYIssuance) internal {

422:         uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD

430:         uint LQTYPerUnitStaked;

433:         uint marginalLQTYGain = LQTYPerUnitStaked.mul(P);

439:     function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

451:         uint LQTYNumerator = _LQTYIssuance.mul(DECIMAL_PRECISION).add(lastLQTYError);

453:         uint LQTYPerUnitStaked = LQTYNumerator.div(_totalLUSDDeposits);

466:     function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {

468:         uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD

473:         (uint collGainPerUnitStaked,

474:             uint LUSDLossPerUnitStaked) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, _debtToOffset, totalLUSD);

486:     function updateRewardSum(address _collateral, uint _collToAdd) external override {

488:         uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD

493:         (uint collGainPerUnitStaked, ) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, 0, totalLUSD);

497:         uint sum = collAmounts[_collateral].add(_collToAdd);

506:         uint _collToAdd,

507:         uint _debtToOffset,

508:         uint _totalLUSDDeposits

511:         returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)

524:         uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);

531:             uint LUSDLossNumerator = _debtToOffset.mul(DECIMAL_PRECISION).sub(lastLUSDLossError_Offset);

547:     function _updateRewardSumAndProduct(address _collateral, uint _collGainPerUnitStaked, uint _LUSDLossPerUnitStaked) internal {

548:         uint currentP = P;

549:         uint newP;

556:         uint newProductFactor = uint(DECIMAL_PRECISION).sub(_LUSDLossPerUnitStaked);

560:         uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral];

569:         uint marginalCollGain = _collGainPerUnitStaked.mul(currentP);

570:         uint newS = currentS.add(marginalCollGain);

597:     function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

608:         uint sum = collAmounts[_collateral].add(_collToAdd);

613:     function _decreaseLUSD(uint _amount) internal {

614:         uint newTotalLUSDDeposits = totalLUSDDeposits.sub(_amount);

626:     function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {

627:         uint initialDeposit = deposits[_depositor].initialValue;

637:     function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

639:         amounts = new uint[](assets.length);

640:         for (uint i = 0; i < assets.length; i++) {

650:         uint P_Snapshot;

651:         uint S_Snapshot;

652:         uint firstPortion;

653:         uint secondPortion;

654:         uint gain;

657:     function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {

683:     function getDepositorLQTYGain(address _depositor) public view override returns (uint) {

684:         uint initialDeposit = deposits[_depositor].initialValue;

688:         uint LQTYGain = _getLQTYGainFromSnapshots(initialDeposit, snapshots);

693:     function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

701:         uint G_Snapshot = snapshots.G;

702:         uint P_Snapshot = snapshots.P;

704:         uint firstPortion = epochToScaleToG[epochSnapshot][scaleSnapshot].sub(G_Snapshot);

705:         uint secondPortion = epochToScaleToG[epochSnapshot][scaleSnapshot.add(1)].div(SCALE_FACTOR);

707:         uint LQTYGain = initialDeposit.mul(firstPortion.add(secondPortion)).div(P_Snapshot).div(DECIMAL_PRECISION);

718:     function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {

719:         uint initialDeposit = deposits[_depositor].initialValue;

724:         uint compoundedDeposit = _getCompoundedDepositFromSnapshots(initialDeposit, snapshots);

730:         uint initialDeposit,

735:         returns (uint)

737:         uint snapshot_P = snapshots.P;

744:         uint compoundedDeposit;

776:     function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

778:         uint newTotalLUSDDeposits = totalLUSDDeposits.add(_amount);

783:     function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {

785:         uint newCollAmount = collAmounts[_collateral].sub(_amount);

794:     function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

803:     function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {

807:         uint[] memory amounts = new uint[](collaterals.length);

810:             for (uint i = 0; i < collaterals.length; i++) {

819:         uint currentP = P;

822:         uint currentG = epochToScaleToG[currentEpochCached][currentScaleCached];

831:         for (uint i = 0; i < collaterals.length; i++) {

833:             uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];

841:         uint depositorLQTYGain = getDepositorLQTYGain(_depositor);

858:         uint numCollaterals = collaterals.length;

859:         for (uint i = 0; i < numCollaterals; i++) {

861:             uint price = priceFeed.fetchPrice(collateral);

864:             uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);

869:     function _requireUserHasDeposit(uint _initialDeposit) internal pure {

873:     function _requireNonZeroAmount(uint _amount) internal pure {

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

48:     uint constant public SECONDS_IN_ONE_MINUTE = 60;

53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

61:     uint constant public BETA = 2;

63:     uint public baseRate;

66:     uint public lastFeeOperationTime;

78:         uint debt;

79:         uint coll;

80:         uint stake;

88:     mapping (address => uint) public totalStakes;

91:     mapping (address => uint) public totalStakesSnapshot;

94:     mapping (address => uint) public totalCollateralSnapshot;

104:     mapping (address => uint) public L_Collateral;

105:     mapping (address => uint) public L_LUSDDebt;

112:     struct RewardSnapshot { uint collAmount; uint LUSDDebt;}

112:     struct RewardSnapshot { uint collAmount; uint LUSDDebt;}

119:     mapping (address => uint) public lastCollateralError_Redistribution;

120:     mapping (address => uint) public lastLUSDDebtError_Redistribution;

133:         uint price;

134:         uint LUSDInStabPool;

136:         uint liquidatedDebt;

137:         uint liquidatedColl;

141:         uint collToLiquidate;

142:         uint pendingDebtReward;

143:         uint pendingCollReward;

147:         uint remainingLUSDInStabPool;

148:         uint i;

149:         uint ICR;

150:         uint TCR;

153:         uint entireSystemDebt;

154:         uint entireSystemColl;

161:         uint entireTroveDebt;

162:         uint entireTroveColl;

163:         uint collGasCompensation;

164:         uint LUSDGasCompensation;

165:         uint debtToOffset;

166:         uint collToSendToSP;

167:         uint debtToRedistribute;

168:         uint collToRedistribute;

169:         uint collSurplus;

173:         uint totalCollInSequence;

174:         uint totalDebtInSequence;

175:         uint totalCollGasCompensation;

176:         uint totalLUSDGasCompensation;

177:         uint totalDebtToOffset;

178:         uint totalCollToSendToSP;

179:         uint totalDebtToRedistribute;

180:         uint totalCollToRedistribute;

181:         uint totalCollSurplus;

200:     event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);

201:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint _stake, TroveManagerOperation _operation);

202:     event TroveLiquidated(address indexed _borrower, address _collateral, uint _debt, uint _coll, TroveManagerOperation _operation);

203:     event BaseRateUpdated(uint _baseRate);

204:     event LastFeeOpTimeUpdated(uint _lastFeeOpTime);

205:     event TotalStakesUpdated(address _collateral, uint _newTotalStakes);

206:     event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);

207:     event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

208:     event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

209:     event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);

212:         uint _attemptedLUSDAmount,

213:         uint _actualLUSDAmount,

214:         uint _collSent,

215:         uint _collFee

299:     function getTroveOwnersCount(address _collateral) external view override returns (uint) {

303:     function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {

326:         uint _LUSDInStabPool

343:         uint collToLiquidate = singleLiquidation.entireTroveColl.sub(singleLiquidation.collGasCompensation);

362:         uint _ICR,

363:         uint _LUSDInStabPool,

364:         uint _TCR,

365:         uint _price,

420:             uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);

444:         uint _debt,

445:         uint _coll,

446:         uint _LUSDInStabPool

450:         returns (uint debtToOffset, uint collToSendToSP, uint debtToRedistribute, uint collToRedistribute)

480:         uint _entireTroveDebt,

481:         uint _entireTroveColl,

482:         uint _price,

484:         uint _collDecimals

492:         uint cappedCollPortion = _entireTroveDebt.mul(_MCR).div(_price);

513:     function liquidateTroves(address _collateral, uint _n) external override {

588:         uint _price,

589:         uint _LUSDInStabPool,

590:         uint _n

676:         uint _price,

678:         uint _LUSDInStabPool,

679:         uint _n

790:         uint _price,

791:         uint _LUSDInStabPool,

819:                 uint TCR = LiquityMath._computeCR(vars.entireSystemColl, vars.entireSystemDebt, _price, vars.collDecimals);

869:         uint _price,

871:         uint _LUSDInStabPool,

915:     function _sendGasCompensation(IActivePool _activePool, address _collateral, address _liquidator, uint _LUSD, uint _collAmount) internal {

926:     function _movePendingTroveRewardsToActivePool(IActivePool _activePool, IDefaultPool _defaultPool, address _collateral, uint _LUSD, uint _collAmount) internal {

985:         uint _attemptedLUSDAmount,

986:         uint _actualLUSDAmount,

987:         uint _collSent,

988:         uint _collFee

1018:         uint _LUSDamount,

1022:         uint _partialRedemptionHintNICR,

1023:         uint _maxIterations,

1024:         uint _maxFeePercentage

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1046:         (uint currentCollateral, uint currentLUSDDebt) = _getCurrentTroveAmounts(_borrower, _collateral);

1049:         uint NICR = LiquityMath._computeNominalCR(currentCollateral, currentLUSDDebt, collDecimals);

1057:         uint _price

1058:     ) public view override returns (uint) {

1059:         (uint currentCollateral, uint currentLUSDDebt) = _getCurrentTroveAmounts(_borrower, _collateral);

1062:         uint ICR = LiquityMath._computeCR(currentCollateral, currentLUSDDebt, _price, collDecimals);

1066:     function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {

1067:         uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);

1068:         uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);

1070:         uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);

1071:         uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

1087:             uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);

1088:             uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);

1123:     function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1124:         uint snapshotCollateral = rewardSnapshots[_borrower][_collateral].collAmount;

1125:         uint rewardPerUnitStaked = L_Collateral[_collateral].sub(snapshotCollateral);

1129:         uint stake = Troves[_borrower][_collateral].stake;

1132:         uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1138:     function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1139:         uint snapshotLUSDDebt = rewardSnapshots[_borrower][_collateral].LUSDDebt;

1140:         uint rewardPerUnitStaked = L_LUSDDebt[_collateral].sub(snapshotLUSDDebt);

1144:         uint stake = Troves[_borrower][_collateral].stake;

1147:         uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1190:         uint stake = Troves[_borrower][_collateral].stake;

1195:     function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {

1201:     function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {

1202:         uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);

1203:         uint oldStake = Troves[_borrower][_collateral].stake;

1213:     function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1214:         uint stake;

1234:         uint _debt,

1235:         uint _coll,

1251:         uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);

1252:         uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);

1255:         uint collateralRewardPerUnitStaked = collateralNumerator.div(totalStakes[_collateral]);

1256:         uint LUSDDebtRewardPerUnitStaked = LUSDDebtNumerator.div(totalStakes[_collateral]);

1281:         uint TroveOwnersArrayLength = TroveOwners[_collateral].length;

1305:     function _updateSystemSnapshots_excludeCollRemainder(IActivePool _activePool, address _collateral, uint _collRemainder) internal {

1308:         uint activeColl = _activePool.getCollateral(_collateral);

1309:         uint liquidatedColl = defaultPool.getCollateral(_collateral);

1316:     function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {

1339:     function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {

1345:         uint length = TroveOwnersArrayLength;

1346:         uint idxLast = length.sub(1);

1361:     function getTCR(address _collateral, uint _price) external view override returns (uint) {

1366:     function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {

1374:         uint _entireSystemColl,

1375:         uint _entireSystemDebt,

1376:         uint _price,

1384:         uint TCR = LiquityMath._computeCR(_entireSystemColl, _entireSystemDebt, _price, _collDecimals);

1398:         uint _collateralDrawn,

1399:         uint _price,

1401:         uint _totalLUSDSupply

1402:     ) external override returns (uint) {

1404:         uint decayedBaseRate = _calcDecayedBaseRate();

1408:         uint redeemedLUSDFraction =

1411:         uint newBaseRate = decayedBaseRate.add(redeemedLUSDFraction.div(BETA));

1425:     function getRedemptionRate() public view override returns (uint) {

1429:     function getRedemptionRateWithDecay() public view override returns (uint) {

1433:     function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {

1433:     function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1444:     function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1444:     function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1448:     function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {

1449:         uint redemptionFee = _redemptionRate.mul(_collateralDrawn).div(DECIMAL_PRECISION);

1456:     function getBorrowingRate() public view override returns (uint) {

1460:     function getBorrowingRateWithDecay() public view override returns (uint) {

1464:     function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {

1471:     function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {

1475:     function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {

1479:     function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {

1488:         uint decayedBaseRate = _calcDecayedBaseRate();

1501:         uint timePassed = block.timestamp.sub(lastFeeOperationTime);

1509:     function _calcDecayedBaseRate() internal view returns (uint) {

1510:         uint minutesPassed = _minutesPassedSinceLastFeeOp();

1511:         uint decayFactor = LiquityMath._decPow(MINUTE_DECAY_FACTOR, minutesPassed);

1516:     function _minutesPassedSinceLastFeeOp() internal view returns (uint) {

1538:     function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {

1544:     function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {

1545:         return uint(Troves[_borrower][_collateral].status);

1548:     function getTroveStake(address _borrower, address _collateral) external view override returns (uint) {

1552:     function getTroveDebt(address _borrower, address _collateral) external view override returns (uint) {

1556:     function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {

1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

1567:     function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {

1569:         uint newColl = Troves[_borrower][_collateral].coll.add(_collIncrease);

1574:     function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

1576:         uint newColl = Troves[_borrower][_collateral].coll.sub(_collDecrease);

1581:     function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {

1583:         uint newDebt = Troves[_borrower][_collateral].debt.add(_debtIncrease);

1588:     function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

1590:         uint newDebt = Troves[_borrower][_collateral].debt.sub(_debtDecrease);

```

### [NC-4] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-5] SAME CONSTANT REDEFINED ELSEWHERE

#### Description:

Keeping the same constants in different files may cause some problems or errors, reading constants from a single file is preferable. This should also be preferred in gas optimization.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

41:     uint256 public constant PERCENT_DIVISOR = 10000;

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

23:     uint256 public constant PERCENT_DIVISOR = 10_000;

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [NC-6] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes.

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

23:     uint256 public constant PERCENT_DIVISOR = 10_000;

24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF

25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [NC-7] SOLIDITY COMPILER VERSIONS MISMATCH

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

3: pragma solidity ^0.8.0;

```

### [NC-8] SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`

#### Description:

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121 and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57.

While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

287:         address recoveredAddress = ecrecover(digest, v, r, s);

```

#### Recommended Mitigation Steps:

Use the `ecrecover` function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

### [NC-9] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

5: import './Interfaces/IActivePool.sol';

6: import "./Interfaces/ICollateralConfig.sol";

7: import './Interfaces/IDefaultPool.sol';

8: import "./Interfaces/ICollSurplusPool.sol";

9: import "./Interfaces/ILQTYStaking.sol";

10: import "./Interfaces/IStabilityPool.sol";

11: import "./Interfaces/ITroveManager.sol";

12: import "./Dependencies/SafeMath.sol";

13: import "./Dependencies/Ownable.sol";

14: import "./Dependencies/CheckContract.sol";

15: import "./Dependencies/console.sol";

16: import "./Dependencies/SafeERC20.sol";

17: import "./Dependencies/IERC4626.sol";

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

5: import "./Interfaces/IBorrowerOperations.sol";

6: import "./Interfaces/ICollateralConfig.sol";

7: import "./Interfaces/ITroveManager.sol";

8: import "./Interfaces/ILUSDToken.sol";

9: import "./Interfaces/ICollSurplusPool.sol";

10: import "./Interfaces/ISortedTroves.sol";

11: import "./Interfaces/ILQTYStaking.sol";

12: import "./Dependencies/LiquityBase.sol";

13: import "./Dependencies/Ownable.sol";

14: import "./Dependencies/CheckContract.sol";

15: import "./Dependencies/console.sol";

16: import "./Dependencies/SafeERC20.sol";

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

5: import "./Dependencies/CheckContract.sol";

6: import "./Dependencies/Ownable.sol";

7: import "./Dependencies/SafeERC20.sol";

8: import "./Interfaces/ICollateralConfig.sol";

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

5: import "../Dependencies/IERC20.sol";

6: import "../Interfaces/ICommunityIssuance.sol";

7: import "../Dependencies/BaseMath.sol";

8: import "../Dependencies/LiquityMath.sol";

9: import "../Dependencies/Ownable.sol";

10: import "../Dependencies/CheckContract.sol";

11: import "../Dependencies/SafeMath.sol";

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

5: import "../Dependencies/BaseMath.sol";

6: import "../Dependencies/SafeMath.sol";

7: import "../Dependencies/Ownable.sol";

8: import "../Dependencies/CheckContract.sol";

9: import "../Dependencies/console.sol";

10: import "../Dependencies/IERC20.sol";

11: import "../Interfaces/ICollateralConfig.sol";

12: import "../Interfaces/ILQTYStaking.sol";

13: import "../Interfaces/ITroveManager.sol";

14: import "../Dependencies/LiquityMath.sol";

15: import "../Interfaces/ILUSDToken.sol";

16: import "../Dependencies/SafeERC20.sol";

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

5: import "./Interfaces/ILUSDToken.sol";

6: import "./Interfaces/ITroveManager.sol";

7: import "./Dependencies/SafeMath.sol";

8: import "./Dependencies/CheckContract.sol";

9: import "./Dependencies/console.sol";

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

5: import './Interfaces/IBorrowerOperations.sol';

6: import "./Interfaces/ICollateralConfig.sol";

7: import './Interfaces/IStabilityPool.sol';

8: import './Interfaces/IBorrowerOperations.sol';

9: import './Interfaces/ITroveManager.sol';

10: import './Interfaces/ILUSDToken.sol';

11: import './Interfaces/ISortedTroves.sol';

12: import "./Interfaces/ICommunityIssuance.sol";

13: import "./Dependencies/LiquityBase.sol";

14: import "./Dependencies/SafeMath.sol";

15: import "./Dependencies/LiquitySafeMath128.sol";

16: import "./Dependencies/Ownable.sol";

17: import "./Dependencies/CheckContract.sol";

18: import "./Dependencies/console.sol";

19: import "./Dependencies/SafeERC20.sol";

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

5: import "./Interfaces/ICollateralConfig.sol";

6: import "./Interfaces/ITroveManager.sol";

7: import "./Interfaces/IStabilityPool.sol";

8: import "./Interfaces/ICollSurplusPool.sol";

9: import "./Interfaces/ILUSDToken.sol";

10: import "./Interfaces/ISortedTroves.sol";

11: import "./Interfaces/ILQTYStaking.sol";

12: import "./Interfaces/IRedemptionHelper.sol";

13: import "./Dependencies/LiquityBase.sol";

15: import "./Dependencies/CheckContract.sol";

16: import "./Dependencies/IERC20.sol";

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

5: import "./abstract/ReaperBaseStrategyv4.sol";

6: import "./interfaces/IAToken.sol";

7: import "./interfaces/IAaveProtocolDataProvider.sol";

8: import "./interfaces/ILendingPool.sol";

9: import "./interfaces/ILendingPoolAddressesProvider.sol";

10: import "./interfaces/IRewardsController.sol";

11: import "./libraries/ReaperMathUtils.sol";

12: import "./mixins/VeloSolidMixin.sol";

13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

5: import "./ReaperVaultV2.sol";

6: import "./interfaces/IERC4626Functions.sol";

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

5: import "./interfaces/IERC4626Events.sol";

6: import "./interfaces/IStrategy.sol";

7: import "./libraries/ReaperMathUtils.sol";

8: import "./mixins/ReaperAccessControl.sol";

9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";

10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";

13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

14: import "@openzeppelin/contracts/utils/math/Math.sol";

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

5: import "../interfaces/IStrategy.sol";

6: import "../interfaces/IVault.sol";

7: import "../libraries/ReaperMathUtils.sol";

8: import "../mixins/ReaperAccessControl.sol";

9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";

10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;

```solidity
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

### [NC-10] MARK VISIBILITY OF `INITIALIZE(…)` FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden `initialize(...)` function calls the `internal {...}\_init` function, not the parent public `initialize(...)` function.

`External` instead of `public` would give more sense of the `initialize(...)` functions to behave like a `constructor` (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use `external` over `public`.

It is possible to override a function from `external` to `public `(= “opening it up”) ✅ but it is not possible to override a function from `public` to `external` (= “narrow it down”). ❌

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:     function initialize(
            address _vault,
            address[] memory _strategists,
            address[] memory _multisigRoles,
            IAToken _gWant
    )   public initializer {
            gWant = _gWant;
            want = _gWant.UNDERLYING_ASSET_ADDRESS();
            __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
            rewardClaimingTokens = [address(_gWant)];
    }

```

#### Recommended Mitigation Steps:

For the above reasons, you can change `initialize(...)` to `external:`

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

### [NC-11] CONSTANT VALUES SUCH AS A CALL `TO KECCAK256()`, SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

WConstants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [NC-12] LACK OF EVENT EMISSION AFTER CRITICAL `INITIALIZE()` FUNCTIONS

#### Description:

It is important to record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` function (as in `CollateralConfig.sol` contract).

`ReaperStrategyGranarySupplyOnly.sol` contract initialized but its critical init parameters are not logged for any off-chain monitoring.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:     function initialize(

```

#### Recommended Mitigation Steps:

Please consider emitting an event after the `initialize()` function.

### [NC-13] LACK OF CHECKS SUPPORTSINTERFACE

#### Description:

The EIP-165 standard helps detect that a smart contract implements the expected logic, prevents human error when configuring smart contract bindings, so it is recommended to check that the received argument is a contract and supports the expected interface.

Reference: https://eips.ethereum.org/EIPS/eip-165

### [NC-14] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

Use (e.g. 1e5) rather than decimal literals (e.g. 10000), for better code readability.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

41:     uint256 public constant PERCENT_DIVISOR = 10000;

```

### [NC-15] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

3: pragma solidity ^0.8.0;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-16] USE A MORE RECENT VERSION OF SOLIDITY

#### Context:

All contracts

#### Description:

For security, it is best practice to use the latest Solidity version.

For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

3: pragma solidity ^0.8.0;

```

#### Recommendation mitigation steps:

Old version of Solidity is used , newer version can be used (0.8.17)

### [NC-17] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-access-control

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160:     function setHarvestSteps(address[2][] calldata _newSteps) external {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

156:     function setEmergencyExit() external {

```

### [NC-18] FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

#### Description:

Use camel case for all functions, parameters and variables and snake case for constants

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

220:     struct LocalVariables_rebalance {

240:         LocalVariables_rebalance memory vars;

299:                     ILQTYStaking(lqtyStakingAddress).increaseF_Collateral(_collateral, vars.stakingSplit);

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

47:      struct LocalVariables_adjustTrove {

66:     struct LocalVariables_openTrove {

176:         LocalVariables_openTrove memory vars;

284:         LocalVariables_adjustTrove memory vars;

426:         lqtyStaking.increaseF_LUSD(LUSDFee);

577:         LocalVariables_adjustTrove memory _vars

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

28:     mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked

29:     uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked

35:         mapping (address => uint) F_Collateral_Snapshot;

36:         uint F_LUSD_Snapshot;

58:     event F_CollateralUpdated(address _collateral, uint _F_Collateral);

58:     event F_CollateralUpdated(address _collateral, uint _F_Collateral);

59:     event F_LUSDUpdated(uint _F_LUSD);

59:     event F_LUSDUpdated(uint _F_LUSD);

62:     event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);

177:     function increaseF_Collateral(address _collateral, uint _collFee) external override {

183:         F_Collateral[_collateral] = F_Collateral[_collateral].add(collFeePerLQTYStaked);

184:         emit F_CollateralUpdated(_collateral, F_Collateral[_collateral]);

187:     function increaseF_LUSD(uint _LUSDFee) external override {

193:         F_LUSD = F_LUSD.add(LUSDFeePerLQTYStaked);

194:         emit F_LUSDUpdated(F_LUSD);

208:             uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];

218:         uint F_LUSD_Snapshot = snapshots[_user].F_LUSD_Snapshot;

219:         uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);

230:             snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];

231:             amounts[i] = F_Collateral[collateral];

234:         snapshots[_user].F_LUSD_Snapshot = fLUSD;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

195:     uint public P = DECIMAL_PRECISION;

228:     mapping (address => uint) public lastCollateralError_Offset;

229:     uint public lastLUSDLossError_Offset;

246:     event P_Updated(uint _P);

247:     event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);

248:     event G_Updated(uint _G, uint128 _epoch, uint128 _scale);

410:     function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {

436:         emit G_Updated(epochToScaleToG[currentEpoch][currentScale], currentEpoch, currentScale);

524:         uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);

524:         uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);

529:             lastLUSDLossError_Offset = 0;

531:             uint LUSDLossNumerator = _debtToOffset.mul(DECIMAL_PRECISION).sub(lastLUSDLossError_Offset);

537:             lastLUSDLossError_Offset = (LUSDLossPerUnitStaked.mul(_totalLUSDDeposits)).sub(LUSDLossNumerator);

541:         lastCollateralError_Offset[_collateral] = collNumerator.sub(collGainPerUnitStaked.mul(_totalLUSDDeposits));

594:         emit P_Updated(newP);

646:     struct LocalVariables_getSingularCollateralGain {

650:         uint P_Snapshot;

651:         uint S_Snapshot;

663:         LocalVariables_getSingularCollateralGain memory vars;

667:         vars.P_Snapshot = _snapshots.P;

668:         vars.S_Snapshot = _snapshots.S[_collateral];

670:         vars.firstPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot][_collateral].sub(vars.S_Snapshot);

673:         vars.gain = _initialDeposit.mul(vars.firstPortion.add(vars.secondPortion)).div(vars.P_Snapshot).div(DECIMAL_PRECISION);

701:         uint G_Snapshot = snapshots.G;

702:         uint P_Snapshot = snapshots.P;

704:         uint firstPortion = epochToScaleToG[epochSnapshot][scaleSnapshot].sub(G_Snapshot);

707:         uint LQTYGain = initialDeposit.mul(firstPortion.add(secondPortion)).div(P_Snapshot).div(DECIMAL_PRECISION);

737:         uint snapshot_P = snapshots.P;

752:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

104:     mapping (address => uint) public L_Collateral;

105:     mapping (address => uint) public L_LUSDDebt;

119:     mapping (address => uint) public lastCollateralError_Redistribution;

120:     mapping (address => uint) public lastLUSDDebtError_Redistribution;

129:     struct LocalVariables_OuterLiquidationFunction {

140:     struct LocalVariables_InnerSingleLiquidateFunction {

146:     struct LocalVariables_LiquidationSequence {

207:     event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

208:     event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

331:         LocalVariables_InnerSingleLiquidateFunction memory vars;

371:         LocalVariables_InnerSingleLiquidateFunction memory vars;

518:         LocalVariables_OuterLiquidationFunction memory vars;

530:             totals = _getTotalsFromLiquidateTrovesSequence_RecoveryMode(

540:             totals = _getTotalsFromLiquidateTrovesSequence_NormalMode(

568:         _updateSystemSnapshots_excludeCollRemainder(activePoolCached, _collateral, totals.totalCollGasCompensation);

582:     function _getTotalsFromLiquidateTrovesSequence_RecoveryMode

595:         LocalVariables_LiquidationSequence memory vars;

671:     function _getTotalsFromLiquidateTrovesSequence_NormalMode

684:         LocalVariables_LiquidationSequence memory vars;

722:         LocalVariables_OuterLiquidationFunction memory vars;

734:             totals = _getTotalFromBatchLiquidate_RecoveryMode(

743:             totals = _getTotalsFromBatchLiquidate_NormalMode(

771:         _updateSystemSnapshots_excludeCollRemainder(activePoolCached, _collateral, totals.totalCollGasCompensation);

785:     function _getTotalFromBatchLiquidate_RecoveryMode

797:         LocalVariables_LiquidationSequence memory vars;

864:     function _getTotalsFromBatchLiquidate_NormalMode

877:         LocalVariables_LiquidationSequence memory vars;

1117:         rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];

1118:         rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];

1119:         emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1125:         uint rewardPerUnitStaked = L_Collateral[_collateral].sub(snapshotCollateral);

1140:         uint rewardPerUnitStaked = L_LUSDDebt[_collateral].sub(snapshotLUSDDebt);

1160:         return (rewardSnapshots[_borrower][_collateral].collAmount < L_Collateral[_collateral]);

1251:         uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);

1252:         uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);

1258:         lastCollateralError_Redistribution[_collateral] = collateralNumerator.sub(collateralRewardPerUnitStaked.mul(totalStakes[_collateral]));

1259:         lastLUSDDebtError_Redistribution[_collateral] = LUSDDebtNumerator.sub(LUSDDebtRewardPerUnitStaked.mul(totalStakes[_collateral]));

1262:         L_Collateral[_collateral] = L_Collateral[_collateral].add(collateralRewardPerUnitStaked);

1263:         L_LUSDDebt[_collateral] = L_LUSDDebt[_collateral].add(LUSDDebtRewardPerUnitStaked);

1265:         emit LTermsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1305:     function _updateSystemSnapshots_excludeCollRemainder(IActivePool _activePool, address _collateral, uint _collRemainder) internal {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

473:     struct LocalVariables_report {

494:         LocalVariables_report memory vars;

```

### [NC-19] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

### Context:

All Contracts

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

#### Recommended Mitigation Steps:

NatSpec comments should be increased in contracts

### [NC-20] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

### Context:

All Contracts

#### Description:

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

Reference: https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### Recommended Mitigation Steps:

Include return parameters in NatSpec comments

### [NC-21] NO SAME VALUE INPUT CONTROL

#### Recommended Mitigation Steps:

Add code like this; `if (oracle == _oracle revert ADDRESS_SAME();`

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

96:         collateralConfigAddress = _collateralConfigAddress;

97:         borrowerOperationsAddress = _borrowerOperationsAddress;

98:         troveManagerAddress = _troveManagerAddress;

99:         stabilityPoolAddress = _stabilityPoolAddress;

100:         defaultPoolAddress = _defaultPoolAddress;

101:         collSurplusPoolAddress = _collSurplusPoolAddress;

102:         treasuryAddress = _treasuryAddress;

103:         lqtyStakingAddress = _lqtyStakingAddress;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

146:         stabilityPoolAddress = _stabilityPoolAddress;

147:         gasPoolAddress = _gasPoolAddress;

152:         lqtyStakingAddress = _lqtyStakingAddress;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

75:         stabilityPoolAddress = _stabilityPoolAddress;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

88:         troveManagerAddress = _troveManagerAddress;

89:         borrowerOperationsAddress = _borrowerOperationsAddress;

90:         activePoolAddress = _activePoolAddress;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

102:         troveManagerAddress = _troveManagerAddress;

106:         stabilityPoolAddress = _stabilityPoolAddress;

110:         borrowerOperationsAddress = _borrowerOperationsAddress;

114:         governanceAddress = _governanceAddress;

117:         guardianAddress = _guardianAddress;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

266:         borrowerOperationsAddress = _borrowerOperationsAddress;

271:         gasPoolAddress = _gasPoolAddress;

314:         borrowers[0] = _borrower;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

68:         gWant = _gWant;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

123:         tvlCap = _tvlCap;

124:         treasury = _treasury;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

72:         vault = _vault;

73:         want = _want;

```

### [NC-22] OMISSIONS IN EVENTS

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

115:         emit CollateralConfigAddressChanged(_collateralConfigAddress);

116:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);

117:         emit TroveManagerAddressChanged(_troveManagerAddress);

118:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);

119:         emit DefaultPoolAddressChanged(_defaultPoolAddress);

120:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);

135:         emit YieldingPercentageDriftUpdated(_driftBps);

208:         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

155:         emit CollateralConfigAddressChanged(_collateralConfigAddress);

156:         emit TroveManagerAddressChanged(_troveManagerAddress);

157:         emit ActivePoolAddressChanged(_activePoolAddress);

158:         emit DefaultPoolAddressChanged(_defaultPoolAddress);

159:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);

160:         emit GasPoolAddressChanged(_gasPoolAddress);

161:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);

162:         emit PriceFeedAddressChanged(_priceFeedAddress);

163:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);

164:         emit LUSDTokenAddressChanged(_lusdTokenAddress);

165:         emit LQTYStakingAddressChanged(_lqtyStakingAddress);

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

79:         emit OathTokenAddressSet(_oathTokenAddress);

80:         emit StabilityPoolAddressSet(_stabilityPoolAddress);

94:         emit TotalOATHIssuedUpdated(totalOATHIssued);

116:         emit LogRewardPerSecond(rewardPerSecond);

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

93:         emit LQTYTokenAddressSet(_lqtyTokenAddress);

94:         emit LQTYTokenAddressSet(_lusdTokenAddress);

95:         emit TroveManagerAddressSet(_troveManagerAddress);

96:         emit BorrowerOperationsAddressSet(_borrowerOperationsAddress);

97:         emit ActivePoolAddressSet(_activePoolAddress);

98:         emit CollateralConfigAddressSet(_collateralConfigAddress);

125:         emit TotalLQTYStakedUpdated(totalLQTYStaked);

160:             emit TotalLQTYStakedUpdated(totalLQTYStaked);

165:             emit StakeChanged(msg.sender, newStake);

194:         emit F_LUSDUpdated(F_LUSD);

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

104:         emit TroveManagerAddressChanged(_troveManagerAddress);

108:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);

112:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);

115:         emit GovernanceAddressChanged(_governanceAddress);

118:         emit GuardianAddressChanged(_guardianAddress);

150:         emit GovernanceAddressChanged(_newGovernanceAddress);

157:         emit GuardianAddressChanged(_newGuardianAddress);

172:         emit TroveManagerAddressChanged(_newTroveManagerAddress);

176:         emit StabilityPoolAddressChanged(_newStabilityPoolAddress);

180:         emit BorrowerOperationsAddressChanged(_newBorrowerOperationsAddress);

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

293:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);

294:         emit CollateralConfigAddressChanged(_collateralConfigAddress);

295:         emit TroveManagerAddressChanged(_troveManagerAddress);

296:         emit ActivePoolAddressChanged(_activePoolAddress);

297:         emit LUSDTokenAddressChanged(_lusdTokenAddress);

298:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);

299:         emit PriceFeedAddressChanged(_priceFeedAddress);

300:         emit CommunityIssuanceAddressChanged(_communityIssuanceAddress);

577:             emit EpochUpdated(currentEpoch);

579:             emit ScaleUpdated(currentScale);

586:             emit ScaleUpdated(currentScale);

594:         emit P_Updated(newP);

616:         emit StabilityPoolLUSDBalanceUpdated(newTotalLUSDDeposits);

780:         emit StabilityPoolLUSDBalanceUpdated(newTotalLUSDDeposits);

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

280:         emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);

281:         emit CollateralConfigAddressChanged(_collateralConfigAddress);

282:         emit ActivePoolAddressChanged(_activePoolAddress);

283:         emit DefaultPoolAddressChanged(_defaultPoolAddress);

284:         emit StabilityPoolAddressChanged(_stabilityPoolAddress);

285:         emit GasPoolAddressChanged(_gasPoolAddress);

286:         emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);

287:         emit PriceFeedAddressChanged(_priceFeedAddress);

288:         emit LUSDTokenAddressChanged(_lusdTokenAddress);

289:         emit SortedTrovesAddressChanged(_sortedTrovesAddress);

290:         emit LQTYTokenAddressChanged(_lqtyTokenAddress);

291:         emit LQTYStakingAddressChanged(_lqtyStakingAddress);

292:         emit RedemptionHelperAddressChanged(_redemptionHelperAddress);

1418:         emit BaseRateUpdated(newBaseRate);

1492:         emit BaseRateUpdated(decayedBaseRate);

1505:             emit LastFeeOpTimeUpdated(block.timestamp);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

216:         emit StrategyRevoked(_strategy);

270:         emit UpdateWithdrawalQueue(_withdrawalQueue);

570:         emit WithdrawMaxLossUpdated(_withdrawMaxLoss);

581:         emit TvlCapUpdated(tvlCap);

610:         emit EmergencyShutdown(_active);

621:         emit LockedProfitDegradationUpdated(degradation);

```

### [NC-23] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

  solidity: {
    compilers: [
      {
        version: "0.8.11",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-24] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

#### Context:

All Contracts

#### Description:

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last

### [NC-25] PROJECT UPGRADE AND STOP SCENARIO

At the start of the project, the system may need to be stopped or upgraded. I suggest you have a script beforehand and add it to the documentation.

This can also be called an “EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [NC-26] DON’T USE PERIODS WITH FRAGMENTS

#### Context:

All Contracts

#### Description:

Throughout the files, some of the comments have fragments that end with periods and some of them end without periods. It should be consistent. They should either be converted to actual sentences with both a noun phrase and a verb phrase, or the periods should be removed.

#### **Proof Of Concept**

One example:

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
44:     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)

```

### [NC-27] UNUSED IMPORTS

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

8: import "./Interfaces/ICollSurplusPool.sol";

11: import "./Interfaces/ITroveManager.sol";

15: import "./Dependencies/console.sol";

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

15: import "./Dependencies/console.sol";

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

8: import "../Dependencies/LiquityMath.sol";

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

9: import "../Dependencies/console.sol";

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

6: import "./Interfaces/ITroveManager.sol";

9: import "./Dependencies/console.sol";

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

5: import './Interfaces/IBorrowerOperations.sol';

8: import './Interfaces/IBorrowerOperations.sol';

18: import "./Dependencies/console.sol";

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

9: import "./interfaces/ILendingPoolAddressesProvider.sol";

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

```

### [NC-28] DUPLICATED IMPORTS

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

5: import './Interfaces/IBorrowerOperations.sol';

8: import './Interfaces/IBorrowerOperations.sol';
```

### [NC-29] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

41:     mapping (address => uint256) internal collAmount;  // collateral => amount tracker

42:     mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

75:     uint internal immutable deploymentStartTime;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

167:     mapping (address => uint256) internal collAmounts;  // deposited collateral tracker

170:     uint256 internal totalLUSDDeposits;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

269:     function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {

```

### [NC-30] LONG LINES ARE NOT SUITABLE FOR THE SOLIDITY STYLE GUIDE

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Recommended Mitigation Steps

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

41:     mapping (address => uint256) internal collAmount;  // collateral => amount tracker

42:     mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

44:     mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)

45:     mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming

46:     mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault

47:     mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute

49:     uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS

52:     uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS

53:     uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS

54:     uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

59:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

63:     event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);

67:     event YieldDistributionParamsUpdated(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit);

93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

105:         address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

111:             require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

145:         require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

149:         emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);

159:     function getCollateral(address _collateral) external view override returns (uint) {

164:     function getLUSDDebt(address _collateral) external view override returns (uint) {

171:     function sendCollateral(address _collateral, address _account, uint _amount) external override {

176:         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);

180:             IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);

181:             IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

183:             IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);

184:             ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

190:     function increaseLUSDDebt(address _collateral, uint _amount) external override {

197:     function decreaseLUSDDebt(address _collateral, uint _amount) external override {

204:     function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

208:         emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);

210:         IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

239:     function _rebalance(address _collateral, uint256 _amountLeavingPool) internal {

248:         vars.sharesToAssets = vars.yieldGenerator.convertToAssets(vars.ownedShares);

258:         vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);

262:         vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);

264:         if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {

266:             vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);

269:         } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {

271:             vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);

277:         vars.netAssetMovement = int256(vars.toDeposit) - int256(vars.toWithdraw) - int256(vars.profit);

279:             IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));

280:             IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));

282:             IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));

288:             vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);

291:                 vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);

293:                     IERC20(_collateral).safeTransfer(treasuryAddress, vars.treasurySplit);

296:                 vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);

298:                     IERC20(_collateral).safeTransfer(lqtyStakingAddress, vars.stakingSplit);

299:                     ILQTYStaking(lqtyStakingAddress).increaseF_Collateral(_collateral, vars.stakingSplit);

302:                 vars.stabilityPoolSplit = vars.profit.sub(vars.treasurySplit.add(vars.stakingSplit));

304:                     IERC20(_collateral).safeTransfer(stabilityPoolAddress, vars.stabilityPoolSplit);

305:                     IStabilityPool(stabilityPoolAddress).updateRewardSum(_collateral, vars.stabilityPoolSplit);

313:     function _requireValidCollateralAddress(address _collateral) internal view {

315:             ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),

328:         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());

334:             "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");

341:             "ActivePool: Caller is neither BorrowerOperations nor TroveManager");

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

18: contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {

44:     Used to hold, return and assign variables inside a function, in order to avoid the error:

104:     event TroveCreated(address indexed _borrower, address _collateral, uint arrayIndex);

105:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);

106:     event LUSDBorrowingFeePaid(address indexed _borrower, address _collateral, uint _LUSDFee);

172:     function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

174:         _requireSufficientCollateralBalanceAndAllowance(msg.sender, _collateral, _collAmount);

175:         ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

180:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

182:         bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);

185:         _requireTroveisNotActive(contractsCache.troveManager, msg.sender, _collateral);

190:             vars.LUSDFee = _triggerBorrowingFee(contractsCache.troveManager, contractsCache.lusdToken, _LUSDAmount, _maxFeePercentage);

199:         vars.ICR = LiquityMath._computeCR(_collAmount, vars.compositeDebt, vars.price, vars.collDecimals);

200:         vars.NICR = LiquityMath._computeNominalCR(_collAmount, vars.compositeDebt, vars.collDecimals);

220:         contractsCache.troveManager.increaseTroveColl(msg.sender, _collateral, _collAmount);

221:         contractsCache.troveManager.increaseTroveDebt(msg.sender, _collateral, vars.compositeDebt);

223:         contractsCache.troveManager.updateTroveRewardSnapshots(msg.sender, _collateral);

224:         vars.stake = contractsCache.troveManager.updateStakeAndTotalStakes(msg.sender, _collateral);

226:         sortedTroves.insert(_collateral, msg.sender, vars.NICR, _upperHint, _lowerHint);

227:         vars.arrayIndex = contractsCache.troveManager.addTroveOwnerToArray(msg.sender, _collateral);

231:         IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

232:         _activePoolAddColl(contractsCache.activePool, _collateral, _collAmount);

233:         _withdrawLUSD(contractsCache.activePool, contractsCache.lusdToken, _collateral, msg.sender, _LUSDAmount, vars.netDebt);

235:         _withdrawLUSD(contractsCache.activePool, contractsCache.lusdToken, _collateral, gasPoolAddress, LUSD_GAS_COMPENSATION, LUSD_GAS_COMPENSATION);

237:         emit TroveUpdated(msg.sender, _collateral, vars.compositeDebt, _collAmount, vars.stake, BorrowerOperation.openTrove);

242:     function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

243:         _requireSufficientCollateralBalanceAndAllowance(msg.sender, _collateral, _collAmount);

244:         _adjustTrove(msg.sender, _collateral, _collAmount, 0, 0, false, _upperHint, _lowerHint, 0);

248:     function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

250:         _adjustTrove(_borrower, _collateral, _collAmount, 0, 0, false, _upperHint, _lowerHint, 0);

254:     function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override {

255:         _adjustTrove(msg.sender, _collateral, 0, _collWithdrawal, 0, false, _upperHint, _lowerHint, 0);

259:     function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

260:         _adjustTrove(msg.sender, _collateral, 0, 0, _LUSDAmount, true, _upperHint, _lowerHint, _maxFeePercentage);

264:     function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

265:         _adjustTrove(msg.sender, _collateral, 0, 0, _LUSDAmount, false, _upperHint, _lowerHint, 0);

268:     function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {

270:             _requireSufficientCollateralBalanceAndAllowance(msg.sender, _collateral, _collTopUp);

272:         _adjustTrove(msg.sender, _collateral, _collTopUp, _collWithdrawal, _LUSDChange, _isDebtIncrease, _upperHint, _lowerHint, _maxFeePercentage);

282:     function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

283:         ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

288:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

290:         bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);

298:         _requireTroveisActive(contractsCache.troveManager, _borrower, _collateral);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

303:         contractsCache.troveManager.applyPendingRewards(_borrower, _collateral);

306:         (vars.collChange, vars.isCollIncrease) = _getCollChange(_collTopUp, _collWithdrawal);

312:             vars.LUSDFee = _triggerBorrowingFee(contractsCache.troveManager, contractsCache.lusdToken, _LUSDChange, _maxFeePercentage);

313:             vars.netDebtChange = vars.netDebtChange.add(vars.LUSDFee); // The raw debt change includes the fee

316:         vars.debt = contractsCache.troveManager.getTroveDebt(_borrower, _collateral);

317:         vars.coll = contractsCache.troveManager.getTroveColl(_borrower, _collateral);

320:         vars.oldICR = LiquityMath._computeCR(vars.coll, vars.debt, vars.price, vars.collDecimals);

334:         _requireValidAdjustmentInCurrentMode(isRecoveryMode, _collateral, _collWithdrawal, _isDebtIncrease, vars);

338:             _requireAtLeastMinNetDebt(_getNetDebt(vars.debt).sub(vars.netDebtChange));

340:             _requireSufficientLUSDBalance(contractsCache.lusdToken, _borrower, vars.netDebtChange);

343:         (vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);

344:         vars.stake = contractsCache.troveManager.updateStakeAndTotalStakes(_borrower, _collateral);

356:         sortedTroves.reInsert(_borrower, _collateral, newNICR, _upperHint, _lowerHint);

358:         emit TroveUpdated(_borrower, _collateral, vars.newDebt, vars.newColl, vars.stake, BorrowerOperation.adjustTrove);

382:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

391:         _requireSufficientLUSDBalance(lusdTokenCached, msg.sender, debt.sub(LUSD_GAS_COMPENSATION));

393:         uint newTCR = _getNewTCRFromTroveChange(_collateral, coll, false, debt, false, price, collDecimals);

397:         troveManagerCached.closeTrove(msg.sender, _collateral, 2); // 2 = closedByOwner

399:         emit TroveUpdated(msg.sender, _collateral, 0, 0, 0, BorrowerOperation.closeTrove);

402:         _repayLUSD(activePoolCached, lusdTokenCached, _collateral, msg.sender, debt.sub(LUSD_GAS_COMPENSATION));

403:         _repayLUSD(activePoolCached, lusdTokenCached, _collateral, gasPoolAddress, LUSD_GAS_COMPENSATION);

419:     function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {

420:         _troveManager.decayBaseRateFromBorrowing(); // decay the baseRate state variable

432:     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

468:         uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)

469:                                         : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);

470:         uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)

471:                                         : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);

491:             _withdrawLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange, _netDebtChange);

493:             _repayLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange);

497:             IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);

505:     function _activePoolAddColl(IActivePool _activePool, address _collateral, uint _amount) internal {

506:         IERC20(_collateral).safeIncreaseAllowance(address(_activePool), _amount);

507:         _activePool.pullCollateralFromBorrowerOperationsOrDefaultPool(_collateral, _amount);

511:     function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {

517:     function _repayLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSD) internal {

524:     function _requireValidCollateralAddress(address _collateral) internal view {

525:         require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

528:     function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {

529:         require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530:         require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

533:     function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

534:         require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

537:     function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

538:         require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

541:     function _requireTroveisActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

546:     function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

552:         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

562:             !_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),

568:         require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

617:         require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

621:         require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

624:     function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

625:         require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

628:     function _requireNewTCRisAboveCCR(uint _newTCR, uint256 _CCR) internal pure {

629:         require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

633:         require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");

636:     function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

637:         require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

641:         require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

644:      function _requireSufficientLUSDBalance(ILUSDToken _lusdToken, address _borrower, uint _debtRepayment) internal view {

645:         require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

648:     function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {

653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

675:         (uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);

677:         uint newNICR = LiquityMath._computeNominalCR(newColl, newDebt, _collateralDecimals);

697:         (uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);

699:         uint newICR = LiquityMath._computeCR(newColl, newDebt, _price, _collateralDecimals);

718:         newColl = _isCollIncrease ? _coll.add(_collChange) :  _coll.sub(_collChange);

719:         newDebt = _isDebtIncrease ? _debt.add(_debtChange) : _debt.sub(_debtChange);

741:         totalColl = _isCollIncrease ? totalColl.add(_collChange) : totalColl.sub(_collChange);

742:         totalDebt = _isDebtIncrease ? totalDebt.add(_debtChange) : totalDebt.sub(_debtChange);

744:         uint newTCR = LiquityMath._computeCR(totalColl, totalDebt, _price, _collateralDecimals);

748:     function getCompositeDebt(uint _debt) external pure override returns (uint) {

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

34:     address[] public collaterals; // for returning entire list of allowed collaterals

35:     mapping (address => Config) public collateralConfig; // for O(1) checking of collateral's validity and properties

37:     event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);

38:     event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);

53:         require(_MCRs.length == _collaterals.length, "Array lengths must match");

54:         require(_CCRs.length == _collaterals.length, "Array lenghts must match");

72:             emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);

102:     function getAllowedCollaterals() external override view returns (address[] memory) {

106:     function isCollateralAllowed(address _collateral) external override view returns (bool) {

129:         require(collateralConfig[_collateral].allowed, "Invalid collateral address");

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

14: contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

87:             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

28:     mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked

57:     event StakingGainsWithdrawn(address indexed staker, uint LUSDGain, address[] _assets, uint[] _amounts);

62:     event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);

114:             (collGainAssets, collGainAmounts) = _getPendingCollateralGain(msg.sender);

131:         emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);

147:         (address[] memory collGainAssets, uint[] memory collGainAmounts) = _getPendingCollateralGain(msg.sender);

168:         emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);

177:     function increaseF_Collateral(address _collateral, uint _collFee) external override {

181:         if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

183:         F_Collateral[_collateral] = F_Collateral[_collateral].add(collFeePerLQTYStaked);

191:         if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

199:     function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory) {

203:     function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {

208:             uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];

209:             amounts[i] = stakes[_user].mul(F_Collateral[collateral].sub(F_Collateral_Snapshot)).div(DECIMAL_PRECISION);

213:     function getPendingLUSDGain(address _user) external view override returns (uint) {

219:         uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);

226:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

230:             snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];

238:     function _sendCollGainToUser(address[] memory assets, uint[] memory amounts) internal {

252:         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

42:     bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;

44:     bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;

70:     address public governanceAddress; // can pause/unpause minting and upgrade addresses

80:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

126:         _CACHED_DOMAIN_SEPARATOR = _buildDomainSeparator(_TYPE_HASH, hashedName, hashedVersion);

148:         checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)

155:         checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)

196:     function sendToPool(address _sender,  address _poolAddress, uint256 _amount) external override {

201:     function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override {

206:     function getDeploymentStartTime() external view override returns (uint256) {

216:     function balanceOf(address account) external view override returns (uint256) {

220:     function transfer(address recipient, uint256 amount) external override returns (bool) {

226:     function allowance(address owner, address spender) external view override returns (uint256) {

230:     function approve(address spender, uint256 amount) external override returns (bool) {

235:     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {

238:         _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));

242:     function increaseAllowance(address spender, uint256 addedValue) external override returns (bool) {

243:         _approve(msg.sender, spender, _allowances[msg.sender][spender].add(addedValue));

247:     function decreaseAllowance(address spender, uint256 subtractedValue) external override returns (bool) {

248:         _approve(msg.sender, spender, _allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));

258:             return _buildDomainSeparator(_TYPE_HASH, _HASHED_NAME, _HASHED_VERSION);

279:         if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {

292:     function nonces(address owner) external view override returns (uint256) { // FOR EIP 2612

304:     function _buildDomainSeparator(bytes32 typeHash, bytes32 name, bytes32 version) private view returns (bytes32) {

305:         return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

311:     function _transfer(address sender, address recipient, uint256 amount) internal {

315:         _balances[sender] = _balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");

331:         _balances[account] = _balances[account].sub(amount, "ERC20: burn amount exceeds balance");

336:     function _approve(address owner, address spender, uint256 amount) internal {

350:             "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"

356:             "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

371:             "LUSD: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:         require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

167:     mapping (address => uint256) internal collAmounts;  // deposited collateral tracker

186:     mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct

187:     mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct

214:     mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;

233:     event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);

236:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

244:     event CommunityIssuanceAddressChanged(address _newCommunityIssuanceAddress);

247:     event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);

252:     event DepositSnapshotUpdated(address indexed _depositor, uint _P, address[] _assets, uint[] _amounts, uint _G);

255:     event CollateralGainWithdrawn(address indexed _depositor, address _collateral, uint _collAmount);

307:     function getCollateral(address _collateral) external view override returns (uint) {

332:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

339:         uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log

376:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

384:         uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log

410:     function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {

416:     function _triggerLQTYIssuance(ICommunityIssuance _communityIssuance) internal {

434:         epochToScaleToG[currentEpoch][currentScale] = epochToScaleToG[currentEpoch][currentScale].add(marginalLQTYGain);

436:         emit G_Updated(epochToScaleToG[currentEpoch][currentScale], currentEpoch, currentScale);

439:     function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

451:         uint LQTYNumerator = _LQTYIssuance.mul(DECIMAL_PRECISION).add(lastLQTYError);

454:         lastLQTYError = LQTYNumerator.sub(LQTYPerUnitStaked.mul(_totalLUSDDeposits));

466:     function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {

474:             uint LUSDLossPerUnitStaked) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, _debtToOffset, totalLUSD);

476:         _updateRewardSumAndProduct(_collateral, collGainPerUnitStaked, LUSDLossPerUnitStaked);  // updates S and P

486:     function updateRewardSum(address _collateral, uint _collToAdd) external override {

493:         (uint collGainPerUnitStaked, ) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, 0, totalLUSD);

495:         _updateRewardSumAndProduct(_collateral, collGainPerUnitStaked, 0);  // updates S

524:         uint collNumerator = _collToAdd.mul(DECIMAL_PRECISION).add(lastCollateralError_Offset[_collateral]);

528:             LUSDLossPerUnitStaked = DECIMAL_PRECISION;  // When the Pool depletes to 0, so does each deposit

531:             uint LUSDLossNumerator = _debtToOffset.mul(DECIMAL_PRECISION).sub(lastLUSDLossError_Offset);

536:             LUSDLossPerUnitStaked = (LUSDLossNumerator.div(_totalLUSDDeposits)).add(1);

537:             lastLUSDLossError_Offset = (LUSDLossPerUnitStaked.mul(_totalLUSDDeposits)).sub(LUSDLossNumerator);

541:         lastCollateralError_Offset[_collateral] = collNumerator.sub(collGainPerUnitStaked.mul(_totalLUSDDeposits));

547:     function _updateRewardSumAndProduct(address _collateral, uint _collGainPerUnitStaked, uint _LUSDLossPerUnitStaked) internal {

556:         uint newProductFactor = uint(DECIMAL_PRECISION).sub(_LUSDLossPerUnitStaked);

560:         uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral];

571:         epochToScaleToSum[currentEpochCached][currentScaleCached][_collateral] = newS;

572:         emit S_Updated(_collateral, newS, currentEpochCached, currentScaleCached);

583:         } else if (currentP.mul(newProductFactor).div(DECIMAL_PRECISION) < SCALE_FACTOR) {

584:             newP = currentP.mul(newProductFactor).mul(SCALE_FACTOR).div(DECIMAL_PRECISION);

597:     function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {

607:         activePoolCached.sendCollateral(_collateral, address(this), _collToAdd);

626:     function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {

637:     function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

641:             amounts[i] = _getSingularCollateralGain(initialDeposit, assets[i], snapshots);

657:     function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {

664:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

670:         vars.firstPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot][_collateral].sub(vars.S_Snapshot);

671:         vars.secondPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot.add(1)][_collateral].div(SCALE_FACTOR);

673:         vars.gain = _initialDeposit.mul(vars.firstPortion.add(vars.secondPortion)).div(vars.P_Snapshot).div(DECIMAL_PRECISION);

683:     function getDepositorLQTYGain(address _depositor) public view override returns (uint) {

693:     function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

704:         uint firstPortion = epochToScaleToG[epochSnapshot][scaleSnapshot].sub(G_Snapshot);

705:         uint secondPortion = epochToScaleToG[epochSnapshot][scaleSnapshot.add(1)].div(SCALE_FACTOR);

707:         uint LQTYGain = initialDeposit.mul(firstPortion.add(secondPortion)).div(P_Snapshot).div(DECIMAL_PRECISION);

718:     function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {

724:         uint compoundedDeposit = _getCompoundedDepositFromSnapshots(initialDeposit, snapshots);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

776:     function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

783:     function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {

794:     function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

803:     function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {

806:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

814:             emit DepositSnapshotUpdated(_depositor, 0, collaterals, amounts, 0);

822:         uint currentG = epochToScaleToG[currentEpochCached][currentScaleCached];

833:             uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];

837:         emit DepositSnapshotUpdated(_depositor, currentP, collaterals, amounts, currentG);

840:     function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

857:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

864:             uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

109:     mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;

186:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

200:     event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);

201:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint _stake, TroveManagerOperation _operation);

202:     event TroveLiquidated(address indexed _borrower, address _collateral, uint _debt, uint _coll, TroveManagerOperation _operation);

206:     event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);

207:     event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

208:     event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

209:     event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);

299:     function getTroveOwnersCount(address _collateral) external view override returns (uint) {

303:     function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {

310:     function liquidate(address _borrower, address _collateral) external override {

338:         _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);

341:         singleLiquidation.collGasCompensation = _getCollGasCompensation(singleLiquidation.entireTroveColl);

343:         uint collToLiquidate = singleLiquidation.entireTroveColl.sub(singleLiquidation.collGasCompensation);

348:         singleLiquidation.collToRedistribute) = _getOffsetAndRedistributionVals(singleLiquidation.entireTroveDebt, collToLiquidate, _LUSDInStabPool);

351:         emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInNormalMode);

352:         emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInNormalMode);

372:         if (TroveOwners[_collateral].length <= 1) {return singleLiquidation;} // don't liquidate if last trove

378:         singleLiquidation.collGasCompensation = _getCollGasCompensation(singleLiquidation.entireTroveColl);

380:         vars.collToLiquidate = singleLiquidation.entireTroveColl.sub(singleLiquidation.collGasCompensation);

384:             _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);

389:             singleLiquidation.debtToRedistribute = singleLiquidation.entireTroveDebt;

393:             emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);

394:             emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInRecoveryMode);

398:              _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);

404:             singleLiquidation.collToRedistribute) = _getOffsetAndRedistributionVals(singleLiquidation.entireTroveDebt, vars.collToLiquidate, _LUSDInStabPool);

407:             emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);

408:             emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInRecoveryMode);

415:         } else if ((_ICR >= _MCR) && (_ICR < _TCR) && (singleLiquidation.entireTroveDebt <= _LUSDInStabPool)) {

416:             _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);

420:             uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);

421:             singleLiquidation = _getCappedOffsetVals(singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, _price, _MCR, collDecimals);

425:                 collSurplusPool.accountSurplus(_borrower, _collateral, singleLiquidation.collSurplus);

428:             emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.collToSendToSP, TroveManagerOperation.liquidateInRecoveryMode);

429:             emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInRecoveryMode);

431:         } else { // if (_ICR >= _MCR && ( _ICR >= _TCR || singleLiquidation.entireTroveDebt > _LUSDInStabPool))

450:         returns (uint debtToOffset, uint collToSendToSP, uint debtToRedistribute, uint collToRedistribute)

494:             cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));

496:             cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));

499:         singleLiquidation.collGasCompensation = _getCollGasCompensation(cappedCollPortion);

503:         singleLiquidation.collToSendToSP = cappedCollPortion.sub(singleLiquidation.collGasCompensation);

504:         singleLiquidation.collSurplus = _entireTroveColl.sub(cappedCollPortion);

522:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

526:         vars.recoveryModeAtStart = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);

554:         stabilityPoolCached.offset(_collateral, totals.totalDebtToOffset, totals.totalCollToSendToSP);

564:             activePoolCached.sendCollateral(_collateral, address(collSurplusPool), totals.totalCollSurplus);

568:         _updateSystemSnapshots_excludeCollRemainder(activePoolCached, _collateral, totals.totalCollGasCompensation);

571:         vars.liquidatedColl = totals.totalCollInSequence.sub(totals.totalCollGasCompensation).sub(totals.totalCollSurplus);

572:         emit Liquidation(_collateral, vars.liquidatedDebt, vars.liquidatedColl, totals.totalCollGasCompensation, totals.totalLUSDGasCompensation);

575:         _sendGasCompensation(activePoolCached, _collateral, msg.sender, totals.totalLUSDGasCompensation, totals.totalCollGasCompensation);

596:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

616:                 if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

618:                 vars.TCR = LiquityMath._computeCR(vars.entireSystemColl, vars.entireSystemDebt, _price, vars.collDecimals);

633:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

634:                 vars.entireSystemDebt = vars.entireSystemDebt.sub(singleLiquidation.debtToOffset);

641:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

660:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

663:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

665:             }  else break;  // break if the loop reaches a Trove with ICR >= MCR

703:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

706:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

715:     function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {

725:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

730:         vars.recoveryModeAtStart = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);

757:         stabilityPoolCached.offset(_collateral, totals.totalDebtToOffset, totals.totalCollToSendToSP);

767:             activePoolCached.sendCollateral(_collateral, address(collSurplusPool), totals.totalCollSurplus);

771:         _updateSystemSnapshots_excludeCollRemainder(activePoolCached, _collateral, totals.totalCollGasCompensation);

774:         vars.liquidatedColl = totals.totalCollInSequence.sub(totals.totalCollGasCompensation).sub(totals.totalCollSurplus);

775:         emit Liquidation(_collateral, vars.liquidatedDebt, vars.liquidatedColl, totals.totalCollGasCompensation, totals.totalLUSDGasCompensation);

778:         _sendGasCompensation(activePoolCached, _collateral, msg.sender, totals.totalLUSDGasCompensation, totals.totalCollGasCompensation);

798:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

811:             if (Troves[vars.user][_collateral].status != Status.active) { continue; }

817:                 if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }

819:                 uint TCR = LiquityMath._computeCR(vars.entireSystemColl, vars.entireSystemDebt, _price, vars.collDecimals);

834:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

835:                 vars.entireSystemDebt = vars.entireSystemDebt.sub(singleLiquidation.debtToOffset);

842:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

854:                 singleLiquidation = _liquidateNormalMode(_activePool, _defaultPool, _collateral, vars.user, vars.remainingLUSDInStabPool);

855:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

858:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

887:                 singleLiquidation = _liquidateNormalMode(_activePool, _defaultPool, _collateral, vars.user, vars.remainingLUSDInStabPool);

888:                 vars.remainingLUSDInStabPool = vars.remainingLUSDInStabPool.sub(singleLiquidation.debtToOffset);

891:                 totals = _addLiquidationValuesToTotals(totals, singleLiquidation);

898:     function _addLiquidationValuesToTotals(LiquidationTotals memory oldTotals, LiquidationValues memory singleLiquidation)

902:         newTotals.totalCollGasCompensation = oldTotals.totalCollGasCompensation.add(singleLiquidation.collGasCompensation);

903:         newTotals.totalLUSDGasCompensation = oldTotals.totalLUSDGasCompensation.add(singleLiquidation.LUSDGasCompensation);

904:         newTotals.totalDebtInSequence = oldTotals.totalDebtInSequence.add(singleLiquidation.entireTroveDebt);

905:         newTotals.totalCollInSequence = oldTotals.totalCollInSequence.add(singleLiquidation.entireTroveColl);

906:         newTotals.totalDebtToOffset = oldTotals.totalDebtToOffset.add(singleLiquidation.debtToOffset);

907:         newTotals.totalCollToSendToSP = oldTotals.totalCollToSendToSP.add(singleLiquidation.collToSendToSP);

908:         newTotals.totalDebtToRedistribute = oldTotals.totalDebtToRedistribute.add(singleLiquidation.debtToRedistribute);

909:         newTotals.totalCollToRedistribute = oldTotals.totalCollToRedistribute.add(singleLiquidation.collToRedistribute);

910:         newTotals.totalCollSurplus = oldTotals.totalCollSurplus.add(singleLiquidation.collSurplus);

915:     function _sendGasCompensation(IActivePool _activePool, address _collateral, address _liquidator, uint _LUSD, uint _collAmount) internal {

926:     function _movePendingTroveRewardsToActivePool(IActivePool _activePool, IDefaultPool _defaultPool, address _collateral, uint _LUSD, uint _collAmount) internal {

952:         activePool.sendCollateral(_collateral, address(collSurplusPool), _collAmount);

954:         emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.redeemCollateral);

957:     function reInsert(address _id, address _collateral, uint256 _newNICR, address _prevId, address _nextId) external override {

992:         emit Redemption(_collateral, _attemptedLUSDAmount, _actualLUSDAmount, _collSent, _collFee);

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1046:         (uint currentCollateral, uint currentLUSDDebt) = _getCurrentTroveAmounts(_borrower, _collateral);

1048:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1049:         uint NICR = LiquityMath._computeNominalCR(currentCollateral, currentLUSDDebt, collDecimals);

1059:         (uint currentCollateral, uint currentLUSDDebt) = _getCurrentTroveAmounts(_borrower, _collateral);

1061:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1062:         uint ICR = LiquityMath._computeCR(currentCollateral, currentLUSDDebt, _price, collDecimals);

1066:     function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {

1067:         uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);

1068:         uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);

1070:         uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);

1071:         uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

1076:     function applyPendingRewards(address _borrower, address _collateral) external override {

1078:         return _applyPendingRewards(activePool, defaultPool, _borrower, _collateral);

1082:     function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

1087:             uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);

1088:             uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);

1091:             Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);

1092:             Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

1097:             _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, pendingLUSDDebtReward, pendingCollateralReward);

1111:     function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {

1116:     function _updateTroveRewardSnapshots(address _borrower, address _collateral) internal {

1117:         rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];

1118:         rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];

1119:         emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1123:     function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1124:         uint snapshotCollateral = rewardSnapshots[_borrower][_collateral].collAmount;

1125:         uint rewardPerUnitStaked = L_Collateral[_collateral].sub(snapshotCollateral);

1127:         if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }

1131:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1132:         uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1138:     function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1139:         uint snapshotLUSDDebt = rewardSnapshots[_borrower][_collateral].LUSDDebt;

1140:         uint rewardPerUnitStaked = L_LUSDDebt[_collateral].sub(snapshotLUSDDebt);

1142:         if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }

1146:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1147:         uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1152:     function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {

1158:         if (Troves[_borrower][_collateral].status != Status.active) {return false;}

1160:         return (rewardSnapshots[_borrower][_collateral].collAmount < L_Collateral[_collateral]);

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1176:         pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);

1177:         pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);

1183:     function removeStake(address _borrower, address _collateral) external override {

1195:     function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {

1201:     function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {

1202:         uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);

1206:         totalStakes[_collateral] = totalStakes[_collateral].sub(oldStake).add(newStake);

1213:     function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1225:             stake = _coll.mul(totalStakesSnapshot[_collateral]).div(totalCollateralSnapshot[_collateral]);

1251:         uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);

1252:         uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);

1255:         uint collateralRewardPerUnitStaked = collateralNumerator.div(totalStakes[_collateral]);

1256:         uint LUSDDebtRewardPerUnitStaked = LUSDDebtNumerator.div(totalStakes[_collateral]);

1258:         lastCollateralError_Redistribution[_collateral] = collateralNumerator.sub(collateralRewardPerUnitStaked.mul(totalStakes[_collateral]));

1259:         lastLUSDDebtError_Redistribution[_collateral] = LUSDDebtNumerator.sub(LUSDDebtRewardPerUnitStaked.mul(totalStakes[_collateral]));

1262:         L_Collateral[_collateral] = L_Collateral[_collateral].add(collateralRewardPerUnitStaked);

1263:         L_LUSDDebt[_collateral] = L_LUSDDebt[_collateral].add(LUSDDebtRewardPerUnitStaked);

1265:         emit LTermsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);

1273:     function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {

1278:     function _closeTrove(address _borrower, address _collateral, Status closedStatus) internal {

1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1305:     function _updateSystemSnapshots_excludeCollRemainder(IActivePool _activePool, address _collateral, uint _collRemainder) internal {

1310:         totalCollateralSnapshot[_collateral] = activeColl.sub(_collRemainder).add(liquidatedColl);

1312:         emit SystemSnapshotsUpdated(_collateral, totalStakesSnapshot[_collateral], totalCollateralSnapshot[_collateral]);

1316:     function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {

1321:     function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

1323:         debt of liquidation reserve plus MIN_NET_DEBT. 3e30 LUSD dwarfs the value of all wealth in the world ( which is < 1e15 USD). */

1339:     function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {

1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1361:     function getTCR(address _collateral, uint _price) external view override returns (uint) {

1362:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1366:     function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {

1368:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1384:         uint TCR = LiquityMath._computeCR(_entireSystemColl, _entireSystemDebt, _price, _collDecimals);

1409:             LiquityMath._getScaledCollAmount(_collateralDrawn, _collDecimals).mul(_price).div(_totalLUSDSupply);

1412:         newBaseRate = LiquityMath._min(newBaseRate, DECIMAL_PRECISION); // cap baseRate at a maximum of 100%

1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1444:     function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1445:         return _calcRedemptionFee(getRedemptionRateWithDecay(), _collateralDrawn);

1448:     function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {

1449:         uint redemptionFee = _redemptionRate.mul(_collateralDrawn).div(DECIMAL_PRECISION);

1471:     function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {

1475:     function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {

1479:     function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {

1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

1511:         uint decayFactor = LiquityMath._decPow(MINUTE_DECAY_FACTOR, minutesPassed);

1517:         return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);

1530:     function _requireCallerIsBorrowerOperationsOrRedemptionHelper() internal view {

1531:         require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));

1534:     function _requireTroveIsActive(address _borrower, address _collateral) internal view {

1538:     function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {

1539:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

1544:     function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {

1548:     function getTroveStake(address _borrower, address _collateral) external view override returns (uint) {

1552:     function getTroveDebt(address _borrower, address _collateral) external view override returns (uint) {

1556:     function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {

1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

1567:     function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {

1574:     function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

1581:     function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {

1588:     function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

19: contract ReaperStrategyGranarySupplyOnly is ReaperBaseStrategyv4, VeloSolidMixin {

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

26:         ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

104:     function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

114:     function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {

179:             IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest);

180:             ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE);

200:         ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this));

207:         IRewardsController(REWARDER).claimAllRewardsToSelf(rewardClaimingTokens);

231:         (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

29:     function asset() external view override returns (address assetTokenAddress) {

37:     function totalAssets() external view override returns (uint256 totalManagedAssets) {

51:     function convertToShares(uint256 assets) public view override returns (uint256 shares) {

66:     function convertToAssets(uint256 shares) public view override returns (uint256 assets) {

79:     function maxDeposit(address) external view override returns (uint256 maxAssets) {

96:     function previewDeposit(uint256 assets) external view override returns (uint256 shares) {

110:     function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

122:     function maxMint(address) external view override returns (uint256 maxShares) {

139:     function previewMint(uint256 shares) public view override returns (uint256 assets) {

154:     function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

155:         assets = previewMint(shares); // previewMint rounds up so exactly "shares" should be minted and not 1 wei less

165:     function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {

183:     function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {

207:         shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less

220:     function maxRedeem(address owner) external view override returns (uint256 maxShares) {

240:     function previewRedeem(uint256 shares) external view override returns (uint256 assets) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

32:         uint256 lastReport; // block.timestamp of the last time a report occured

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

44:     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)

45:     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies

46:     uint256 public lastReport; // block.timestamp of last report from any strategy

56:     uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block

57:     uint256 public lockedProfit; // how much profit is locked and cant be withdrawn

78:     address public treasury; // address to whom performance fee is remitted in the form of vault shares

80:     event StrategyAdded(address indexed strategy, uint256 feeBPS, uint256 allocBPS);

125:         lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

150:         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

152:         require(strategies[_strategy].activation == 0, "Strategy already added");

153:         require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

154:         require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");

155:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

156:         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

178:     function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {

180:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

181:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

191:     function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {

193:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

231:         uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;

237:             uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;

245:             available = Math.min(available, vaultMaxAllocation - vaultCurrentAllocation);

296:         return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();

319:     function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {

321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

334:             shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"

385:                 uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));

386:                 uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;

405:                 totalLoss <= ((value + totalLoss) * withdrawMaxLoss) / PERCENT_DIVISOR,

419:         uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;

421:             return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

436:         require(loss <= allocation, "Strategy loss cannot be greater than allocation");

440:             uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);

462:     function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {

463:         uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;

466:             uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();

493:     function report(int256 _roi, uint256 _repayment) external returns (uint256) {

526:             token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);

528:             token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);

533:         vars.lockedProfitBeforeLoss = _calculateLockedProfit() + vars.gain - vars.fees;

619:         require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

659:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

673:     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";

12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF

81:         require(_multisigRoles.length == 3, "Invalid number of multisig roles");

95:     function withdraw(uint256 _amount) external override returns (uint256 loss) {

203:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

217:     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {

250:     function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);

276:     function _harvestCore(uint256 _debt) internal virtual returns (int256 roi, uint256 repayment);

```

### [NC-31] USE UNDERSCORES FOR NUMBER LITERALS

#### Recommended Mitigation Steps

Consider using underscores for number literals to improve its readability.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/TroveManager.sol

53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

41:     uint256 public constant PERCENT_DIVISOR = 10000;

```

### [NC-32] VARIABLE NAMES THAT CONSIST OF ALL CAPITAL LETTERS SHOULD BE RESERVED FOR CONSTANT/IMMUTABLE VARIABLES

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

74:         uint ICR;

75:         uint NICR;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

30:         uint256 MCR;

31:         uint256 CCR;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

29:     uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

179:         mapping (address => uint) S;

180:         uint P;

181:         uint G;

195:     uint public P = DECIMAL_PRECISION;

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

149:         uint ICR;

150:         uint TCR;

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | DOS WITH BLOCK GAS LIMIT                                                       |
| L-2  | USAGE OF `PAYABLE.TRANSFER` CAN LEAD TO LOSS OF FUNDS                          |
| L-3  | AVOID USING LOW CALL FUNCTION `ECRECOVER`                                      |
| L-4  | ACRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                                |
| L-5  | DO NOT USE DEPRECATED LIBRARY FUNCTIONS                                        |
| L-6  | DON’T USE `OWNER`                                                              |
| L-7  | HARDCODE THE ADDRESS CAUSES NO FUTURE UPDATES                                  |
| L-8  | `INITIALIZE()` FUNCTION CAN BE CALLED BY ANYBODY                               |
| L-9  | LOSS OF PRECISION DUE TO ROUNDING                                              |
| L-10 | PIREXERC4626’S IMPLMENTATION IS NOT FULLY UP TO EIP-4626’S SPECIFICATION       |
| L-11 | MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS     |
| L-12 | MISSING REENTRANCY GUARD TO WITHDRAW FUNCTION                                  |
| L-13 | `REQUIRE()/REVERT()` STRINGS LONGER THAN 32 BYTES COST EXTRA GAS               |
| L-14 | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-15 | A SINGLE POINT OF FAILURE                                                      |
| L-16 | USE OF `BLOCK.TIMESTAMP`                                                       |
| L-17 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                             |
| L-18 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                       |
| L-19 | POTENTIAL DOS IN CONTRACT INHERITING UUPSUPGRADEABLE.SOL                       |

### [L-1] DOS WITH BLOCK GAS LIMIT

#### Description:

Programming patterns such as looping over arrays of unknown size may lead to DoS when the gas cost of execution exceeds the block gas limit.

Reference: https://swcregistry.io/docs/SWC-128

This loops could drain all user gas and revert.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

47:         address[] calldata _collaterals,

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

332:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

792:         address[] memory _troveArray

872:         address[] memory _troveArray

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160:     function setHarvestSteps(address[2][] calldata _newSteps) external {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

117:         address[] memory _strategists,

258:     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {

659:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

66:         address[] memory _strategists,

```

### [L-2] USAGE OF `PAYABLE.TRANSFER` CAN LEAD TO LOSS OF FUNDS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

The funds that are to be sent can be lost. The issues with `transfer()` are outlined here:
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

127:         OathToken.transfer(_account, _OathAmount);

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

135:             lusdToken.transfer(msg.sender, LUSDGain);

171:         lusdToken.transfer(msg.sender, LUSDGain);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-3] AVOID USING LOW CALL FUNCTION `ECRECOVER`

#### Description:

Use OZ library ECDSA that its battle tested to avoid classic errors.

https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

287:         address recoveredAddress = ecrecover(digest, v, r, s);

```

### [L-4] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

71:     function setAddresses(

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

110:     function setAddresses(

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:     function setAddresses

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:     function setAddresses(

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

232:     function setAddresses(

1562:     function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160:     function setHarvestSteps(address[2][] calldata _newSteps) external {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

258:     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-5] DO NOT USE DEPRECATED LIBRARY FUNCTIONS

#### Description:

You use `safeApprove `of openZeppelin although it’s deprecated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38%3E).

You should change it to increase/decrease Allowance as OpenZeppilin says.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

74:         IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

```

### [L-6] DON’T USE `OWNER`

#### Description:

The owner manipulates the contract without a lock time period.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/TroveManager.sol

250:         require(msg.sender == owner);

```

### [L-7] HARDCODE THE ADDRESS CAUSES NO FUTURE UPDATES

#### Description:

Router etc. In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the `onlyOwner` modifier.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

124:             _swapVelo(step[0], step[1], amount, VELO_ROUTER);

```

### [L-8] `INITIALIZE()` FUNCTION CAN BE CALLED BY ANYBODY

#### Description:

`initialize()` function can be called anybody when the contract is not initialized.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:      function initialize(
63:        address _vault,
64:        address[] memory _strategists,
65:        address[] memory _multisigRoles,
66:        IAToken _gWant
67:    ) public initializer {
68:        gWant = _gWant;
69:        want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:    }

```

#### Recommended Mitigation Steps

Add a control that makes `initialize()` only call the Deployer Contract or EOA;

```solidity
if (msg.sender != DEPLOYER_ADDRESS) {
            revert NotDeployer();
        }
```

### [L-9] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

53:         return (assets * totalSupply()) / _freeFunds();

53:         return (assets * totalSupply()) / _freeFunds();

68:         return (shares * _freeFunds()) / totalSupply();

68:         return (shares * _freeFunds()) / totalSupply();

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

125:         lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

231:         uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;

237:             uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;

334:             shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"

365:         value = (_freeFunds() * _shares) / totalSupply();

405:                 totalLoss <= ((value + totalLoss) * withdrawMaxLoss) / PERCENT_DIVISOR,

421:             return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

440:             uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);

463:         uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;

```

### [L-10] PIREXERC4626’S IMPLMENTATION IS NOT FULLY UP TO EIP-4626’S SPECIFICATION

#### Description:

Must return the maximum amount of shares mint would allow to be deposited to receiver and not cause a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary).

This assumes that the user has infinite assets, i.e. must not rely on balanceOf of asset.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

79:     function maxDeposit(address) external view override returns (uint256 maxAssets) {

122:     function maxMint(address) external view override returns (uint256 maxShares) {

```

#### Recommended Mitigation Steps:

maxMint() and maxDeposit() should reflect the limitation of maxSupply.

### [L-11] MISSING CALLS TO \_\_REENTRANCYGUARD_INIT FUNCTIONS OF INHERITED CONTRACTS

#### Description:

Most contracts use the delegateCall proxy pattern and hence their implementations require the use of `initialize()` functions instead of constructors. This requires derived contracts to call the corresponding init functions of their inherited base contracts. This is done in most places except a few.

Impact: The inherited base classes do not get initialized which may lead to undefined behavior.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:     function initialize(

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:     function initialize(

```

#### Recommended Mitigation Steps:

Add missing calls to init functions of inherited contracts.

### [L-12] MISSING REENTRANCY GUARD TO WITHDRAW FUNCTION

#### Description:

ReaperBaseStrategyV4.sol contract has no Re-Entrancy protection in withdraw function.

It would be worthwhile to ensure the proxy contract is deployed and initialized in the same transaction, or ensure the `initialize()` function is callable only by the deployer of the proxy contract. This could be set in the proxy contracts `constructor()`.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

95:    function withdraw(uint256 _amount) external override returns (uint256 loss) {
96:        require(msg.sender == vault, "Only vault can withdraw");
98:        require(_amount != 0, "Amount cannot be zero");
98:        require(_amount <= balanceOf(), "Ammount must be less than balance");
99:
100:        uint256 amountFreed = 0;
101:        (amountFreed, loss) = _liquidatePosition(_amount);
102:        IERC20Upgradeable(want).safeTransfer(vault, amountFreed);
103:    }

```

#### Recommended Mitigation Steps:

Use Openzeppelin or Solmate Re-Entrancy pattern.

### [L-13] `REQUIRE()/REVERT()` STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

#### Description:

Error definitions should be added to the require block, not exceeding 32 bytes.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

525:         require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

529:         require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530:         require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

534:         require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

538:         require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

543:         require(status == 1, "BorrowerOps: Trove does not exist or is closed");

552:         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

568:         require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

617:         require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

621:         require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

625:         require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

629:         require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

637:         require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

641:         require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

645:         require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

394:         require(!mintingPaused, "LUSDToken: Minting is currently paused");

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

150:         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

436:         require(loss <= allocation, "Strategy loss cannot be greater than allocation");

619:         require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

```

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

### [L-14] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

186:             IERC20(_collateral).safeTransfer(_account, _amount);

293:                     IERC20(_collateral).safeTransfer(treasuryAddress, vars.treasurySplit);

298:                     IERC20(_collateral).safeTransfer(lqtyStakingAddress, vars.stakingSplit);

304:                     IERC20(_collateral).safeTransfer(stabilityPoolAddress, vars.stabilityPoolSplit);

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

163:             lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);

244:                 IERC20(collateral).safeTransfer(msg.sender, amounts[i]);

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

790:         IERC20(_collateral).safeTransfer(msg.sender, _amount);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

410:         token.safeTransfer(_receiver, value);

526:             token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);

642:         IERC20Metadata(_token).safeTransfer(msg.sender, amount);

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

102:         IERC20Upgradeable(want).safeTransfer(vault, amountFreed);

```

### [L-15] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and `onlyOwner` can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

71:      function setAddresses(

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

110:         function setAddresses(onlyOwner

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:     function initialize(

85:     function updateCollateralRatios(

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61:      function setAddresses

101:     function fund(uint amount) external onlyOwner {

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:         function setAddresses

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:       function setAddresses(

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-16] USE OF `BLOCK.TIMESTAMP`

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as `block.timestamp` can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of `block.timestamp`, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers can't rely on the preciseness of the provided timestamp.

Reference: https://swcregistry.io/docs/SWC-116
Reference: https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

87:             uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

93:         lastIssuanceTimestamp = block.timestamp;

113:         lastDistributionTime = block.timestamp.add(distributionPeriod);

114:         lastIssuanceTimestamp = block.timestamp;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

128:         deploymentStartTime = block.timestamp;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1501:         uint timePassed = block.timestamp.sub(lastFeeOperationTime);

1504:             lastFeeOperationTime = block.timestamp;

1505:             emit LastFeeOpTimeUpdated(block.timestamp);

1517:         return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

121:         constructionTime = block.timestamp;

122:         lastReport = block.timestamp;

159:             activation: block.timestamp,

165:             lastReport: block.timestamp

419:         uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;

540:         strategy.lastReport = block.timestamp;

541:         lastReport = block.timestamp;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

137:         lastHarvestTimestamp = block.timestamp;

169:         upgradeProposalTime = block.timestamp;

181:         upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;

192:             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,

```

### [L-17] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

188:         _mint(_account, _amount);

320:     function _mint(address account, uint256 amount) internal {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

336:         _mint(_receiver, shares);

467:             _mint(treasury, shares);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

### [L-18] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s `transfer()` and `transferFrom()` functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

135:             lusdToken.transfer(msg.sender, LUSDGain);

171:         lusdToken.transfer(msg.sender, LUSDGain);

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.

### [L-19] POTENTIAL DOS IN CONTRACT INHERITING UUPSUPGRADEABLE.SOL

#### Description:

ReaperStrategyGranarySupplyOnly.sol inherits ReaperBaseStrategyv4.sol that inherits UUPSUpgradeable.sol. It is important that the contract is deployed and initialized in the same transaction to avoid any malicious frontrunning. ReaperStrategyGranarySupplyOnly.sol may potentially have initialized variable be false, and if this happens, the malicious attacker would take over the control.

It would be worthwhile to ensure the proxy contract is deployed and initialized in the same transaction, or ensure the initialize() function is callable only by the deployer of the proxy contract. This could be set in the proxy contracts constructor().
