# Gas Report 

[Exclusive findings](#exclusive-findings) is below the [automated findings](#automated-findings).

---

## Automated Findings
Automated findings output for the [Ethos-Core package](https://docs.reaper.farm/ethos-reserve-bounty-hunter-documentation/useful-information/slither-output-ethos-core) and [Ethos-Vault package](https://docs.reaper.farm/ethos-reserve-bounty-hunter-documentation/useful-information/slither-output-ethos-vault). 

## Gas Optimizations



| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use assembly to check for `address(0)` | 8 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 9 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 8 |
| [GAS-4](#GAS-4) | State variables should be cached in stack variables rather than re-reading them from storage | 10 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 1 |
| [GAS-6](#GAS-6) | Use Custom Errors | 73 |
| [GAS-7](#GAS-7) | Don't initialize variables with default value | 23 |
| [GAS-8](#GAS-8) | Long revert strings | 35 |
| [GAS-9](#GAS-9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 7 |
| [GAS-10](#GAS-10) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 17 |
| [GAS-11](#GAS-11) | Using `private` rather than `public` for constants, saves gas | 30 |
| [GAS-12](#GAS-12) | Use != 0 instead of > 0 for unsigned integer comparison | 27 |
| [GAS-13](#GAS-13) | `internal` functions not called by the contract should be removed | 1 |
### <a name="GAS-1"></a>[GAS-1] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (8)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

312:         assert(sender != address(0));

313:         assert(recipient != address(0));

321:         assert(account != address(0));

329:         assert(account != address(0));

337:         assert(owner != address(0));

338:         assert(spender != address(0));

348:             _recipient != address(0) && 

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (9)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

32:     bool public addressesSet = false;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

18:     bool public initialized = false;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

21:     bool public initialized = false;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

37:     bool public mintingPaused = false;

62:     mapping (address => bool) public troveManagers;

63:     mapping (address => bool) public stabilityPools;

64:     mapping (address => bool) public borrowerOperations;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

49:     bool public emergencyShutdown;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

30:     bool public emergencyExit;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (8)*:
```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56:         for(uint256 i = 0; i < _collaterals.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="GAS-4"></a>[GAS-4] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (10)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

181:             IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

184:             ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

299:                     ILQTYStaking(lqtyStakingAddress).increaseF_Collateral(_collateral, vars.stakingSplit);

305:                     IStabilityPool(stabilityPoolAddress).updateRewardSum(_collateral, vars.stabilityPoolSplit);   

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

113:         lastDistributionTime = block.timestamp.add(distributionPeriod);

116:         emit LogRewardPerSecond(rewardPerSecond);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

577:             emit EpochUpdated(currentEpoch);

579:             emit ScaleUpdated(currentScale);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (1)*:
```solidity
File: Ethos-Core/contracts/TroveManager.sol

715:     function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="GAS-6"></a>[GAS-6] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (73)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

85:         require(!addressesSet, "Can call setAddresses only once");

93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

111:             require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

127:         require(_bps <= 10_000, "Invalid BPS value");

133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

145:         require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

525:         require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

529:         require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530:         require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

534:         require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

538:         require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

543:         require(status == 1, "BorrowerOps: Trove does not exist or is closed");

548:         require(status != 1, "BorrowerOps: Trove is active");

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

51:         require(!initialized, "Can only initialize once");

52:         require(_collaterals.length != 0, "At least one collateral required");

53:         require(_MCRs.length == _collaterals.length, "Array lengths must match");

54:         require(_CCRs.length == _collaterals.length, "Array lenghts must match");

66:             require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

69:             require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

91:         require(_MCR <= config.MCR, "Can only walk down the MCR");

92:         require(_CCR <= config.CCR, "Can only walk down the CCR");

94:         require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

97:         require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

129:         require(collateralConfig[_collateral].allowed, "Invalid collateral address");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

70:         require(!initialized, "issuance has been initialized");

102:         require(amount != 0, "cannot fund 0");

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:         require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:         require(!mintingPaused, "LUSDToken: Minting is currently paused");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

270:         require(y != 0, "Division by 0");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

150:         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

151:         require(_strategy != address(0), "Invalid strategy address");

152:         require(strategies[_strategy].activation == 0, "Strategy already added");

153:         require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

154:         require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");

155:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

156:         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

180:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

181:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

193:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

197:         require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");

261:         require(queueLength != 0, "Queue must not be empty");

267:             require(params.activation != 0, "Invalid strategy address");

321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

322:         require(_amount != 0, "Invalid amount");

324:         require(pool + _amount <= tvlCap, "Vault is full");

364:         require(_shares != 0, "Invalid amount");

436:         require(loss <= allocation, "Strategy loss cannot be greater than allocation");

497:         require(strategy.activation != 0, "Unauthorized strategy");

568:         require(_withdrawMaxLoss <= PERCENT_DIVISOR, "Invalid BPS value");

619:         require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

629:         require(newTreasury != address(0), "Invalid address");

639:         require(_token != address(token), "!token");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

81:         require(_multisigRoles.length == 3, "Invalid number of multisig roles");

96:         require(msg.sender == vault, "Only vault can withdraw");

97:         require(_amount != 0, "Amount cannot be zero");

98:         require(_amount <= balanceOf(), "Ammount must be less than balance");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="GAS-7"></a>[GAS-7] Don't initialize variables with default value

*Instances (23)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56:         for(uint256 i = 0; i < _collaterals.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

117:         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

165:         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

128:         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

264:         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

369:             uint256 totalLoss = 0;

371:             uint256 vaultBalance = 0;

372:             for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

77:         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

100:         uint256 amountFreed = 0;

112:         uint256 debt = 0;

117:         uint256 repayment = 0;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="GAS-8"></a>[GAS-8] Long revert strings

*Instances (35)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:         require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:         require(!mintingPaused, "LUSDToken: Minting is currently paused");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:         require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

150:         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

436:         require(loss <= allocation, "Strategy loss cannot be greater than allocation");

619:         require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

98:         require(_amount <= balanceOf(), "Ammount must be less than balance");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="GAS-9"></a>[GAS-9] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (7)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

101:     function fund(uint amount) external onlyOwner {

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

### <a name="GAS-10"></a>[GAS-10] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (17)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56:         for(uint256 i = 0; i < _collaterals.length; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

286:                          _nonces[owner]++, deadline))));

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

608:         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:         for (vars.i = 0; vars.i < _n; vars.i++) {

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

273:         if (x % y != 0) q++;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

### <a name="GAS-11"></a>[GAS-11] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (30)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

30:     string constant public NAME = "ActivePool";

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

21:     string constant public NAME = "BorrowerOperations";

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

21:     uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%

25:     uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

19:     string constant public NAME = "CommunityIssuance";

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

23:     string constant public NAME = "LQTYStaking";

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

150:     string constant public NAME = "StabilityPool";

197:     uint public constant SCALE_FACTOR = 1e9;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

48:     uint constant public SECONDS_IN_ONE_MINUTE = 60;

53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

61:     uint constant public BETA = 2;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="GAS-12"></a>[GAS-12] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (27)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

278:         if (vars.netAssetMovement > 0) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

128:         assert(MIN_NET_DEBT > 0);

197:         assert(vars.compositeDebt > 0);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

337:         if (!_isDebtIncrease && _LUSDChange > 0) {

552:         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

152:         if (_LQTYamount > 0) {

181:         if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

191:         if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

279:         if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

591:         assert(newP > 0);

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:         require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

424:             if (singleLiquidation.collSurplus > 0) {

452:         if (_LUSDInStabPool > 0) {

551:         require(totals.totalDebtInSequence > 0);

563:         if (totals.totalCollSurplus > 0) {

754:         require(totals.totalDebtInSequence > 0);

766:         if (totals.totalCollSurplus > 0) {

916:         if (_LUSD > 0) {

920:         if (_collAmount > 0) {

1224:             assert(totalStakesSnapshot[_collateral] > 0);

1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

502:         } else if (_roi > 0) {

518:         } else if (vars.available > 0) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

### <a name="GAS-13"></a>[GAS-13] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (1)*:
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

432:     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

---
## Exclusive Findings
---
### Gas Optimizations Report
| No. | Issue |
| --- | --- |
| 1 | [Splitting `require` which contains `&&` saves gas](#G-01-splitting-require-which-contains--saves-gas)|
| 2 | [Reduce the size of error messages (Long revert Strings)](#G-02-reduce-the-size-of-error-messages-long-revert-strings)|
| 3 | [Use Custom Errors rather than revert() / require() strings to save deployment gas [68 gas per instance]](#G-03-use-custom-errors-rather-than-revert--require-strings-to-save-deployment-gas-68-gas-per-instance)|
| 4 | [Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#G-04-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead)|
| 5 | [++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#G-05-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops)|
| 6 | [Optimize names to save gas [22 gas per instance]](#G-06-optimize-names-to-save-gas-22-gas-per-instance)|
| 7 | [Setting the `constructor` to `payable` [~13 gas per instance]](#G-07-setting-the-constructor-to-payable-13-gas-per-instance)|
| 8 | [Comparison operators](#G-08-comparison-operators)|
| 9 | [Use a more recent version of solidity](#G-09-use-a-more-recent-version-of-solidity)|
| 10 | [`x += y` costs more gas than `x = x + y` for state variables](#G-10-x--y-costs-more-gas-than-x--x--y-for-state-variables)|
| 11 | [`<array>.length` should not be looked up in every loop of a for-loop](#G-11-arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)|
---
### [G-01] Splitting `require` which contains `&&` saves gas

---
**Description:**

Using multiple require instead of operator && can save more gas. When a require statement has 2 or more expressions with &&, separate them into individual expression results in lesser gas.

**Lines of Code:** 

- [BorrowerOperations.sol#L653-L654](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)
- [LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L347-L351)
- [LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L352-L357)

---
### [G-02] Reduce the size of error messages (Long revert Strings)

---
**Description:**

Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition has been met. 

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

**Recommendation:**

Shorten the revert strings to fit in 32 bytes. 

Or in contracts using solc version 0.8.4 or greater use the Custom Errors feature.

**Lines of Code:** 

- [ActivePool.sol#L107](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L107)
- [ActivePool.sol#L133](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L133)
- [ActivePool.sol#L324](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L324)
- [ActivePool.sol#L334](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L334)
- [ActivePool.sol#L341](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L341)
- [BorrowerOperations.sol#L525](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L525)
- [BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [BorrowerOperations.sol#L534](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L534)
- [BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L538)
- [BorrowerOperations.sol#L543](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L543)
- [BorrowerOperations.sol#L552](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L552)
- [BorrowerOperations.sol#L563](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L563)
- [BorrowerOperations.sol#L568](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L568)
- [BorrowerOperations.sol#L617](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L617)
- [BorrowerOperations.sol#L621](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L621)
- [BorrowerOperations.sol#L625](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L625)
- [BorrowerOperations.sol#L629](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L629)
- [BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [BorrowerOperations.sol#L641](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L641)
- [BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [BorrowerOperations.sol#L651](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L651)
- [BorrowerOperations.sol#L654](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L654)
- [CommunityIssuance.sol#L133](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133)
- [LQTYStaking.sol#L257](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257)
- [LQTYStaking.sol#L262](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262)
- [LUSDToken.sol#L136](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L136)
- [LUSDToken.sol#L350](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L350)
- [LUSDToken.sol#L356](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L356)
- [LUSDToken.sol#L362](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L362)
- [LUSDToken.sol#L371](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L371)
- [LUSDToken.sol#L377](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L377)
- [LUSDToken.sol#L386](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L386)
- [LUSDToken.sol#L390](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L390)
- [LUSDToken.sol#L394](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L394)
- [StabilityPool.sol#L849](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L849)
- [StabilityPool.sol#L853](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L853)
- [StabilityPool.sol#L865](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L865)
- [ReaperVaultV2.sol#L150](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150)
- [ReaperVaultV2.sol#L321](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321)
- [ReaperVaultV2.sol#L436](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436)
- [ReaperVaultV2.sol#L619](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L619)
- [ReaperBaseStrategyv4.sol#L98](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98)
- [ReaperBaseStrategyv4.sol#L193](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L193)

---
### [G-03] Use Custom Errors rather than revert() / require() strings to save deployment gas [68 gas per instance]

---
**Description:**

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hitby avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.

<https://blog.soliditylang.org/2021/04/21/custom-errors/>

**Recommendation:**

Use the Custom Errors feature.

**Lines of Code:** 

- [ActivePool.sol#L85](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L85)
- [ActivePool.sol#L93](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L93)
- [ActivePool.sol#L107](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L107)
- [ActivePool.sol#L111](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L111)
- [ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L127)
- [ActivePool.sol#L133](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L133)
- [ActivePool.sol#L145](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L145)
- [ActivePool.sol#L314-L317](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L314-L317)
- [ActivePool.sol#L321-L324](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L321-L324)
- [ActivePool.sol#L329-L334](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L329-L334)
- [ActivePool.sol#L338-L341](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L338-L341)
- [BorrowerOperations.sol#L525](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L525)
- [BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [BorrowerOperations.sol#L534](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L534)
- [BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L538)
- [BorrowerOperations.sol#L543](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L543)
- [BorrowerOperations.sol#L548](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L548)
- [BorrowerOperations.sol#L552](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L552)
- [BorrowerOperations.sol#L561-L564](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L561-L564)
- [BorrowerOperations.sol#L568](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L568)
- [BorrowerOperations.sol#L617](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L617)
- [BorrowerOperations.sol#L621](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L621)
- [BorrowerOperations.sol#L625](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L625)
- [BorrowerOperations.sol#L629](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L629)
- [BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [BorrowerOperations.sol#L641](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L641)
- [BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [BorrowerOperations.sol#L650-L651](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L650-L651)
- [BorrowerOperations.sol#L653-L654](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)
- [CollateralConfig.sol#L51](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L51)
- [CollateralConfig.sol#L52](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L52)
- [CollateralConfig.sol#L53](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L53)
- [CollateralConfig.sol#L54](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L54)
- [CollateralConfig.sol#L66](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L66)
- [CollateralConfig.sol#L69](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L69)
- [CollateralConfig.sol#L91](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L91)
- [CollateralConfig.sol#L92](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L92)
- [CollateralConfig.sol#L94](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L94)
- [CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L97)
- [CollateralConfig.sol#L129](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L129)
- [CommunityIssuance.sol#L70](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L70)
- [CommunityIssuance.sol#L102](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L102)
- [CommunityIssuance.sol#L133](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133)
- [LQTYStaking.sol#L253-L258](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L253-L258)
- [LQTYStaking.sol#L262](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262)
- [LUSDToken.sol#L134-L137](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L134-L137)
- [LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L347-L351)
- [LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L352-L357)
- [LUSDToken.sol#L362](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L362)
- [LUSDToken.sol#L367-L372](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L367-L372)
- [LUSDToken.sol#L377](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L377)
- [LUSDToken.sol#L384-L386](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L384-L386)
- [LUSDToken.sol#L390](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L390)
- [LUSDToken.sol#L394](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L394)
- [StabilityPool.sol#L849](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L849)
- [StabilityPool.sol#L853](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L853)
- [StabilityPool.sol#L865](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L865)
- [ReaperVaultERC4626.sol#L270](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L270)
- [ReaperVaultV2.sol#L150](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150)
- [ReaperVaultV2.sol#L151](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151)
- [ReaperVaultV2.sol#L152](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L152)
- [ReaperVaultV2.sol#L153](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L153)
- [ReaperVaultV2.sol#L154](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L154)
- [ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)
- [ReaperVaultV2.sol#L156](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156)
- [ReaperVaultV2.sol#L180](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180)
- [ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)
- [ReaperVaultV2.sol#L193](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193)
- [ReaperVaultV2.sol#L197](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197)
- [ReaperVaultV2.sol#L261](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L261)
- [ReaperVaultV2.sol#L267](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L267)
- [ReaperVaultV2.sol#L321](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321)
- [ReaperVaultV2.sol#L322](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L322)
- [ReaperVaultV2.sol#L324](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L324)
- [ReaperVaultV2.sol#L364](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L364)
- [ReaperVaultV2.sol#L404-L407](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L404-L407)
- [ReaperVaultV2.sol#L436](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436)
- [ReaperVaultV2.sol#L497](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L497)
- [ReaperVaultV2.sol#L568](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L568)
- [ReaperVaultV2.sol#L619](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L619)
- [ReaperVaultV2.sol#L629](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L629)
- [ReaperVaultV2.sol#L639](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L639)
- [ReaperBaseStrategyv4.sol#L81](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L81)
- [ReaperBaseStrategyv4.sol#L96](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L96)
- [ReaperBaseStrategyv4.sol#L97](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L97)
- [ReaperBaseStrategyv4.sol#L98](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98)
- [ReaperBaseStrategyv4.sol#L191-L194](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L191-L194)

---
### [G-04] Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

---
**Description:**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so.

**Recommendation:**

Use a larger size then downcast where needed

**Lines of Code:** 

- [LUSDToken.sol#L35](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L35)
- [LUSDToken.sol#L268](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L268)
- [StabilityPool.sol#L182](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L182)
- [StabilityPool.sol#L183](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L183)
- [StabilityPool.sol#L200](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L200)
- [StabilityPool.sol#L203](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L203)
- [StabilityPool.sol#L214](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L214)
- [StabilityPool.sol#L214](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L214)
- [StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L223)
- [StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L223)
- [StabilityPool.sol#L247](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L247)
- [StabilityPool.sol#L247](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L247)
- [StabilityPool.sol#L248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L248)
- [StabilityPool.sol#L248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L248)
- [StabilityPool.sol#L249](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L249)
- [StabilityPool.sol#L250](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L250)
- [StabilityPool.sol#L558](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L558)
- [StabilityPool.sol#L559](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L559)
- [StabilityPool.sol#L648](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L648)
- [StabilityPool.sol#L649](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L649)
- [StabilityPool.sol#L699](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L699)
- [StabilityPool.sol#L700](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L700)
- [StabilityPool.sol#L738](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L738)
- [StabilityPool.sol#L739](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L739)
- [StabilityPool.sol#L745](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L745)
- [StabilityPool.sol#L817](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L817)
- [StabilityPool.sol#L818](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L818)
- [TroveManager.sol#L82](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L82)
- [TroveManager.sol#L1321](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1321)
- [TroveManager.sol#L1344](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1344)
- [ReaperStrategyGranarySupplyOnly.sol#L35](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35)

---
### [G-05] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

---
**Description:**

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

**Recommendation:**

Using Solidity's unchecked block saves the overflow checks.

**Proof Of Concept:**

<https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---unnecessary-checked-arithmetic-in-for-loop>

**Lines of Code:** 

- [ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L108)
- [CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L56)
- [LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
- [LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)
- [LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
- [StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L351)
- [StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L397)
- [StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L640)
- [StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L810)
- [StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L831)
- [StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L859)
- [TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L608)
- [TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L690)
- [TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L808)
- [TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L882)

---
### [G-06] Optimize names to save gas [22 gas per instance]

---
**Description:**

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Recommendation:**

Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas. For example, the function IDs in the L1GraphTokenGateway.sol contract will be the most used; A lower method ID may be given.

**Proof Of Concept:**

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

**Lines of Code:** 

- [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)
- [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)
- [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)
- [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
- [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
- [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)
- [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)
- [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)
- [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
- [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
- [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

---
### [G-07] Setting the `constructor` to `payable` [~13 gas per instance]

---
**Lines of Code:** 

- [CommunityIssuance.sol#L57](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57)
- [TroveManager.sol#L225](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L225)
- [ReaperVaultERC4626.sol#L16](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16)
- [ReaperVaultV2.sol#L111](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111)
- [ReaperBaseStrategyv4.sol#L61](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61)

---
### [G-08] Comparison operators

---
**Description:**

In the EVM, there is no opcode for `>=` or `<=`. When using greater than or equal, two operations are performed: `>` and `=`. Using strict comparison operators hence saves gas.

**Recommendation:**

 Replace `<=` with `<`, and `>=` with `>`. Do not forget to increment/decrement the compared variable.

**Lines of Code:** 

- [ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L127)
- [ActivePool.sol#L133](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L133)
- [BorrowerOperations.sol#L331](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L331)
- [BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [BorrowerOperations.sol#L588](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L588)
- [BorrowerOperations.sol#L617](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L617)
- [BorrowerOperations.sol#L621](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L621)
- [BorrowerOperations.sol#L621](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L621)
- [BorrowerOperations.sol#L625](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L625)
- [BorrowerOperations.sol#L629](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L629)
- [BorrowerOperations.sol#L633](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L633)
- [BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [BorrowerOperations.sol#L650](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L650)
- [BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)
- [BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)
- [CollateralConfig.sol#L66](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L66)
- [CollateralConfig.sol#L69](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L69)
- [CollateralConfig.sol#L91](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L91)
- [CollateralConfig.sol#L92](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L92)
- [CollateralConfig.sol#L94](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L94)
- [CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L97)
- [LUSDToken.sol#L282](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L282)
- [StabilityPool.sol#L526](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L526)
- [StabilityPool.sol#L551](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L551)
- [StabilityPool.sol#L865](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L865)
- [TroveManager.sol#L383](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L383)
- [TroveManager.sol#L410](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L410)
- [TroveManager.sol#L415](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L415)
- [TroveManager.sol#L415](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L415)
- [TroveManager.sol#L616](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L616)
- [TroveManager.sol#L817](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L817)
- [TroveManager.sol#L1220](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1220)
- [TroveManager.sol#L1348](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1348)
- [TroveManager.sol#L1503](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1503)
- [ReaperVaultERC4626.sol#L80](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L80)
- [ReaperVaultERC4626.sol#L123](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L123)
- [ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)
- [ReaperVaultV2.sol#L156](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156)
- [ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)
- [ReaperVaultV2.sol#L197](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L197)
- [ReaperVaultV2.sol#L240](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L240)
- [ReaperVaultV2.sol#L324](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L324)
- [ReaperVaultV2.sol#L374](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L374)
- [ReaperVaultV2.sol#L405](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L405)
- [ReaperVaultV2.sol#L436](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L436)
- [ReaperVaultV2.sol#L568](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L568)
- [ReaperVaultV2.sol#L619](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L619)
- [ReaperBaseStrategyv4.sol#L98](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98)
- [ReaperBaseStrategyv4.sol#L238](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L238)

---
### [G-09] Use a more recent version of solidity

---
**Description:**

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

**Lines of Code:** 

- [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)
- [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)
- [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)
- [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
- [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
- [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)
- [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)
- [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)
- [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
- [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
- [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

---
### [G-10] `x += y` costs more gas than `x = x + y` for state variables

---
**Lines of Code:** 

- [ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
- [ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
- [ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
- [ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
- [ReaperVaultV2.sol#L395](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395)
- [ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)
- [ReaperVaultV2.sol#L445](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445)
- [ReaperVaultV2.sol#L452](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452)
- [ReaperVaultV2.sol#L515](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515)
- [ReaperVaultV2.sol#L521](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521)

---
### [G-11] `<array>.length` should not be looked up in every loop of a for-loop

---
**Description:**

The overheads outlined below are *PER LOOP*, excluding the first loop 

- storage arrays incur a Gwarmaccess (100 gas)

- memory arrays use `MLOAD` (3 gas)

- calldata arrays use `CALLDATALOAD` (3 gas)

Caching the length changes each of these to a `DUP<N>` (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset



**Lines of Code:** 

- [CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L56)
- [LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
- [LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)
- [StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L640)
- [StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L810)
- [StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L831)
- [TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L808)
- [TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L882)