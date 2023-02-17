## [G-01] Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::108 => for(uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::56 => for(uint256 i = 0; i < _collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::206 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::228 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::240 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::72 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::78 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::101 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::107 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::113 => for (uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::351 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::397 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::640 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::810 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::831 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::859 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::117 => for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::165 => for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::128 => for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::264 => for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::369 => uint256 totalLoss = 0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::371 => uint256 vaultBalance = 0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::372 => for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
```

## [G-02] Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::56 => for(uint256 i = 0; i < _collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::206 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::228 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::640 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::810 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::831 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::808 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::882 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

## [G-03] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::552 => require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::266 => require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::270 => require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::179 => require(totals.totalCollateralDrawn > 0);
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::317 => require(_amount > 0);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::115 => require(_NICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::216 => require(_newNICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::870 => require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::874 => require(_amount > 0, 'StabilityPool: Amount must be non-zero');
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::551 => require(totals.totalDebtInSequence > 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::754 => require(totals.totalDebtInSequence > 0);
```

## [G-04] Long Revert Strings

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::107 => require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::133 => require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::525 => require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::529 => require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::530 => require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::534 => require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::538 => require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::543 => require(status == 1, "BorrowerOps: Trove does not exist or is closed");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::552 => require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::568 => require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::617 => require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::621 => require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::625 => require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::629 => require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::633 => require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::637 => require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::641 => require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::645 => require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::127 => require(msg.sender == activePoolAddress, "DefaultPool: Caller is not the ActivePool");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::131 => require(msg.sender == troveManagerAddress, "DefaultPool: Caller is not the TroveManager");
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::133 => require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::262 => require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::362 => require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::377 => require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::390 => require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::394 => require(!mintingPaused, "LUSDToken: Minting is currently paused");
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::296 => require(msg.sender == address(troveManager), "RedemptionHelper: Caller is not TroveManager");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::111 => require(!contains(_collateral, _id), "SortedTroves: List already contains the node");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::115 => require(_NICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::166 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::214 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::216 => require(_newNICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::403 => require(msg.sender == address(troveManager), "SortedTroves: Caller is not the TroveManager");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::849 => require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::853 => require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::865 => require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::150 => require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::321 => require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::436 => require(loss <= allocation, "Strategy loss cannot be greater than allocation");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::619 => require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");
```

## [G-05] Use Shift Right/Left instead of Division/Multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::44 => uint constant public TIMEOUT = 14400;  // 4 hours: 60 * 60 * 4
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::51 => * (1/2) = d^720 => d = (1/2)^(1/720)
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::125 => lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

## [G-06] Use calldata instead of memory

Use calldata instead of memory for function parameters saves gas if the function argument is only read.

```
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::407 => function _chainlinkIsBroken(ChainlinkResponse memory _currentResponse, ChainlinkResponse memory _prevResponse) internal view returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::411 => function _badChainlinkResponse(ChainlinkResponse memory _response) internal view returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::424 => function _chainlinkIsFrozen(ChainlinkResponse memory _response) internal view returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::428 => function _chainlinkPriceChangeAboveMax(ChainlinkResponse memory _currentResponse, ChainlinkResponse memory _prevResponse) internal pure returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::446 => function _tellorIsBroken(TellorResponse memory _response) internal view returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::457 => function _tellorIsFrozen(TellorResponse  memory _tellorResponse) internal view returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::486 => function _bothOraclesSimilarPrice( ChainlinkResponse memory _chainlinkResponse, TellorResponse memory _tellorResponse) internal pure returns (bool) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::541 => function _storeTellorPrice(address _collateral, TellorResponse memory _tellorResponse) internal returns (uint) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::548 => function _storeChainlinkPrice(address _collateral, ChainlinkResponse memory _chainlinkResponse) internal returns (uint) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::693 => function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
```

## [G-07] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::48 => uint256 public immutable constructionTime;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::52 => IERC20Metadata public immutable token;
```

## [G-08] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::125 => function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::132 => function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::138 => function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::144 => function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::214 => function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::101 => function fund(uint amount) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::120 => function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::80 => function setParams(address _troveManagerAddress, address _borrowerOperationsAddress) external override onlyOwner {
```

## [G-09] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::35 => uint8 constant internal _DECIMALS = 18;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::59 => uint80 roundId;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::63 => uint8 decimals;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::182 => uint128 scale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::183 => uint128 epoch;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::200 => uint128 public currentScale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::203 => uint128 public currentEpoch;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::558 => uint128 currentScaleCached = currentScale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::559 => uint128 currentEpochCached = currentEpoch;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::648 => uint128 epochSnapshot;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::649 => uint128 scaleSnapshot;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::699 => uint128 epochSnapshot = snapshots.epoch;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::700 => uint128 scaleSnapshot = snapshots.scale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::738 => uint128 scaleSnapshot = snapshots.scale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::739 => uint128 epochSnapshot = snapshots.epoch;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::745 => uint128 scaleDiff = currentScale.sub(scaleSnapshot);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::817 => uint128 currentScaleCached = currentScale;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::818 => uint128 currentEpochCached = currentEpoch;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::82 => uint128 arrayIndex;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1344 => uint128 index = Troves[_borrower][_collateral].arrayIndex;
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::35 => uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
```

## [G-10] Using bools for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
Use uint256(1) and uint256(2) for true/false instead

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::32 => bool public addressesSet = false;
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::18 => bool public initialized = false;
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::21 => bool public initialized = false;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::37 => bool public mintingPaused = false;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::62 => mapping (address => bool) public troveManagers;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::63 => mapping (address => bool) public stabilityPools;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::64 => mapping (address => bool) public borrowerOperations;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::29 => bool public initialized = false;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::49 => bool public emergencyShutdown;
```

## [G-11] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, for example when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::108 => for(uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::56 => for(uint256 i = 0; i < _collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::191 => i++;
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::206 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::228 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::240 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::286 => _nonces[owner]++, deadline))));
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::72 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::78 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::101 => for (uint idx = 0; idx < _startIdx; ++idx) {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::107 => for (uint idx = 0; idx < _count; ++idx) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::113 => for (uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::351 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::397 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::640 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::810 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::831 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::859 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::608 => for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::690 => for (vars.i = 0; vars.i < _n; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::808 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::882 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::273 => if (x % y != 0) q++;
```

## [G-12] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

use <x> = <x> + <y> or <x> = <x> - <y> instead to save gas

```
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::133 => toFree += profit;
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::141 => roi -= int256(loss);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::168 => totalAllocBPS += _allocBPS;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::194 => totalAllocBPS -= strategies[_strategy].allocBPS;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::196 => totalAllocBPS += _allocBPS;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::214 => totalAllocBPS -= strategies[_strategy].allocBPS;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::390 => value -= loss;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::391 => totalLoss += loss;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::395 => strategies[stratAddr].allocated -= actualWithdrawn;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::396 => totalAllocated -= actualWithdrawn;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::444 => stratParams.allocBPS -= bpsChange;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::445 => totalAllocBPS -= bpsChange;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::450 => stratParams.losses += loss;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::451 => stratParams.allocated -= loss;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::452 => totalAllocated -= loss;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::505 => strategy.gains += vars.gain;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::514 => strategy.allocated -= vars.debtPayment;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::515 => totalAllocated -= vars.debtPayment;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::516 => vars.debt -= vars.debtPayment; // tracked for return value
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::520 => strategy.allocated += vars.credit;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::521 => totalAllocated += vars.credit;
```

## [G-13] abi.encode() is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

```
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::284 => domainSeparator(), keccak256(abi.encode(
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::305 => return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
```

## [G-14] Use custom errors rather than revert()/require() strings to save gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hitby avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::85 => require(!addressesSet, "Can call setAddresses only once");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::93 => require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::107 => require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::111 => require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::127 => require(_bps <= 10_000, "Invalid BPS value");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::133 => require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::145 => require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::525 => require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::529 => require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::530 => require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::534 => require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::538 => require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::543 => require(status == 1, "BorrowerOps: Trove does not exist or is closed");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::548 => require(status != 1, "BorrowerOps: Trove is active");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::552 => require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::568 => require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::617 => require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::621 => require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::625 => require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::629 => require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::633 => require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::637 => require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::641 => require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::645 => require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::97 => require(claimableColl > 0, "CollSurplusPool: No collateral available to claim");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::51 => require(!initialized, "Can only initialize once");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::52 => require(_collaterals.length != 0, "At least one collateral required");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::53 => require(_MCRs.length == _collaterals.length, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::54 => require(_CCRs.length == _collaterals.length, "Array lenghts must match");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::66 => require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::69 => require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::91 => require(_MCR <= config.MCR, "Can only walk down the MCR");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::92 => require(_CCR <= config.CCR, "Can only walk down the CCR");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::94 => require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::97 => require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::129 => require(collateralConfig[_collateral].allowed, "Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::127 => require(msg.sender == activePoolAddress, "DefaultPool: Caller is not the ActivePool");
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::131 => require(msg.sender == troveManagerAddress, "DefaultPool: Caller is not the TroveManager");
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::206 => require(collateralConfig.isCollateralAllowed(_collateral),"Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::70 => require(!initialized, "issuance has been initialized");
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::102 => require(amount != 0, "cannot fund 0");
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::133 => require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::262 => require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::362 => require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::377 => require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::390 => require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::394 => require(!mintingPaused, "LUSDToken: Minting is currently paused");
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::36 => require(collateralConfig.isCollateralAllowed(_collateral),"Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::99 => require(!initialized, "Can only initialize once");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::106 => require(numCollaterals != 0, "At least one collateral required");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::107 => require(_priceAggregatorAddresses.length == numCollaterals, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::108 => require(_tellorQueryIds.length == numCollaterals, "Array lengths must match");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::119 => require(queryId != bytes32(0), "Invalid Tellor Query ID");
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::642 => require(collateralConfig.isCollateralAllowed(_collateral), "Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::296 => require(msg.sender == address(troveManager), "RedemptionHelper: Caller is not TroveManager");
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::300 => require(collateralConfig.isCollateralAllowed(_collateral), "Invalid collateral address");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::111 => require(!contains(_collateral, _id), "SortedTroves: List already contains the node");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::113 => require(_id != address(0), "SortedTroves: Id cannot be zero");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::115 => require(_NICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::166 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::214 => require(contains(_collateral, _id), "SortedTroves: List does not contain the id");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::216 => require(_newNICR > 0, "SortedTroves: NICR must be positive");
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::403 => require(msg.sender == address(troveManager), "SortedTroves: Caller is not the TroveManager");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::849 => require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::853 => require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::865 => require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::270 => require(y != 0, "Division by 0");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::150 => require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::151 => require(_strategy != address(0), "Invalid strategy address");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::152 => require(strategies[_strategy].activation == 0, "Strategy already added");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::153 => require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::154 => require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::155 => require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::156 => require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::180 => require(strategies[_strategy].activation != 0, "Invalid strategy address");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::181 => require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::193 => require(strategies[_strategy].activation != 0, "Invalid strategy address");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::197 => require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::261 => require(queueLength != 0, "Queue must not be empty");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::267 => require(params.activation != 0, "Invalid strategy address");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::321 => require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::322 => require(_amount != 0, "Invalid amount");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::324 => require(pool + _amount <= tvlCap, "Vault is full");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::364 => require(_shares != 0, "Invalid amount");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::436 => require(loss <= allocation, "Strategy loss cannot be greater than allocation");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::497 => require(strategy.activation != 0, "Unauthorized strategy");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::568 => require(_withdrawMaxLoss <= PERCENT_DIVISOR, "Invalid BPS value");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::619 => require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::629 => require(newTreasury != address(0), "Invalid address");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::639 => require(_token != address(token), "!token");
```

## [G-15] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.
Consequences: each usage of a constant costs more gas on each access. Since these are not real constants, they can't be referenced from a real constant environment (e.g. from assembly, or from another library)

```
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::54 => uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::55 => uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::40 => uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
```

## [G-16] Use a more recent version of solidity

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/GasPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
```

## [G-17] Prefix increments cheaper than Postfix increments

++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)
Saves 5 gas PER LOOP

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::108 => for(uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::56 => for(uint256 i = 0; i < _collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::191 => i++;
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::206 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::228 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::240 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::113 => for (uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::351 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::397 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::640 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::810 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::831 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::859 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::608 => for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::690 => for (vars.i = 0; vars.i < _n; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::808 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::882 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

## [G-18] Use bytes32 instead of string

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::30 => string constant public NAME = "ActivePool";
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::21 => string constant public NAME = "BorrowerOperations";
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::17 => string constant public NAME = "CollSurplusPool";
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::25 => string constant public NAME = "DefaultPool";
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::13 => string constant public NAME = "HintHelpers";
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::19 => string constant public NAME = "CommunityIssuance";
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::23 => string constant public NAME = "LQTYStaking";
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::32 => string constant internal _NAME = "LUSD Stablecoin";
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::33 => string constant internal _SYMBOL = "LUSD";
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::34 => string constant internal _VERSION = "1";
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::27 => string constant public NAME = "PriceFeed";
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::49 => string constant public NAME = "SortedTroves";
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::150 => string constant public NAME = "StabilityPool";
```

## [G-19] Splitting require() statements that use && saves gas

Saves 16 gas per instance.
If you're using the Optimizer at 200, instead of using the && operator in a single require statement to check multiple conditions, multiple require statements with 1 condition per require statement should be used to save gas:

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::653 => require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::131 => require(!_chainlinkIsBroken(chainlinkResponse, prevChainlinkResponse) && !_chainlinkIsFrozen(chainlinkResponse),
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::158 => require(!_chainlinkIsBroken(chainlinkResponse, prevChainlinkResponse) && !_chainlinkIsFrozen(chainlinkResponse),
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::304 => require(_maxFeePercentage >= troveManager.REDEMPTION_FEE_FLOOR() && _maxFeePercentage <= DECIMAL_PRECISION);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1539 => require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

## [G-20] Usage of assert() instead of require()

Between solc 0.4.10 and 0.8.0, require() used REVERT (0xfd) opcode which refunded remaining gas on failure while assert() used INVALID (0xfe) opcode which consumed all the supplied gas.
require() should be used for checking error conditions on inputs and return values while assert() should be used for invariant checking (properly functioning code should never reach a failing assert statement, unless there is a bug in your contract you should fix).

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::128 => assert(MIN_NET_DEBT > 0);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::197 => assert(vars.compositeDebt > 0);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::301 => assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::331 => assert(_collWithdrawal <= vars.coll);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::312 => assert(sender != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::313 => assert(recipient != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::321 => assert(account != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::329 => assert(account != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::337 => assert(owner != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::338 => assert(spender != address(0));
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::128 => assert(lusdToken.balanceOf(_redeemer) <= totals.totalLUSDSupplyAtStart);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::526 => assert(_debtToOffset <= _totalLUSDDeposits);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::551 => assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::591 => assert(newP > 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::417 => assert(_LUSDInStabPool != 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1219 => * The following assert() holds true because:
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1224 => assert(totalStakesSnapshot[_collateral] > 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1279 => assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1342 => assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1348 => assert(index <= idxLast);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1413 => //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1414 => assert(newBaseRate > 0); // Base rate is always non-zero after redemption
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1489 => assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```

## [G-21] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::21 => function upgrade(address new_address) public restricted {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1045 => function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1440 => function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::295 => function getPricePerFullShare() public view returns (uint256) {
```

## [G-22] Not using the named return variables when a function returns, wastes deployment gas

It is not necessary to have both a named return and a return statement.

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::466 => returns (uint, uint)
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::713 => returns (uint, uint)
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::162 => returns (address hintAddress, uint diff, uint latestRandomSeed)
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::557 => function _getCurrentTellorResponse(address _collateral) internal view returns (TellorResponse memory tellorResponse) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::578 => function _getCurrentChainlinkResponse(address _collateral) internal view returns (ChainlinkResponse memory chainlinkResponse) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::580 => try priceAggregator[_collateral].decimals() returns (uint8 decimals) {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::610 => function _getPrevChainlinkResponse(address _collateral, uint80 _currentRoundId, uint8 _currentDecimals) internal view returns (ChainlinkResponse memory prevChainlinkResponse) {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::245 => internal returns (SingleRedemptionValues memory singleRedemption)
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::313 => function _descendList(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _startId) internal view returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::338 => function _ascendList(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _startId) internal view returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::363 => function findInsertPosition(address _collateral, uint256 _NICR, address _prevId, address _nextId) external view override returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::367 => function _findInsertPosition(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _prevId, address _nextId) internal view returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::329 => returns (LiquidationValues memory singleLiquidation)
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::369 => returns (LiquidationValues memory singleLiquidation)
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::899 => internal pure returns(LiquidationTotals memory newTotals) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1066 => function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1316 => function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1321 => function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::104 => function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::29 => function asset() external view override returns (address assetTokenAddress) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::37 => function totalAssets() external view override returns (uint256 totalManagedAssets) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::51 => function convertToShares(uint256 assets) public view override returns (uint256 shares) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::66 => function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::79 => function maxDeposit(address) external view override returns (uint256 maxAssets) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::96 => function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::122 => function maxMint(address) external view override returns (uint256 maxShares) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::139 => function previewMint(uint256 shares) public view override returns (uint256 assets) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::165 => function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::183 => function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::220 => function maxRedeem(address owner) external view override returns (uint256 maxShares) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::240 => function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
```

## [G-23] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::320 => function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::337 => function _requireCallerIsBOorTroveM() internal view {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::432 => function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::524 => function _requireValidCollateralAddress(address _collateral) internal view {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::533 => function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::537 => function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::546 => function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::551 => function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::567 => function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::624 => function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::636 => function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::640 => function _requireCallerIsStabilityPool() internal view {
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::125 => function _requireCallerIsBorrowerOperations() internal view {
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::131 => function _requireCallerIsTroveManager() internal view {
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::137 => function _requireCallerIsActivePool() internal view {
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::126 => function _requireCallerIsActivePool() internal view {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::251 => function _requireCallerIsTroveManagerOrActivePool() internal view {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::261 => function _requireCallerIsBorrowerOperations() internal view {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::265 => function _requireUserHasStake(uint currentStake) internal pure {
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::269 => function _requireNonZeroAmount(uint _amount) internal pure {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::320 => function _mint(address account, uint256 amount) internal {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::328 => function _burn(address account, uint256 amount) internal {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::360 => function _requireCallerIsBorrowerOperations() internal view {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::365 => function _requireCallerIsBOorTroveMorSP() internal view {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::375 => function _requireCallerIsStabilityPool() internal view {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::380 => function _requireCallerIsTroveMorSP() internal view {
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::393 => function _requireMintingNotPaused() internal view {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::557 => function _getCurrentTellorResponse(address _collateral) internal view returns (TellorResponse memory tellorResponse) {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::295 => function _requireCallerIsTroveManager() internal view {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::299 => function _requireValidCollateralAddress(address _collateral) internal view {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::303 => function _requireValidMaxFeePercentage(uint _maxFeePercentage) internal view {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::307 => function _requireAfterBootstrapPeriod() internal view {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::312 => function _requireTCRoverMCR(address _collateral, uint _price, uint256 _collDecimals, uint256 _MCR) internal view {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::316 => function _requireAmountGreaterThanZero(uint _amount) internal pure {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::320 => function _requireLUSDBalanceCoversRedemption(ILUSDToken _lusdToken, address _redeemer, uint _amount) internal view {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::44 => * - Public functions with parameters have been made internal to save gas, and given an external wrapper function for external access
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::338 => function _ascendList(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _startId) internal view returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::402 => function _requireCallerIsTroveManager() internal view {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::421 => function _updateG(uint _LQTYIssuance) internal {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::439 => function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::597 => function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::637 => function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::657 => function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::693 => function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::776 => function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::794 => function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::848 => function _requireCallerIsActivePool() internal view {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::852 => function _requireCallerIsTroveManager() internal view {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::856 => function _requireNoUnderCollateralizedTroves() internal {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::869 => function _requireUserHasDeposit(uint _initialDeposit) internal pure {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::873 => function _requireNonZeroAmount(uint _amount) internal pure {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1082 => function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1213 => function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1321 => function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1339 => function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1516 => function _minutesPassedSinceLastFeeOp() internal view returns (uint) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1538 => function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::176 => function _deposit(uint256 toReinvest) internal {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::187 => function _withdraw(uint256 _amount) internal {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::206 => function _claimRewards() internal {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::462 => function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {
```

## [G-24] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::432 => function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::44 => * - Public functions with parameters have been made internal to save gas, and given an external wrapper function for external access
```