# Report


## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
|[G-1]| Setting the `constructor` to `payable` saves gas (sayan)|2|
|[G-2]| Mark functions as payable (with discretion)              |49|
|[G-3]| Use assembly for math (add, sub, mul, div)                 |2|
|[G-4]| Short Revert Strings                               |8|
|[G-5]| Use assembly to write storage values                 |13|
|[G-6]| Use multiple require() statments insted of require(expression && expression && …) |4|
|[G-7]| Optimal Comparison |8|
|[G-8]| Tightly pack storage variables |2|
|[G-9]| Instead of if(x), use assembly with iszeroiszero (x)(secoalba) |12|




**[G-1] Setting the `constructor` to `payable` saves gas (sayan)**

Saves ~13 gas per instance
```solidity
2 instances

File: Ethos-Vault/contracts/ReaperVaultV2.sol
111:     constructor(
112:         address _token,
113:         string memory _name,
114:         string memory _symbol,
115:         uint256 _tvlCap,
116:         address _treasury,
117:         address[] memory _strategists,
118:         address[] memory _multisigRoles
119:     ) ERC20(string(_name), string(_symbol)) {


File: Ethos-Vault/contracts/ReaperVaultERC4626.sol
16:     constructor(
17:         address _token,
18:         string memory _name,
19:         string memory _symbol,
20:         uint256 _tvlCap,
21:         address _treasury,
22:         address[] memory _strategists,
23:         address[] memory _multisigRoles
24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

```

**[G-2] Mark functions as payable (with discretion)**

You can mark public or external functions as payable to save gas. Functions that are not payable have additional logic to check if there was a value sent with a call, however, making a function payable eliminates this check. This optimization should be carefully considered due to potentially unwanted behavior when a function does not need to accept ether.

Saves ~24 gas per instance

```solidity
49 instances

File; Ethos-Core/contracts/TroveManager.sol

225:     constructor() public {

232:    function setAddresses(

299:    function getTroveOwnersCount(address _collateral) external view override returns (uint) {

303:    function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {

310:    function liquidate(address _borrower, address _collateral) external override {

513:    function liquidateTroves(address _collateral, uint _n) external override {

715:    function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {

939:    function redeemCloseTrove(

962:    function updateDebtAndCollAndStakesPostRedemption(

982:    function burnLUSDAndEmitRedemptionEvent(

1016:    function redeemCollateral(

1045:    function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1054:    function getCurrentICR(

1076:    function applyPendingRewards(address _borrower, address _collateral) external override {

1111:    function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {

1123:    function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1138:    function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1152:    function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {

1164:    function getEntireDebtAndColl(

1183:    function removeStake(address _borrower, address _collateral) external override {

1195:    function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {

1273:    function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {

1316:    function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {

1361:    function getTCR(address _collateral, uint _price) external view override returns (uint) {

1366:    function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {

1397:    function updateBaseRateFromRedemption(

1425:    function getRedemptionRate() public view override returns (uint) {

1440:    function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1444:    function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1456:     function getBorrowingRate() public view override returns (uint) {

1460:    function getBorrowingRateWithDecay() public view override returns (uint) {

1471:    function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {

1475:    function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {

1485:    function decayBaseRateFromBorrowing() external override {

1544:    function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {

1556:    function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {

1562:    function setTroveStatus(address _borrower, address _collateral, uint _num) external override {

1567:    function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {

1574:    function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

1588:    function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {


File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

95:     function withdraw(uint256 _amount) external override returns (uint256 loss) {

109:    function harvest() external override returns (int256 roi) {

156:    function setEmergencyExit() external {

167:    function initiateUpgradeCooldown() external {

```


[G-3] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

Saves ~40 gas per instance

```solidity
2 instances

File: Ethos-Core/contracts/TroveManager.sol

54:    uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
55:    uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%


```

**[G-4] Short Revert Strings**

Keeping revert strings under 32-bytes prevents the string from being stored in more than one memory slot.

Saves ~3 gas per instance

```solidity
8 instances

File: Ethos-Core/contracts/StabilityPool.sol

849:        require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:        require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:            require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:        require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:        require(_amount > 0, 'StabilityPool: Amount must be non-zero');


File:  Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

81:        require(_multisigRoles.length == 3, "Invalid number of multisig roles");

98:        require(_amount <= balanceOf(), "Ammount must be less than balance");

193:         require(
            upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
            "Upgrade cooldown not initiated or still ongoing"


```

**[G-5] Use assembly to write storage values**

Saves ~66 gas per instance

```solidity
13 instances

File; Ethos-Core/contracts/TroveManager.sol

227:        owner = msg.sender;

266:        borrowerOperationsAddress = _borrowerOperationsAddress;

271:        gasPoolAddress = _gasPoolAddress;

294:        owner = address(0);

1417:        baseRate = newBaseRate;

1504:            lastFeeOperationTime = block.timestamp;



File:  Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

72:        vault = _vault;

73:        want = _want;

137:        lastHarvestTimestamp = block.timestamp;

158:        emergencyExit = true;

169:        IVault(vault).revokeStrategy(address(this));

181:        upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;


```

**[G-6] Use multiple require() statments insted of require(expression && expression && …)**

You can safe gas by breaking up a require statement with multiple conditions, into multiple require statements with a single condition.

Saves ~16 gas per instance

```solidity
4 instances

File: Ethos-Core/contracts/BorrowerOperations.sol

653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

File: Ethos-Core/contracts/LUSDToken.sol

347:        require(

352:        require(

File: Ethos-Core/contracts/TroveManager.sol

1539:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```


**[G-7] Optimal Comparison**

When comparing integers, it is cheaper to use strict > & < operators over >= & <= operators, even if you must increment or decrement one of the operands. Note: before using this technique, it’s important to consider whether incrementing/decrementing one of the operators could result in an over/underflow.
This optimization is applicable when the optimizer is turned off.

Saves ~3 gas per instance

```solidity
8 instances

File: Ethos-Core/contracts/TroveManager.sol

372:        if (TroveOwners[_collateral].length <= 1) {return singleLiquidation;} // don't liquidate if last trove

383:        if (_ICR <= _100pct) {

415:        } else if ((_ICR >= _MCR) && (_ICR < _TCR) && (singleLiquidation.entireTroveDebt <= _LUSDInStabPool)) {

616:                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

817:                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }

1348:        assert(index <= idxLast);

1489:        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

1503:        if (timePassed >= SECONDS_IN_ONE_MINUTE) {



```



**[G-8] Tightly pack storage variables**

When defining storage variables, make sure to declare them in ascending order, according to size. When multiple variables are able to fit into one 256 bit slot, this will save storage size and gas during runtime. For example, if you have a bool, uint256 and a bool, instead of defining the variables in the previously mentioned order, defining the two boolean variables first will pack them both into one storage slot since they only take up one byte of storage.

Saves ~17,298 gas per instance

```solidity
2 instances

File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

19: contract ReaperStrategyGranarySupplyOnly is ReaperBaseStrategyv4, VeloSolidMixin {

File: Ethos-Vault/contracts/ReaperVaultV2.sol

21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

```

**[G-9] Instead of if(x), use assembly with iszeroiszero (x)(secoalba)**

Saves ~5 gas per instance

```solidity
12 instances

File: Ethos-Core/contracts/BorrowerOperations.sol

189:        if (!isRecoveryMode) {

202:        if (isRecoveryMode) {

292:        if (_isDebtIncrease) {

311:        if (_isDebtIncrease && !isRecoveryMode) { 

337:        if (!_isDebtIncrease && _LUSDChange > 0) {

490:        if (_isDebtIncrease) {

496:        if (_isCollIncrease) {

597:            if (_isDebtIncrease) {

649:        if (_isRecoveryMode) {

File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

75:         if (emergencyExit) {

File: Ethos-Vault/contracts/ReaperVaultV2.sol

604:        if (_active) {



```



