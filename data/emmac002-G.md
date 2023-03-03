# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES | 8 |
| [GAS-2] | USING UNCHECKED BLOCKS TO SAVE GAS | 6 |
| [GAS-3] | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION | 6 |
| [GAS-4] | SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS | 4 |
| [GAS-5] | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD | 2 |
| [GAS-6] | INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS | 1 |
| [GAS-7] | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE | 12 |
| [GAS-8] | EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT | 8 |
| [GAS-9] | USING BOOLS FOR STORAGE INCURS OVERHEAD | 3 |
| [GAS-10] | STACK VARIABLE USED AS A CHEAPER CACHE FOR A STATE VARIABLE IS ONLY USED ONCE | 1 |
| [GAS-11] | CACHE ARRAY LENGTH OUTSIDE OF LOOP | 7 |
| [GAS-12] | USE ASSEMBLY TO CHECK FOR ADDRESS(0) | 6 |
| [GAS-13] | USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD  | 5 |
| [GAS-14] | EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING | 2 |
| [GAS-15] | USE CONSTANTS FOR VARIABLES WHOSE VALUE IS KNOWN BEFOREHAND AND IS NEVER CHANGED | 1 |
| [GAS-16] | STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE | 3 |

### [GAS-1] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

*Instances (8)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

168:                 totalAllocBPS += _allocBPS;
196:                 totalAllocBPS += _allocBPS;
391:                 totalLoss += loss;    
450:                 stratParams.losses += loss;
505:                 strategy.gains += vars.gain;
520:                 strategy.allocated += vars.credit;
521:                 totalAllocated += vars.credit;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L391
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L450
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L505
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L520
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

133:                 toFree += profit;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133

### [GAS-2] USING UNCHECKED BLOCKS TO SAVE GAS
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

121:         roi = -int256(debt - amountFreed);
123:         roi = int256(amountFreed - debt);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L123

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

81:        uint256 toReinvest = wantBalance - _debt;
100:       loss = _amountNeeded - liquidatedAmount;
132:       uint256 profit = totalAssets - allocated;
136:       roi = -int256(allocated - totalAssets);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136

### [GAS-3] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

551:         require(totals.totalDebtInSequence > 0);
754:         require(totals.totalDebtInSequence > 0);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754


```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

180:         require(strategies[_strategy].activation != 0, "Invalid strategy address");
193:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

155:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
181:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181

### [GAS-4] SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS
Instead of using operator && on a single require. Using a two require can save more gas.

i.e. for require(version == 1 && _bytecodeHash[1] == bytes1(0), "zf"); use:
```
    require(version == 1);
	require(_bytecodeHash[1] == bytes1(0));
```

*Instances (4)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

653: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654: "Max fee percentage must be between 0.5% and 100%");

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654

```solidity
File: Ethos-Core/contracts/TroveManager.sol

539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

347: require(
348:  _recipient != address(0) && 
349: _recipient != address(this),
350:  "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"

352: require(
353:  !stabilityPools[_recipient] &&
354:  !troveManagers[_recipient] &&
355:  !borrowerOperations[_recipient],
356:  "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L350
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L353-L356

### [GAS-5] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

295:    function getPricePerFullShare() public view returns (uint256) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295

### [GAS-6] INTERNAL FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE REMOVED TO SAVE DEPLOYMENT GAS

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

432:         function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432

### [GAS-7] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*Instances (12)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:    function initialize(
47:        address[] calldata _collaterals,
48:        uint256[] calldata _MCRs,
49:        uint256[] calldata _CCRs
50:    ) external override onlyOwner 

85:    function updateCollateralRatios(
86:            address _collateral,
87:            uint256 _MCR,
88:            uint256 _CCR
89:    ) external onlyOwner checkCollateral(_collateral) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L50
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85-L89

```solidity
File: Ethos-Core/contracts/ActivePool.sol

46:  function setAddresses(
47:        address _collateralConfigAddress,
48:        address _borrowerOperationsAddress,
49:        address _troveManagerAddress,
50:        address _stabilityPoolAddress,
51:        address _defaultPoolAddress,
52:        address _collSurplusPoolAddress,
53:        address _treasuryAddress,
54:        address _lqtyStakingAddress,
55:        address[] calldata _erc4626vaults
56:    )
57:        external
58:        onlyOwner

132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214: function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L83
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L214

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:    function setAddresses(
262:        address _borrowerOperationsAddress,
263:        address _collateralConfigAddress,
264:           address _troveManagerAddress,
265:            address _activePoolAddress,
266:            address _lusdTokenAddress,
267:            address _sortedTrovesAddress,
268:            address _priceFeedAddress,
269:            address _communityIssuanceAddress
270:        )
271:        external
272:        override
273:        onlyOwner  
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L273

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

61:        function setAddresses
62:    (
63:        address _oathTokenAddress, 
64:        address _stabilityPoolAddress
65:    ) 
66:        external 
67:        onlyOwner 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L67

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

101:    function fund(uint amount) external onlyOwner {

120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:    function setAddresses
67:    (
68:        address _lqtyTokenAddress,
69:        address _lusdTokenAddress,
70:        address _troveManagerAddress, 
71:        address _borrowerOperationsAddress,
72:        address _activePoolAddress,
73:        address _collateralConfigAddress
74:    ) 
75:        external 
76:        onlyOwner 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L76

### [GAS-8] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

*Instances (8)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:         bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:         bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:         bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:         bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

49:         bytes32 public constant KEEPER = keccak256("KEEPER");
50:         bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:         bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:         bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52

### [GAS-9] USING BOOLS FOR STORAGE INCURS OVERHEAD
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead

*Instances (3)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

62:    mapping (address => bool) public troveManagers;
63:    mapping (address => bool) public stabilityPools;
64:    mapping (address => bool) public borrowerOperations;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L63
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L64

### [GAS-10] STACK VARIABLE USED AS A CHEAPER CACHE FOR A STATE VARIABLE IS ONLY USED ONCE
If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend:

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

1345:   uint length = TroveOwnersArrayLength;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1345

### [GAS-11] CACHE ARRAY LENGTH OUTSIDE OF LOOP

*Instances (7)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56:     for(uint256 i = 0; i < _collaterals.length; i++) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56

```solidity
File: Ethos-Core/contracts/TroveManager.sol

808:    for (vars.i = 0; vars.i < _troveArray.length; vars.i++)
882:    for (vars.i = 0; vars.i < _troveArray.length; vars.i++)

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

640:    for (uint i = 0; i < assets.length; i++)
810:    for (uint i = 0; i < collaterals.length; i++)
831:    for (uint i = 0; i < collaterals.length; i++) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

228:    for (uint i = 0; i < collaterals.length; i++)

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228

### [GAS-12] USE ASSEMBLY TO CHECK FOR ADDRESS(0)
Saves 6 gas per instance if using assembly to check for address(0)

e.g.
```
assembly {
    if iszero(_addr) {
        mstore(0x00, "zero address")
        revert(0x00, 0x20)
    }
}
```

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/ActivePool.sol

93:     require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

347:        require(
348:            _recipient != address(0) && 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L348

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

151:    require(_strategy != address(0), "Invalid strategy address");
629:    require(newTreasury != address(0), "Invalid address");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L151
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L629

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

167:    require(step[0] != address(0));
168:    require(step[1] != address(0));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168

### [GAS-13] USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

*Instances (5)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

35:     uint8 constant internal _DECIMALS = 18;
268:    uint8 v
407:    function decimals() external view override returns (uint8) {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L268
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L407

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

650:    function decimals() public view override returns (uint8) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L650

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
    
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L35

### [GAS-14] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L24

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

61:     constructor() initializer {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61

### [GAS-15] USE CONSTANTS FOR VARIABLES WHOSE VALUE IS KNOWN BEFOREHAND AND IS NEVER CHANGED
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

55:     uint256 public withdrawMaxLoss = 1;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L55

### [GAS-16] STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*Instances (3)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/ActivePool.sol

246:     vars.yieldGenerator = IERC4626(yieldGenerator[_collateral]);
279:     IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement));
280:     IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this));
282:     IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L246
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L279-L280
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282

### Tool
Manual Audit