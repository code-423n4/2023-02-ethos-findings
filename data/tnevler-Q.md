# Report
## Non-Critical Issues ##
### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

1. ```return singleLiquidation;``` [L353](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L353) 
1. ```return zeroVals;``` [L433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L433) 
1. ```return singleLiquidation;``` [L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L436) 
1. ```return newTotals;``` [L912](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L912) 
1. ```return _addTroveOwnerToArray(_borrower, _collateral);``` [L1318](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1318) 
1. ```return index;``` [L1332](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1332) 
1. ```return (collGainPerUnitStaked, LUSDLossPerUnitStaked);``` [L543](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L543) 
1. ```return _getCollateralGainFromSnapshots(initialDeposit, snapshots);``` [L632](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L632) 
1. ```return address(token);``` [L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L30) 
1. ```return balance();``` [L38](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L38) 
1. ```return (assets * totalSupply()) / _freeFunds();``` [L53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L53) 
1. ```return (shares * _freeFunds()) / totalSupply();``` [L68](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L68) 
1. ```if (tvlCap == type(uint256).max) return type(uint256).max;``` [L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L81) 
1. ```return tvlCap - balance();``` [L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82) 
1. ```return convertToShares(assets);``` [L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L97) 
1. ```shares = _deposit(assets, receiver);``` [L111](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L111) 
1. ```if (tvlCap == type(uint256).max) return type(uint256).max;``` [L124](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L124) 
1. ```if (totalSupply() == 0) return shares;``` [L140](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L140) 
1. ```return convertToAssets(balanceOf(owner));``` [L166](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L166) 
1. ```if (totalSupply() == 0 || _freeFunds() == 0) return 0;``` [L184](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L184) 
1. ```return balanceOf(owner);``` [L221](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L221) 
1. ```return convertToAssets(shares);``` [L241](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L241) 
1. ```return balanceOfWant();``` [L106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L106) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Use of immutable instead of constant keccak expression
**Context:**

1. ```bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");``` [L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73) 
1. ```bytes32 public constant STRATEGIST = keccak256("STRATEGIST");``` [L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74) 
1. ```bytes32 public constant GUARDIAN = keccak256("GUARDIAN");``` [L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75) 
1. ```bytes32 public constant ADMIN = keccak256("ADMIN");``` [L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76) 
1. ```bytes32 public constant KEEPER = keccak256("KEEPER");``` [L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49) 
1. ```bytes32 public constant STRATEGIST = keccak256("STRATEGIST");``` [L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50) 
1. ```bytes32 public constant GUARDIAN = keccak256("GUARDIAN");``` [L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51) 
1. ```bytes32 public constant ADMIN = keccak256("ADMIN");``` [L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52) 

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/contracts.html#constant-and-immutable-state-variables) for a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. It is recommended to use immutable instead. 

### [N-3]: Variable is unused
**Context:**

```uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log``` [L384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L384) 

### [N-4]: Follow Solidity standard naming conventions
**Context:**

1. ```struct LocalVariables_adjustTrove {``` [L47](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47) 
1. ```struct LocalVariables_openTrove {``` [L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L66) 
1. ```mapping (address => uint256) internal collAmount;  // collateral => amount tracker``` [L41](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41) (collAmount should start with _)
1. ```mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker``` [L42](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L42) (LUSDDebt should start with _)
1. ```mapping (address => uint256) internal collAmounts;  // deposited collateral tracker``` [L167](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167) (collAmounts should start with _)
1. ```uint256 internal totalLUSDDeposits;``` [L170](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L170) (totalLUSDSeposits should start with _)
1. ```mapping (address => uint) public lastCollateralError_Offset;``` [L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228) (Variable name must be in mixedCase)
1. ```uint public lastLUSDLossError_Offset;``` [L229](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L229) (Variable name must be in mixedCase)
1. ```function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {``` [L410](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L410) (Function name must be in mixedCase)
1. ```IERC20 public OathToken;``` [L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L37) (Variable name must be in mixedCase)
1. ```function sendOath(address _account, uint _OathAmount) external override {``` [L124](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124) (Variable name _OathAmount must be in mixedCase)
1. ```function increaseF_Collateral(address _collateral, uint _collFee) external override {``` [L177](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L177) (Function name must be in mixedCase)
1. ```function increaseF_LUSD(uint _LUSDFee) external override {``` [L187](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L187) (Function name must be in mixedCase)
1. ```function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {``` [L269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L269) (roundUpDiv should start with _)
1. ```uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;``` [L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35) (LENDER_REFERRAL_CODE_NONE should start with _)

**Description:**

The above codes don't follow [Solidity's standard naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).  
- Internal and private functions should use mixedCase format starting with an underscore.
- Structs should be named using the CapWords style. Examples: MyCoin, Position, PositionXY.
- Events should be named using the CapWords style. Examples: Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer.
- Functions should use mixedCase. Examples: getBalance, transfer, verifyOwner, addMember, changeOwner.
- Function arguments should use mixedCase. Examples: initialSupply, account, recipientAddress, senderAddress, newOwner.
- Local and State Variable should use mixedCase. Examples: totalSupply, remainingSupply, balancesOf, creatorAddress, isPreSale, tokenExchangeRate.
- Constants should be named with all capital letters with underscores separating words. Examples: MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION.
- Modifier should use mixedCase. Examples: onlyBy, onlyAfter, onlyDuringThePreSale.
- Enums, in the style of simple type declarations, should be named using the CapWords style. Examples: TokenGroup, Frame, HashStyle, CharacterLocation.

### [N-5]: Wrong order of functions
**Context:**

1. ```function sendCollateral(address _collateral, address _account, uint _amount) external override {``` [L171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L171) (external function can not go after external view function)
1. ```function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {``` [L258](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258) (extarnal function can not go after public function)
1. ```function maxDeposit(address) external view override returns (uint256 maxAssets) {``` [L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79) (external function can not go after public function)
1. ```function updateVeloSwapPath(``` [L147](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L147) (external function can not go after internal function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-6]: NatSpec comments should be increased in contracts
**Context:**

All contracts.

**Description:**

https://docs.soliditylang.org/en/v0.8.19/natspec-format.html

**Recommendation:**

Increase NatSpec comments in all contracts.

### [N-7]: Line is too long
**Context:**

1. ```event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);``` [L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L105) 
1. ```function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {``` [L172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172) 
1. ```vars.LUSDFee = _triggerBorrowingFee(contractsCache.troveManager, contractsCache.lusdToken, _LUSDAmount, _maxFeePercentage);``` [L190](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L190) 
1. ```_withdrawLUSD(contractsCache.activePool, contractsCache.lusdToken, _collateral, msg.sender, _LUSDAmount, vars.netDebt);``` [L233](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L233) 
1. ```_withdrawLUSD(contractsCache.activePool, contractsCache.lusdToken, _collateral, gasPoolAddress, LUSD_GAS_COMPENSATION, LUSD_GAS_COMPENSATION);``` [L235](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L235) 
1. ```emit TroveUpdated(msg.sender, _collateral, vars.compositeDebt, _collAmount, vars.stake, BorrowerOperation.openTrove);``` [L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L237) 
1. ```function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {``` [L248](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L248) 
1. ```function withdrawColl(address _collateral, uint _collWithdrawal, address _upperHint, address _lowerHint) external override {``` [L254](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L254) 
1. ```function withdrawLUSD(address _collateral, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {``` [L259](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L259) 
1. ```function repayLUSD(address _collateral, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {``` [L264](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L264) 
1. ```function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {``` [L268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268) 
1. ```_adjustTrove(msg.sender, _collateral, _collTopUp, _collWithdrawal, _LUSDChange, _isDebtIncrease, _upperHint, _lowerHint, _maxFeePercentage);``` [L272](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L272) 
1. ```function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {``` [L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282) 
1. ```vars.LUSDFee = _triggerBorrowingFee(contractsCache.troveManager, contractsCache.lusdToken, _LUSDChange, _maxFeePercentage);``` [L312](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L312) 
1. ```(vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);``` [L343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343) 
1. ```function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {``` [L419](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419) 
1. ```function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {``` [L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511) 
1. ```function _repayLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSD) internal {``` [L517](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L517) 
1. ```function _requireSufficientCollateralBalanceAndAllowance(address _user, address _collateral, uint _collAmount) internal view {``` [L528](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L528) 
1. ```require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");``` [L529](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L529) 
1. ```require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");``` [L530](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L530) 
1. ```function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {``` [L546](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L546) 
1. ```require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");``` [L637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L637) 
1. ```function _requireSufficientLUSDBalance(ILUSDToken _lusdToken, address _borrower, uint _debtRepayment) internal view {``` [L644](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L644) 
1. ```require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");``` [L645](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L645) 
1. ```(uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);``` [L675](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L675) 
1. ```(uint newColl, uint newDebt) = _getNewTroveAmounts(_coll, _debt, _collChange, _isCollIncrease, _debtChange, _isDebtIncrease);``` [L697](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L697) 
1. ```mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute``` [L47](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L47) 
1. ```function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {``` [L144](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144) 
1. ```vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);``` [L258](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258) 
1. ```if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {``` [L264](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L264) 
1. ```} else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {``` [L269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L269) 
1. ```IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));``` [L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282) 
1. ```// profit is ultimately (coll at hand) + (coll allocated to yield generator) - (recorded total coll Amount in pool)``` [L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L287) 
1. ```vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);``` [L288](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L288) 
1. ```uint LUSDLossPerUnitStaked) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, _debtToOffset, totalLUSD);``` [L474](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L474) 
1. ```function _updateRewardSumAndProduct(address _collateral, uint _collGainPerUnitStaked, uint _LUSDLossPerUnitStaked) internal {``` [L547](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L547) 
1. ```function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {``` [L626](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626) 
1. ```function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {``` [L637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637) 
1. ```function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {``` [L657](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L657) 
1. ```vars.secondPortion = epochToScaleToSum[vars.epochSnapshot][vars.scaleSnapshot.add(1)][_collateral].div(SCALE_FACTOR);``` [L671](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L671) 
1. ```vars.gain = _initialDeposit.mul(vars.firstPortion.add(vars.secondPortion)).div(vars.P_Snapshot).div(DECIMAL_PRECISION);``` [L673](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L673) 
1. ```function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {``` [L203](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203) 
1. ```shares = previewWithdraw(assets); // previewWithdraw() rounds up so exactly "assets" are withdrawn and not 1 wei less``` [L207](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L207) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
