## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Bytes constants are more efficient than string constants | 8 |
| [GAS-2](#GAS-2) | Incrementing with a smaller type than `uint256` incurs overhead | 3 |
| [GAS-3](#GAS-3) | Use `storage` instead of `memory` for structs/arrays | 16 |
| [GAS-4](#GAS-4) | Increments can be `unchecked` in for-loops | 14 |
| [GAS-5](#GAS-5) | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 15 |
| [GAS-6](#GAS-6) | Use shift Right/Left instead of division/multiplication if possible | 1 |
| [GAS-7](#GAS-7) | Use assembly to check for `address(0)` | 8 |
| [GAS-8](#GAS-8) | Using bools for storage incurs overhead | 7 |
| [GAS-9](#GAS-9) | Cache array length outside of loop | 7 |
| [GAS-10](#GAS-10) | State variables should be cached in stack variables rather than re-reading them from storage | 10 |
| [GAS-11](#GAS-11) | Use calldata instead of memory for function arguments that do not get mutated | 1 |
| [GAS-12](#GAS-12) | Use Custom Errors | 58 |
| [GAS-13](#GAS-13) | Don't initialize variables with default value | 18 |
| [GAS-14](#GAS-14) | Long revert strings | 34 |
| [GAS-15](#GAS-15) | Using `private` rather than `public` for constants, saves gas | 21 |
| [GAS-16](#GAS-16) | Use != 0 instead of > 0 for unsigned integer comparison | 27 |
| [GAS-17](#GAS-17) | `internal` functions not called by the contract should be removed | 1 |
### <a name="GAS-1"></a>[GAS-1] Bytes constants are more efficient than string constants

*Instances (8)*:
```solidity
File: ActivePool.sol

30:     string constant public NAME = "ActivePool";

```

```solidity
File: BorrowerOperations.sol

21:     string constant public NAME = "BorrowerOperations";

```

```solidity
File: LQTY/CommunityIssuance.sol

19:     string constant public NAME = "CommunityIssuance";

```

```solidity
File: LQTY/LQTYStaking.sol

23:     string constant public NAME = "LQTYStaking";

```

```solidity
File: LUSDToken.sol

32:     string constant internal _NAME = "LUSD Stablecoin";

33:     string constant internal _SYMBOL = "LUSD";

34:     string constant internal _VERSION = "1";

```

```solidity
File: StabilityPool.sol

150:     string constant public NAME = "StabilityPool";

```

### <a name="GAS-2"></a>[GAS-2] Incrementing with a smaller type than `uint256` incurs overhead

*Instances (3)*:
```solidity
File: LQTY/CommunityIssuance.sol

15:     using SafeMath for uint;

```

```solidity
File: LQTY/LQTYStaking.sol

20:     using SafeMath for uint;

```

```solidity
File: StabilityPool.sol

147:     using LiquitySafeMath128 for uint128;

```

### <a name="GAS-3"></a>[GAS-3] Use `storage` instead of `memory` for structs/arrays
Using `memory` copies the struct or array in memory. Use `storage` to save the location in storage and have cheaper reads:

*Instances (16)*:
```solidity
File: ActivePool.sol

106:         uint256 numCollaterals = collaterals.length;

```

```solidity
File: BorrowerOperations.sol

176:         LocalVariables_openTrove memory vars;

284:         LocalVariables_adjustTrove memory vars;

```

```solidity
File: LQTY/LQTYStaking.sol

148:         uint LUSDGain = _getPendingLUSDGain(msg.sender);

227:         uint[] memory amounts = new uint[](collaterals.length);

228:         for (uint i = 0; i < collaterals.length; i++) {

```

```solidity
File: ReaperStrategyGranarySupplyOnly.sol

167:             require(step[0] != address(0));

```

```solidity
File: ReaperVaultV2.sol

661:         cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;

```

```solidity
File: StabilityPool.sol

333:         uint compoundedLUSDDeposit = getCompoundedLUSDDeposit(msg.sender);

377:         uint compoundedLUSDDeposit = getCompoundedLUSDDeposit(msg.sender);

688:         uint LQTYGain = _getLQTYGainFromSnapshots(initialDeposit, snapshots);

723: 

807:         uint[] memory amounts = new uint[](collaterals.length);

808: 

858:         uint numCollaterals = collaterals.length;

```

```solidity
File: TroveManager.sol

314:         borrowers[0] = _borrower;

```

### <a name="GAS-4"></a>[GAS-4] Increments can be `unchecked` in for-loops

*Instances (14)*:
```solidity
File: ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: TroveManager.sol

608:         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:         for (vars.i = 0; vars.i < _n; vars.i++) {

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```

### <a name="GAS-5"></a>[GAS-5] Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
*Saves 5 gas per iteration*

*Instances (15)*:
```solidity
File: ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: ReaperVaultERC4626.sol

273:         if (x % y != 0) q++;

```

```solidity
File: StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: TroveManager.sol

608:         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:         for (vars.i = 0; vars.i < _n; vars.i++) {

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```

### <a name="GAS-6"></a>[GAS-6] Use shift Right/Left instead of division/multiplication if possible
Shifting left by N is like multiplying by 2^N and shifting right by N is like dividing by 2^N

*Instances (1)*:
```solidity
File: BorrowerOperations.sol

398: 

```

### <a name="GAS-7"></a>[GAS-7] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (8)*:
```solidity
File: ActivePool.sol

93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

```

```solidity
File: LUSDToken.sol

312:         assert(sender != address(0));

313:         assert(recipient != address(0));

321:         assert(account != address(0));

329:         assert(account != address(0));

337:         assert(owner != address(0));

338:         assert(spender != address(0));

348:             _recipient != address(0) && 

```

### <a name="GAS-8"></a>[GAS-8] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (7)*:
```solidity
File: ActivePool.sol

32:     bool public addressesSet = false;

```

```solidity
File: LQTY/CommunityIssuance.sol

21:     bool public initialized = false;

```

```solidity
File: LUSDToken.sol

37:     bool public mintingPaused = false;

62:     mapping (address => bool) public troveManagers;

63:     mapping (address => bool) public stabilityPools;

64:     mapping (address => bool) public borrowerOperations;

```

```solidity
File: ReaperVaultV2.sol

49:     bool public emergencyShutdown;

```

### <a name="GAS-9"></a>[GAS-9] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (7)*:
```solidity
File: LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

```

```solidity
File: StabilityPool.sol

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

```

```solidity
File: TroveManager.sol

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```

### <a name="GAS-10"></a>[GAS-10] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Saves 100 gas per instance*

*Instances (10)*:
```solidity
File: ActivePool.sol

181:             IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

184:             ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);

299:                     ILQTYStaking(lqtyStakingAddress).increaseF_Collateral(_collateral, vars.stakingSplit);

305:                     IStabilityPool(stabilityPoolAddress).updateRewardSum(_collateral, vars.stabilityPoolSplit);   

```

```solidity
File: LQTY/CommunityIssuance.sol

113:         lastDistributionTime = block.timestamp.add(distributionPeriod);

116:         emit LogRewardPerSecond(rewardPerSecond);

```

```solidity
File: StabilityPool.sol

577:             emit EpochUpdated(currentEpoch);

579:             emit ScaleUpdated(currentScale);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

754:             compoundedDeposit = initialDeposit.mul(P).div(snapshot_P).div(SCALE_FACTOR);

```

### <a name="GAS-11"></a>[GAS-11] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

*Instances (1)*:
```solidity
File: TroveManager.sol

715:     function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {

```

### <a name="GAS-12"></a>[GAS-12] Use Custom Errors
[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

*Instances (58)*:
```solidity
File: ActivePool.sol

85:         require(!addressesSet, "Can call setAddresses only once");

93:         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

111:             require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

127:         require(_bps <= 10_000, "Invalid BPS value");

133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

145:         require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

```

```solidity
File: BorrowerOperations.sol

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

```solidity
File: LQTY/CommunityIssuance.sol

70:         require(!initialized, "issuance has been initialized");

102:         require(amount != 0, "cannot fund 0");

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: LQTY/LQTYStaking.sol

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

```

```solidity
File: LUSDToken.sol

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:         require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:         require(!mintingPaused, "LUSDToken: Minting is currently paused");

```

```solidity
File: ReaperVaultERC4626.sol

270:         require(y != 0, "Division by 0");

```

```solidity
File: ReaperVaultV2.sol

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

```solidity
File: StabilityPool.sol

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

```

### <a name="GAS-13"></a>[GAS-13] Don't initialize variables with default value

*Instances (18)*:
```solidity
File: ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

117:         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

165:         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

```

```solidity
File: ReaperVaultV2.sol

128:         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {

264:         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

369:             uint256 totalLoss = 0;

371:             uint256 vaultBalance = 0;

372:             for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {

```

```solidity
File: StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```

### <a name="GAS-14"></a>[GAS-14] Long revert strings

*Instances (34)*:
```solidity
File: ActivePool.sol

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

133:         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

```

```solidity
File: BorrowerOperations.sol

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
File: LQTY/CommunityIssuance.sol

133:         require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");

```

```solidity
File: LQTY/LQTYStaking.sol

262:         require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```

```solidity
File: LUSDToken.sol

362:         require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");

377:         require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");

390:         require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");

394:         require(!mintingPaused, "LUSDToken: Minting is currently paused");

```

```solidity
File: ReaperVaultV2.sol

150:         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");

321:         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

436:         require(loss <= allocation, "Strategy loss cannot be greater than allocation");

619:         require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");

```

```solidity
File: StabilityPool.sol

849:         require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");

853:         require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");

865:             require(ICR >= collMCR, "StabilityPool: Cannot withdraw while there are troves with ICR < MCR");

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:         require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```

### <a name="GAS-15"></a>[GAS-15] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (21)*:
```solidity
File: ActivePool.sol

30:     string constant public NAME = "ActivePool";

```

```solidity
File: BorrowerOperations.sol

21:     string constant public NAME = "BorrowerOperations";

```

```solidity
File: LQTY/CommunityIssuance.sol

19:     string constant public NAME = "CommunityIssuance";

```

```solidity
File: LQTY/LQTYStaking.sol

23:     string constant public NAME = "LQTYStaking";

```

```solidity
File: ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

```solidity
File: ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: StabilityPool.sol

150:     string constant public NAME = "StabilityPool";

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: TroveManager.sol

48:     uint constant public SECONDS_IN_ONE_MINUTE = 60;

53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

61:     uint constant public BETA = 2;

```

### <a name="GAS-16"></a>[GAS-16] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (27)*:
```solidity
File: ActivePool.sol

278:         if (vars.netAssetMovement > 0) {

```

```solidity
File: BorrowerOperations.sol

128:         assert(MIN_NET_DEBT > 0);

197:         assert(vars.compositeDebt > 0);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

337:         if (!_isDebtIncrease && _LUSDChange > 0) {

552:         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

```

```solidity
File: LQTY/LQTYStaking.sol

152:         if (_LQTYamount > 0) {

181:         if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

191:         if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  

270:         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```

```solidity
File: LUSDToken.sol

279:         if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {

```

```solidity
File: ReaperVaultV2.sol

502:         } else if (_roi > 0) {

518:         } else if (vars.available > 0) {

```

```solidity
File: StabilityPool.sol

591:         assert(newP > 0);

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:         require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```

```solidity
File: TroveManager.sol

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

### <a name="GAS-17"></a>[GAS-17] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (1)*:
```solidity
File: BorrowerOperations.sol

432:     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

```