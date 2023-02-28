# Gas Optimization

## [G-01] **internal** functions only called once can be **inlined** to save gas

### Summary

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Details

If function called once in the contract will be inline to save gas

There are **57** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Core/contracts/BorrowerOperations.sol

438    function _getCollChange(
439       uint _collReceived,
440       uint _requestedCollWithdrawal
441     )
442:       internal
443        pure
444        returns(uint collChange, bool isCollIncrease)


455    function _updateTroveFromAdjustment
456        (
457            ITroveManager _troveManager,
458            address _borrower,
459            address _collateral,
460            uint _collChange,
461            bool _isCollIncrease,
462            uint _debtChange,
463            bool _isDebtIncrease
464        )
465:           internal
466            returns (uint, uint)


476    function _moveTokensAndCollateralfromAdjustment
477      (
478         IActivePool _activePool,
479         ILUSDToken _lusdToken,
480         address _borrower,
481         address _collateral,
482         uint _collChange,
483         bool _isCollIncrease,
484         uint _LUSDChange,
485         bool _isDebtIncrease,
486         uint _netDebtChange
487      )
488:         internal


524:   function _requireValidCollateralAddress(address _collateral) internal view {
525        require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
526     }


533:    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
534        require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
535    }


537:    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {
538        require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
539    }


546:    function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {
547        uint status = _troveManager.getTroveStatus(_borrower, _collateral);
548        require(status != 1, "BorrowerOps: Trove is active");
549    }


551:    function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
552        require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
553    }


555    function _requireNotInRecoveryMode(
556        address _collateral,
557        uint _price,
558        uint256 _CCR,
559        uint256 _collateralDecimals
560:    ) internal view {


567:    function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {
568        require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
569    }


571    function _requireValidAdjustmentInCurrentMode
572    (
573        bool _isRecoveryMode,
574        address _collateral,
575        uint _collWithdrawal,
576        bool _isDebtIncrease,
577        LocalVariables_adjustTrove memory _vars
578    )
579:        internal
578        view


624:    function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {
625        require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
626    }


636:    function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {
637        require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
638    }


640:    function _requireCallerIsStabilityPool() internal view {
641        require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
642    }


661    function _getNewNominalICRFromTroveChange
662    (
663        uint _coll,
664        uint _debt,
665        uint _collChange,
666        bool _isCollIncrease,
667        uint _debtChange,
668        bool _isDebtIncrease,
669        uint256 _collateralDecimals
670    )
671         pure
672:        internal
673         returns (uint)



682    function _getNewICRFromTroveChange
683    (
684        uint _coll,
685        uint _debt,
686        uint _collChange,
687        bool _isCollIncrease,
688        uint _debtChange,
689        bool _isDebtIncrease,
690        uint _price,
691        uint256 _collateralDecimals
692    )
693         pure
694:        internal
695         returns (uint)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438-L441

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455-L466

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476-L488

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L524-L526

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533-L535

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L537-L539

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L546-L549

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551-L553

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L555-L560

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L567-L569

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L571-L578

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L624-L626

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L636-L638

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L640-L642

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661-L673

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L682-L695

```javascript
File: /Ethos-Core/contracts/TroveManager.sol

478    function _getCappedOffsetVals
479    (
480        uint _entireTroveDebt,
481        uint _entireTroveColl,
482        uint _price,
483        uint256 _MCR,
484        uint _collDecimals
485    )
486:       internal
487        pure
488        returns (LiquidationValues memory singleLiquidation)


671    function _getTotalsFromLiquidateTrovesSequence_NormalMode
672    (
673        IActivePool _activePool,
674        IDefaultPool _defaultPool,
675        address _collateral,
676        uint _price,
677        uint256 _MCR,
678        uint _LUSDInStabPool,
679        uint _n
680    )
681:        internal

785    function _getTotalFromBatchLiquidate_RecoveryMode
786    (
787        IActivePool _activePool,
788        IDefaultPool _defaultPool,
789        address _collateral,
790        uint _price,
791        uint _LUSDInStabPool,
792        address[] memory _troveArray
793    )
794:        internal
795         returns(LiquidationTotals memory totals)


864    function _getTotalsFromBatchLiquidate_NormalMode
865    (
866        IActivePool _activePool,
867        IDefaultPool _defaultPool,
868        address _collateral,
869        uint _price,
870        uint256 _MCR,
871        uint _LUSDInStabPool,
872        address[] memory _troveArray
873    )
874:      internal
875       returns(LiquidationTotals memory totals)


1082:  function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

1213:   function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {


1321:   function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

1339:   function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {

1516:   function _minutesPassedSinceLastFeeOp() internal view returns (uint) {

1538:   function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478-L488

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L671-L681

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L785-L795

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L864-L875

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1082

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1213

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1339

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1516

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1538

```javascript
File: /Ethos-Core/contracts/ActivePool.sol

320:  function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320

```javascript
File: /Ethos-Core/contracts/StabilityPool.sol

439: function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {


597: function _moveOffsetCollAndDebt(address _collateral, uint _collToAdd, uint _debtToOffset) internal {


637:  function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {

693:  function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {


729    function _getCompoundedDepositFromSnapshots(
730          uint initialDeposit,
731          Snapshots memory snapshots
732       )
733:         internal
734          view
735          returns (uint)


776:  function _sendLUSDtoStabilityPool(address _address, uint _amount) internal {

794:  function _sendLUSDToDepositor(address _depositor, uint LUSDWithdrawal) internal {

848:  function _requireCallerIsActivePool() internal view {

852:  function _requireCallerIsTroveManager() internal view {

856:  function _requireNoUnderCollateralizedTroves() internal {

869:  function _requireUserHasDeposit(uint _initialDeposit) internal pure {

873:  function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L439

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L597

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L693

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L729

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L776

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L794

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L848

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L852

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L856

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L869

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L873

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

251:  function _requireCallerIsTroveManagerOrActivePool() internal view {

261:  function _requireCallerIsBorrowerOperations() internal view {

265:  function _requireUserHasStake(uint currentStake) internal pure {

269:  function _requireNonZeroAmount(uint _amount) internal pure {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L261

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L269

```javascript
File: /Ethos-Core/contracts/LUSDToken.sol;

320:  function _mint(address account, uint256 amount) internal {

328:  function _burn(address account, uint256 amount) internal {

365:  function _requireCallerIsBOorTroveMorSP() internal view {

375:  function _requireCallerIsStabilityPool() internal view {

380:  function _requireCallerIsTroveMorSP() internal view {

393:  function _requireMintingNotPaused() internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L320

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L328

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L365

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L375

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L380

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L393

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

462:  function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462

```javascript
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

240:  function _liquidatePosition(uint256 _amountNeeded)
241:       internal

250:  function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);


276:  function _harvestCore(uint256 _debt) internal virtual returns (int256 roi, uint256 repayment);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L240

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L250

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L276

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

86:   function _liquidatePosition(uint256 _amountNeeded)
87:        internal


176:  function _deposit(uint256 toReinvest) internal {

187:  function _withdraw(uint256 _amount) internal {

206:  function _claimRewards() internal {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86-L87

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L187

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206

### Mitigation

instead of internal use inline.

## [GAS-2] Functions guaranteed to **revert** when called by normal users can be marked **payable**

### Summary

These are missed from **4naly3er**

https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#gas-9-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable

### Details

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

There are **7** instances of this issue.

### Github Permalinks

```javascript
File:/Ethos-Core/contracts/CollateralConfig.sol

46    function initialize(
47        address[] calldata _collaterals,
48        uint256[] calldata _MCRs,
49        uint256[] calldata _CCRs
50:    ) external override onlyOwner {


85    function updateCollateralRatios(
86        address _collateral,
87        uint256 _MCR,
88        uint256 _CCR
89:    ) external onlyOwner checkCollateral(_collateral) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L50

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85-L89

```javascript
File: /Ethos-Core/contracts/BorrowerOperations.sol

110    function setAddresses(
111        address _collateralConfigAddress,
112        address _troveManagerAddress,
113        address _activePoolAddress,
114        address _defaultPoolAddress,
115        address _stabilityPoolAddress,
116        address _gasPoolAddress,
117        address _collSurplusPoolAddress,
118        address _priceFeedAddress,
119        address _sortedTrovesAddress,
121        address _lusdTokenAddress,
122        address _lqtyStakingAddress
123    )
124        external
125        override
126        onlyOwner
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110-L126

```javascript

File: /Ethos-Core/contracts/ActivePool.sol

71    function setAddresses(
72        address _collateralConfigAddress,
73        address _borrowerOperationsAddress,
74        address _troveManagerAddress,
75        address _stabilityPoolAddress,
76        address _defaultPoolAddress,
77        address _collSurplusPoolAddress,
78        address _treasuryAddress,
79        address _lqtyStakingAddress,
80        address[] calldata _erc4626vaults
81     )
82         external
83:        onlyOwner
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L83

```javascript
File: /Ethos-Core/contracts/StabilityPool.sol

261    function setAddresses(
262        address _borrowerOperationsAddress,
263        address _collateralConfigAddress,
264        address _troveManagerAddress,
265        address _activePoolAddress,
266        address _lusdTokenAddress,
267        address _sortedTrovesAddress,
268        address _priceFeedAddress,
269        address _communityIssuanceAddress
270    )
271        external
272        override
273:        onlyOwner
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L273

```javascript
File: /Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61    function setAddresses
62    (
63      address _oathTokenAddress,
64        address _stabilityPoolAddress
65    )
66        external
67:        onlyOwner
68        override
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L68

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

66    function setAddresses
67    (
68        address _lqtyTokenAddress,
69        address _lusdTokenAddress,
70        address _troveManagerAddress,
70        address _borrowerOperationsAddress,
71        address _activePoolAddress,
72        address _collateralConfigAddress
73    )
74        external
75:       onlyOwner
76        override
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L76

### Mitigation

Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

## [G-3] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

### Summary

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

### Details

There are **9** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

//44: uint256 public totalAllocBPS;  State Variable
//45: uint256 public totalAllocated; State Variable

168: totalAllocBPS += _allocBPS;

194: totalAllocBPS -= strategies[_strategy].allocBPS;

196: totalAllocBPS += _allocBPS;

214: totalAllocBPS -= strategies[_strategy].allocBPS;

445: totalAllocBPS -= bpsChange;

396: totalAllocated -= actualWithdrawn;

452: totalAllocated -= loss;

515: totalAllocated -= vars.debtPayment;

521: totalAllocated += vars.credit;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

### Mitigation

Avoid compound assignment operator in **state variables**

## [G-4] USING FIXED BYTES IS CHEAPER THAN USING STRING

### Summary

Bytes and Strings are basically doing the same just in different formats and using **bytes is cheaper**

### Details

If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.
as long as you are certain such data won't be longer than 32 characters.

There are **8** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Core/contracts/BorrowerOperations.sol

21: string constant public NAME = "BorrowerOperations";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21

```javascript
File: /Ethos-Core/contracts/ActivePool.sol

30: string constant public NAME = "ActivePool";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30

```javascript
File: /Ethos-Core/contracts/StabilityPool.sol

150:  string constant public NAME = "StabilityPool";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150

```javascript
File: /Ethos-Core/contracts/LQTY/CommunityIssuance.sol

19: string constant public NAME = "CommunityIssuance";\
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

23: string constant public NAME = "LQTYStaking";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23

```javascript
File: /Ethos-Core/contracts/LQTY/LUSDToken.sol

32: string constant internal _NAME = "LUSD Stablecoin";

33: string constant internal _SYMBOL = "LUSD";

34: string constant internal _VERSION = "1";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34

### Mitigation

If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

## [G-5] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

### Details

There are **8** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

73:  bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:  bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:  bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:  bytes32 public constant ADMIN = keccak256("ADMIN");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76

```javascript
File:/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

49:  bytes32 public constant KEEPER = keccak256("KEEPER");

50:  bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:  bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:  bytes32 public constant ADMIN = keccak256("ADMIN");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52

### Mitigation

Expressions for constant values such as a call to keccak256(), should use **immutable** rather than **constant**

## [G-6] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

### Summary

Do not use return variables when a function returns

### Detials

There are **42** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Core/contracts/BorrowerOperations.sol

444: returns(uint collChange, bool isCollIncrease)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L444

```javascript
File: /Ethos-Core/contracts/TroveManager.sol

329: returns (LiquidationValues memory singleLiquidation)

369: returns (LiquidationValues memory singleLiquidation)

450: returns (uint debtToOffset, uint collToSendToSP, uint debtToRedistribute, uint collToRedistribute)

488: returns (LiquidationValues memory singleLiquidation)

593: returns(LiquidationTotals memory totals)

682: returns(LiquidationTotals memory totals)

795: returns(LiquidationTotals memory totals)

875: returns(LiquidationTotals memory totals)

899: internal pure returns(LiquidationTotals memory newTotals) {

1171: returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

1316: function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {

1321: function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L329

```javascript
File: /Ethos-Core/contracts/LQTY/CommunityIssuance.sol

84: function issueOath() external override returns (uint issuance) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

203: function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203

```javascript
File: /Ethos-Core/contracts/LUSDToken.sol

298: function _chainID() private pure returns (uint256 chainID) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L298

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

319: function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {

363: ) internal nonReentrant returns (uint256 value) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L363

```javascript
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol

29: function asset() external view override returns (address assetTokenAddress) {

37: function totalAssets() external view override returns (uint256 totalManagedAssets) {

51: function convertToShares(uint256 assets) public view override returns (uint256 shares) {

66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) {

79: function maxDeposit(address) external view override returns (uint256 maxAssets) {

96: function previewDeposit(uint256 assets) external view override returns (uint256 shares) {

110: function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

122: function maxMint(address) external view override returns (uint256 maxShares) {

139: function previewMint(uint256 shares) public view override returns (uint256 assets) {

154: function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

165: function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {

183: function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {

206: ) external override returns (uint256 shares) {

220: function maxRedeem(address owner) external view override returns (uint256 maxShares) {

240: function previewRedeem(uint256 shares) external view override returns (uint256 assets) {

262: ) external override returns (uint256 assets) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29

```javascript
File: /Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

95: function withdraw(uint256 _amount) external override returns (uint256 loss) {

109: function harvest() external override returns (int256 roi) {

243: returns (uint256 liquidatedAmount, uint256 loss);

250: function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);

276: function _harvestCore(uint256 _debt) internal virtual returns (int256 roi, uint256 repayment);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

89: returns (uint256 liquidatedAmount, uint256 loss)

104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

114: function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L89

## [G-7] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

### Summary

Consider making the stack variables before the loop which gonna save gas

### Detials

There are **25** instances of this issue.

### Github Permalinks

```javascript
File: /Ethos-Core/contracts/CollateralConfig.sol

56    for(uint256 i = 0; i < _collaterals.length; i++) {
57:      address collateral = _collaterals[i];

61:   Config storage config = collateralConfig[collateral];

63:   uint256 decimals = IERC20(collateral).decimals();
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56

```javascript
File: /Ethos-Core/contracts/TroveManager.sol

608     for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
609          // we need to cache it, because current user is likely going to be deleted
610:         address nextUser = _sortedTroves.getPrev(_collateral, vars.user);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L610

```javascript
File: /Ethos-Core/contracts/ActivePool.sol

109 :   address collateral = collaterals[i];

110:    address vault = _erc4626vaults[i];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L109

```javascript
File: /Ethos-Core/contracts/StabilityPool.sol

352: address collateral = assets[i];

353: uint amount = amounts[i];

398: address collateral = assets[i];

399: uint amount = amounts[i];

832: address collateral = collaterals[i];

833: uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];

860: address collateral = collaterals[i];

861: uint price = priceFeed.fetchPrice(collateral);

862: address lowestTrove = sortedTroves.getLast(collateral);

863: uint256 collMCR = collateralConfig.getCollateralMCR(collateral);

864: uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L352

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

207: address collateral = assets[i];

208: uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];

229: address collateral = collaterals[i];

242: address collateral = assets[i];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

265: address strategy = _withdrawalQueue[i];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L265

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

118: address[2] storage step = steps[i];

119: IERC20Upgradeable startToken = IERC20Upgradeable(step[0]);

120: uint256 amount = startToken.balanceOf(address(this));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L118-L120

### Mitigation

Make the variable outside the loop.

## [G-8] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

### Summary

These are missed from **4naly3er**

https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#gas-5-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated

### Details

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

There is **1** instance of this issue.

### Github Permalinks

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62    function initialize(
63         address _vault,
64:        address[] memory _strategists,
65:        address[] memory _multisigRoles,
66         IAToken _gWant
67     ) public initializer {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L67

## [G-9] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

### Summary

Contracts are allowed to override their parents functions and change the visibility from external to public and can save gas by doing so, public functions not called in the contract itself should be declared externals.

### Details

if a function is override public and not called in the contract should be declared external instead

There are **4** instance of this issue.

### Github Permalinks

```javascript
File:/Ethos-Core/contracts/TroveManager.sol

1045: function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1440: function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1456: function getBorrowingRate() public view override returns (uint) {

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

295: function getPricePerFullShare() public view returns (uint256) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295

## [G-10] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE

### Summary

state variables should not used in emit

### Details

There are **5** instance of this issue.

### Github Permalinks

```javascript
File:/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

//uint public totalOATHIssued; State Variable
//uint public rewardPerSecond; State Variable

94:  emit TotalOATHIssuedUpdated(totalOATHIssued);

116: emit LogRewardPerSecond(rewardPerSecond);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L41

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

//uint public totalLQTYStaked; State Variable

125: emit TotalLQTYStakedUpdated(totalLQTYStaked);

160: emit TotalLQTYStakedUpdated(totalLQTYStaked);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L125

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

//uint256 public tvlCap; state variables

581:  emit TvlCapUpdated(tvlCap);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581

## [G-11] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

### Summary

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

### Details

There are **6** instance of this issue.

### Github Permalinks

```javascript
File: /Ethos-Core/contracts/BorrowerOperations.sol

//check  _collAmount
231: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

//check  _collChange
497: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L231

```javascript
File: /Ethos-Core/contracts/ActivePool.sol

//check  _amount
210: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L210

```javascript
File: /Ethos-Core/contracts/LQTY/CommunityIssuance.sol

//check  amount
103: OathToken.transferFrom(msg.sender, address(this), amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

```javascript
File: /Ethos-Core/contracts/LQTY/LQTYStaking.sol

//check  _LQTYamount
128: lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L128

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

//check _amount
328:  token.safeTransferFrom(msg.sender, address(this), _amount);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L328

## [G-12] internal functions not called by the contract should be removed

### Summary

These are missed from **4naly3er**
https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae#GAS-13

### Details

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

There are **8** instance of this issue.

### Github Permalinks

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

104:     function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

114:     function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104

```javascript
File: /Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

63    function __ReaperBaseStrategy_init(
64        address _vault,
65        address _want,
66        address[] memory _strategists,
67        address[] memory _multisigRoles
68:    ) internal onlyInitializing {

189:     function _authorizeUpgrade(address) internal override {

203:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L68

```javascript
File: /Ethos-Vault/contracts/ReaperVaultV2.sol

659:    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

673:    function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L659

```javascript
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

74:    function _adjustPosition(uint256 _debt) internal override {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L74
