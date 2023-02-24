### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 16 |
|:--:|:--:|


### Non-Critical Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Explicitly mark state variables visibility | 5 |
| [N-02] | Explicitly mark uint type size | 557 |
| [N-03] | Create your own import names instead of using the regular ones | 97 |
| [N-04] | Insufficient coverage | - |


| Total Non-Critical Issues | 4 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | Use require instead of assert | 18 |
| [R-02] | Some number values can be refactored with _ | 2 |
| [R-03] | Validate input before storage reads | 1 |
| [R-04] | Remove unused variables | 2 |
| [R-05] | Function state mutability can be restricted to pure | 1 |
| [R-06] | For functions, follow solidity standard naming conventions | 1 |
| [R-07] | Enum values should start with capital letter by convention | 1 |

| Total Refactor Issues | 7 |
|:--:|:--:|

### Ordinary Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | Remove TODOs in code | 2 |
| [O-02] | Commented out code | 3 |
| [O-03] | Use a more recent pragma version | 15 |
| [O-04] | Locking pragma | 4 |
| [O-05] | Increase NatSpec comments | 103 |


| Total Ordinary Issues | 5 |
|:--:|:--:|

### [N-01] Explicitly mark state variables visibility

Explicitly marking state variables visibility improves code readability and consistency.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

28: address stabilityPoolAddress;
30: address gasPoolAddress;
32: ICollSurplusPool collSurplusPool;
```

```solidity
Ethos-Core/contracts/TroveManager.sol

31: address gasPoolAddress;
33: ICollSurplusPool collSurplusPool;
```

### [N-02] Explicitly mark uint type size

Explicitly marking uint type size improves code readability and consistency.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol    128 Instances
Ethos-Core/contracts/TroveManager.sol    233 Instances
Ethos-Core/contracts/ActivePool.sol    8 Instances
Ethos-Core/contracts/StabilityPool.sol    135 Instances
Ethos-Core/contracts/LQTY/LQTYStaking.sol    49 Instances
Ethos-Core/contracts/LQTY/LQTYStaking.sol    3 Instances
Ethos-Vault/contracts/ReaperVaultV2.sol    1 Instance
```

### [N-03] Create your own import names instead of using the regular ones

For better readability, you should name the imports instead of using the regular ones.

Example:
```solidity
5: import {IBorrowerOperations} from "./Interfaces/IBorrowerOperations.sol";
```

### [N-04] Insufficient code coverage

Best practice is to come as close to 100% as possible for smart contracts.

According to documentation, coverage is 93%.

### [R-01] Use require instead of assert
The Solidity assert() function is meant to assert invariants. Properly functioning code should never reach a failing assert statement.

Instances:

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331: assert(_collWithdrawal <= vars.coll);

Ethos-Core/contracts/TroveManager.sol

417:  assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);

Ethos-Core/contracts/StabilityPool.sol

526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);

Ethos-Core/contracts/LUSDToken.sol

312: assert(sender != address(0));
313: assert(recipient != address(0));
321: assert(account != address(0));
329: assert(account != address(0));
337: assert(owner != address(0));
338: assert(spender != address(0));
```

### [R-02] Some number values can be refactored with _

#### Ethos-Core/contracts/TroveManager.sol

```solidity
uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

Refactor to:

```solidity 
uint constant public MINUTE_DECAY_FACTOR = 999_037_758_833_783_000;
```

#### Ethos-Vault/contracts/ReaperVaultV2.sol

```solidity
41:    uint256 public constant PERCENT_DIVISOR = 10000;
```

Refactor to:

```solidity
41:    uint256 public constant PERCENT_DIVISOR = 10_000;
```

### [R-03] Validate input before storage reads

If input parameters are validated by comparing with constants or immutables they should be done first to avoid storage reads and spend gas.

```solidity
Ethos-Core/contracts/CollateralConfig.sol

85: function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

Above function can be refactored to:

```solidity    
85: function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");
        config.MCR = _MCR;
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

### [R-04] Remove unused variables

```solidity 
Ethos-Core/contracts/StabilityPool.sol

339: uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit);
384: uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit);
```

### [R-05] Function state mutability can be restricted to pure

Functions can be declared pure in which case they promise not to read from or modify the state.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

659:    function _cascadingAccessRoles() internal view override returns (bytes32[] memory)
```

### [R-06] For functions, follow solidity standard naming conventions

```solidity 
Ethos-Core/contracts/BorrowerOperations.sol

546: function _requireTroveisNotActive(...)
```

The above codes don’t follow Solidity’s standard naming convention, consider the following changes:

```diff
- function _requireTroveisNotActive(...)
+ function _requireTroveIsNotActive(...)
```

### [R-07] Enum values should start with capital letter by convention

```solidity 
Ethos-Core/contracts/PriceFeed.sol

73: enum Status {...}
```

### [O-01] Remove TODOs in code

```solidity 
Ethos-Core/contracts/StabilityPool.sol

380:     /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
335:     /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
```

### [O-02] Commented out code

```solidity
Ethos-Core/contracts/TroveManager.sol

18:    contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
19:    // string constant public NAME = "TroveManager";
1413:    //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
```

### [O-03] Use a more recent pragma version

One of the most important best practices for writing secure Solidity code is to always use the latest version of the language. New versions of Solidity may include security updates and bug fixes that can help protect your code from potential vulnerabilities.

```solidity
Ethos-Core/contracts/ActivePool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/CollateralConfig.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/CollSurplusPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/DefaultPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/GasPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/HintHelpers.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/LUSDToken.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/Migrations.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/MultiTroveGetter.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/PriceFeed.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/RedemptionHelper.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/SortedTroves.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/StabilityPool.sol

3:    pragma solidity 0.6.11;
```

```solidity
Ethos-Core/contracts/TroveManager.sol

3:    pragma solidity 0.6.11;
```

### [O-04] Locking pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

#### Instances

- Ethos-Vault/contracts/ReaperVaultV2.sol
- Ethos-Vault/contracts/ReaperVaultERC4626.sol
- Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
- Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

### [O-05] Increase NatSpec comments

Some of the functions are missing NatSpec comments. Properly commenting the code is very important because it makes it easier for developers and other users to understand the code. Another more reason for commenting the smart contracts is to make it easier for auditing companies to audit the code. The minimum is all external and public functions to have NatSpec comments.

#### Instances

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

172:    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override
242:    function addColl(address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override
248:    function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override
254:    function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override
259:    function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override
264:    function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override
268:    function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override
375:    function closeTrove(address _collateral) external override
412:    function claimCollateral(address _collateral) external override
```

```solidity
Ethos-Core/contracts/TroveManager.sol

299:    function getTroveOwnersCount(address _collateral) external view override returns (uint)
303:    function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address)
310:    function liquidate(address _borrower, address _collateral) external override
513:    function liquidateTroves(address _collateral, uint _n) external override
715:    function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override
939:    function redeemCloseTrove(address _borrower, address _collateral, uint256 _LUSD, uint256 _collAmount) external override
957:    function reInsert(address _id, address _collateral, uint256 _newNICR, address _prevId, address _nextId) external override
962:    function updateDebtAndCollAndStakesPostRedemption(address _borrower, address _collateral, uint256 _newDebt, uint256 _newColl) external override
982:    function burnLUSDAndEmitRedemptionEvent(address _redeemer, address _collateral, uint _attemptedLUSDAmount, uint _actualLUSDAmount, uint _collSent, uint _collFee) external override
1016:    function redeemCollateral(address _collateral, uint _LUSDamount, address _firstRedemptionHint, address _upperPartialRedemptionHint, address _lowerPartialRedemptionHint, uint _partialRedemptionHintNICR, uint _maxIterations, uint _maxFeePercentage) external override
1045:    function getNominalICR(address _borrower, address _collateral) public view override returns (uint)
1054:    function getCurrentICR(address _borrower, address _collateral, uint _price) public view override returns (uint)
1076:    function applyPendingRewards(address _borrower, address _collateral) external override
1111:    function updateTroveRewardSnapshots(address _borrower, address _collateral) external override
1123:    function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint)
1138:    function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint)
1152:    function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool)
1164:    function getEntireDebtAndColl(address _borrower, address _collateral) public view override returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)
1183:    function removeStake(address _borrower, address _collateral) external override
1195:    function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint)
1273:    function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override
1316:    function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index)
1361:    function getTCR(address _collateral, uint _price) external view override returns (uint)
1366:    function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool)
1397:    function updateBaseRateFromRedemption(uint _collateralDrawn, uint _price, uint256 _collDecimals, uint _totalLUSDSupply) external override returns (uint)
1425:    function getRedemptionRate() public view override returns (uint)
1429:    function getRedemptionRateWithDecay() public view override returns (uint)
1440:    function getRedemptionFee(uint _collateralDrawn) public view override returns (uint)
1444:    function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint)
1456:    function getBorrowingRate() public view override returns (uint)
1460:    function getBorrowingRateWithDecay() public view override returns (uint)
1471:    function getBorrowingFee(uint _LUSDDebt) external view override returns (uint)
1475:    function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint)
1485:    function decayBaseRateFromBorrowing() external override
1544:    function getTroveStatus(address _borrower, address _collateral) external view override returns (uint)
1548:    function getTroveStake(address _borrower, address _collateral) external view override returns (uint)
1552:    function getTroveDebt(address _borrower, address _collateral) external view override returns (uint)
1556:    function getTroveColl(address _borrower, address _collateral) external view override returns (uint)
1562:    function setTroveStatus(address _borrower, address _collateral, uint _num) external override
1567:    function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint)
1574:    function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint)
1581:    function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint)
1588:    function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint)
```

```solidity
Ethos-Core/contracts/ActivePool.sol

125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner
132:    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner
138:    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner
144:    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner
159:    function getCollateral(address _collateral) external view override returns (uint)
164:    function getLUSDDebt(address _collateral) external view override returns (uint)
171:    function sendCollateral(address _collateral, address _account, uint _amount) external override
190:    function increaseLUSDDebt(address _collateral, uint _amount) external override
197:    function decreaseLUSDDebt(address _collateral, uint _amount) external override
204:    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override
214:    function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner
```

```solidity
Ethos-Core/contracts/StabilityPool.sol

307:    function getCollateral(address _collateral) external view override returns (uint)
311:    function getTotalLUSDDeposits() external view override returns (uint)
323:    function provideToSP(uint _amount) external override
367:    function withdrawFromSP(uint _amount) external override
410:    function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint)
466:    function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override
486:    function updateRewardSum(address _collateral, uint _collToAdd) external override
626:    function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts)
683:    function getDepositorLQTYGain(address _depositor) public view override returns (uint)
718:    function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint)
```

```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

84:    function issueOath() external override returns (uint issuance)
120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner
124:    function sendOath(address _account, uint _OathAmount) external override
```

```solidity
Ethos-Core/contracts/LQTY/LQTYStaking.sol

104:    function stake(uint _LQTYamount) external override
142:    function unstake(uint _LQTYamount) external override
177:    function increaseF_Collateral(address _collateral, uint _collFee) external override
187:    function increaseF_LUSD(uint _LUSDFee) external override
199:    function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory)
213:    function getPendingLUSDGain(address _user) external view override returns (uint)
```

```solidity
Ethos-Core/contracts/LUSDToken.sol

133:    function pauseMinting() external
141:    function unpauseMinting() external
146:    function updateGovernance(address _newGovernanceAddress) external
153:    function updateGuardian(address _newGuardianAddress) external
160:    function upgradeProtocol(address _newTroveManagerAddress, address _newStabilityPoolAddress, address _newBorrowerOperationsAddress) external
185:    function mint(address _account, uint256 _amount) external override
191:    function burn(address _account, uint256 _amount) external override
196:    function sendToPool(address _sender,  address _poolAddress, uint256 _amount) external override
201:    function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override
206:    function getDeploymentStartTime() external view override returns (uint256)
212:    function totalSupply() external view override returns (uint256)
216:    function balanceOf(address account) external view override returns (uint256)
220:    function transfer(address recipient, uint256 amount) external override returns (bool)
226:    function allowance(address owner, address spender) external view override returns (uint256)
230:    function approve(address spender, uint256 amount) external override returns (bool)
235:    function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool)
242:    function increaseAllowance(address spender, uint256 addedValue) external override returns (bool)
247:    function decreaseAllowance(address spender, uint256 subtractedValue) external override returns (bool)
254:    function domainSeparator() public view override returns (bytes32)
262:    function permit (address owner, address spender, uint amount, uint deadline, uint8 v, bytes32 r, bytes32 s) external override
292:    function nonces(address owner) external view override returns (uint256)
```
