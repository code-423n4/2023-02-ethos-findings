
# Gas

| | Issue index |
| ----------- | ----------- |
| 1 | [Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate](#multiple-`address`/`id`-`mappings`-can-be-combined-into-a-single-`mapping`-of-an-`address`/`id`-to-a-`struct`,-where-appropriate) |
| 2 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#`<x>-+=-<y>`-costs-more-gas-than-`<x>-=-<x>-+-<y>`-for-state-variables) |
| 3 | [Redeclaring variables each loop iteration consumes extra gas](#redeclaring-variables-each-loop-iteration-consumes-extra-gas) |
| 4 | [Increments at loops can be `unchecked{++i}` to save gas](#increments-at-loops-can-be-`unchecked{++i}`-to-save-gas) |
| 5 | [`require()`/`revert()` strings longer than 32 bytes cost extra gas](#`require()`/`revert()`-strings-longer-than-32-bytes-cost-extra-gas) |
| 6 | [`bytes` constants are more efficient than `string` constants](#`bytes`-constants-are-more-efficient-than-`string`-constants) |
| 7 | [Should use arguments instead of state variable](#should-use-arguments-instead-of-state-variable) |
| 8 | [Superfluous event fields can be omitted](#superfluous-event-fields-can-be-omitted) |
| 9 | [`abi.encode()` is less gas efficient than `abi.encodePacked()`](#`abi.encode()`-is-less-gas-efficient-than-`abi.encodepacked()`) |
| 10 | [Splitting `require()` / `assert()` statements that use `&&` saves gas](#splitting-`require()`-/-`assert()`-statements-that-use-`&&`-saves-gas) |
| 11 | [Use of `safeMath` can be avoided to save gas](#use-of-`safemath`-can-be-avoided-to-save-gas) |
| 12 | [Use Custom Errors consumes less gas than regular `require` strings](#use-custom-errors-consumes-less-gas-than-regular-`require`-strings) |
| 13 | [Use assembly to check for `address(0)` for gas saves](#use-assembly-to-check-for-`address(0)`-for-gas-saves) |
| 14 | [Initializing variables with default value wastes extra gas](#initializing-variables-with-default-value-wastes-extra-gas) |
| 15 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-`payable`) |
| 16 | [`unchecked` block can be used to save gas](#`unchecked`-block-can-be-used-to-save-gas) |
| 17 | [Explicit named return can be removed](#explicit-named-return-can-be-removed) |
| 18 | [decimals can be set as `uint8` for convention and to save 1 storage slot](#decimals-can-be-set-as-`uint8`-for-convention-and-to-save-1-storage-slot) |
| 19 | [value of `distributionPeriod` can be set as default value to save gas](#value-of-`distributionperiod`-can-be-set-as-default-value-to-save-gas) |
| 20 | [Remove testing libs for gas saves](#remove-testing-libs-for-gas-saves) |

## Multiple `address`/`ID` `mappings` can be combined into a single `mapping` of an `address`/`ID` to a `struct`, where appropriate

if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s `keccak256` hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

Both mapping being used in the same functions mostly consider making them a struct instead 

- [StabilityPool.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L214)
    mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
- [StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L223)
    mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
- - - - - -
- [TroveManager.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L86)
    mapping (address => mapping (address => Trove)) public Troves;
- [TroveManager.sol#L88](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L88)
    mapping (address => uint) public totalStakes;
- [TroveManager.sol#L91](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L91)
    mapping (address => uint) public totalStakesSnapshot;
- [TroveManager.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L94)
    mapping (address => uint) public totalCollateralSnapshot;
- [TroveManager.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L104)
    mapping (address => uint) public L_Collateral;
- [TroveManager.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L105)
    mapping (address => uint) public L_LUSDDebt;
- [TroveManager.sol#L109](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L109)
    mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;
- [TroveManager.sol#L116](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L116)
    mapping (address => address[]) public TroveOwners;
- [TroveManager.sol#L119](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L119)
    mapping (address => uint) public lastCollateralError_Redistribution;
- [TroveManager.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L120)
    mapping (address => uint) public lastLUSDDebtError_Redistribution;
- - - - - -
- [ActivePool.sol#L41](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L41)
    mapping (address => uint256) internal collAmount;  // collateral => amount tracker
- [ActivePool.sol#L42](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L42)
    mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker
- [ActivePool.sol#L44](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L44)
    mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
- [ActivePool.sol#L45](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L45)
    mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
- [ActivePool.sol#L46](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L46)
    mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
- [ActivePool.sol#L47](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L47)
    mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
- - - - - -
- [StabilityPool.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L167)
    mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
- [StabilityPool.sol#L179](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L179)
        mapping (address => uint) S;
- [StabilityPool.sol#L186](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L186)
    mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct
- [StabilityPool.sol#L187](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L187)
    mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct
- [StabilityPool.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L228)
    mapping (address => uint) public lastCollateralError_Offset;
- - - - - -
- [StabilityPool.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L214)
    mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
- [StabilityPool.sol#L223](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L223)
    mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
- - - - - -

- [LQTYStaking.sol#L28](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28)
    mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked
- [LQTYStaking.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L32)
    mapping (address => Snapshot) public snapshots; 
- [LQTYStaking.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L35)
        mapping (address => uint) F_Collateral_Snapshot;
- - - - - -
- [LUSDToken.sol#L54](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L54)
    mapping (address => uint256) private _nonces;
- [LUSDToken.sol#L57](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L57)
    mapping (address => uint256) private _balances;
- [LUSDToken.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L58)
    mapping (address => mapping (address => uint256)) private _allowances;  
- [LUSDToken.sol#L62](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L62)
    mapping (address => bool) public troveManagers;
- [LUSDToken.sol#L63](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L63)
    mapping (address => bool) public stabilityPools;
- [LUSDToken.sol#L64](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L64)
    mapping (address => bool) public borrowerOperations;

## `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using the addition operator instead of plus-equals saves gas. Same goes for other operands like `-=`.

- [ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
        totalAllocBPS += _allocBPS;
- [ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
        totalAllocBPS += _allocBPS;

- [ReaperVaultV2.sol#L450](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L450)
        stratParams.losses += loss;
- [ReaperVaultV2.sol#L505](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L505)
            strategy.gains += vars.gain;
- [ReaperVaultV2.sol#L520](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L520)
            strategy.allocated += vars.credit;
- [ReaperVaultV2.sol#L521](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L521)
            totalAllocated += vars.credit;

- [ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
        totalAllocBPS -= strategies[_strategy].allocBPS;
- [ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
        totalAllocBPS -= strategies[_strategy].allocBPS;
- [ReaperVaultV2.sol#L395](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L395)
                strategies[stratAddr].allocated -= actualWithdrawn;
- [ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)
                totalAllocated -= actualWithdrawn;
- [ReaperVaultV2.sol#L444](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L444)
                stratParams.allocBPS -= bpsChange;
- [ReaperVaultV2.sol#L445](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L445)
                totalAllocBPS -= bpsChange;
- [ReaperVaultV2.sol#L451](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L451)
        stratParams.allocated -= loss;
- [ReaperVaultV2.sol#L452](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L452)
        totalAllocated -= loss;
- [ReaperVaultV2.sol#L514](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L514)
                strategy.allocated -= vars.debtPayment;
- [ReaperVaultV2.sol#L515](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L515)
                totalAllocated -= vars.debtPayment;


## Redeclaring variables each loop iteration consumes extra gas

Rather than re assigning values to the variables each iterations, at for loops it is being redeclared each variable all over again. 

-[StabilityPool.sol#L352-L353](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L352-L353)

-[StabilityPool.sol#L398-L399](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L398-L399)

-[StabilityPool.sol#L832-L833](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L832-L833)

-[StabilityPool.sol#L860-L864](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L860-L864)

-[TroveManager.sol#L610](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L610)

-[TroveManager.sol#L819](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L819)

-[LQTYStaking.sol#L207-L208](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L207-L208)

-[LQTYStaking.sol#L229](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L229)

-[LQTYStaking.sol#L242](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L242)

-[ReaperStrategyGranarySupplyOnly.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120)

-[ReaperVaultV2.sol#L265](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L265)


## Increments at loops can be `unchecked{++i}` to save gas

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline without an extra function. This extra function is used in some parts of the [code](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L77), however in some locations it is not used

- [CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/CollateralConfig.sol#L56)
        for(uint256 i = 0; i < _collaterals.length; i++) {
- [ActivePool.sol#L108](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L108)
        for(uint256 i = 0; i < numCollaterals; i++) {
- [TroveManager.sol#L608](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L608)
        for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
- [TroveManager.sol#L690](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L690)
        for (vars.i = 0; vars.i < _n; vars.i++) {
- [TroveManager.sol#L808](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L808)
        for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
- [TroveManager.sol#L882](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L882)
        for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
- [TroveManager.sol#L934](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L934)
    * The redeemer swaps (debt - liquidation reserve) LUSD for (debt - liquidation reserve) worth of collateral, so the LUSD liquidation reserve left corresponds to the remaining debt.
- [StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L351)
        for (uint i = 0; i < numCollaterals; i++) {
- [StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L397)
        for (uint i = 0; i < numCollaterals; i++) {
- [StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L640)
        for (uint i = 0; i < assets.length; i++) {
- [StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L810)
            for (uint i = 0; i < collaterals.length; i++) {
- [StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L831)
        for (uint i = 0; i < collaterals.length; i++) {
- [StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L859)
        for (uint i = 0; i < numCollaterals; i++) {
- [LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
        for (uint i = 0; i < assets.length; i++) {
- [LQTYStaking.sol#L228](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228)
        for (uint i = 0; i < collaterals.length; i++) {
- [LQTYStaking.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240)
        for (uint i = 0; i < numCollaterals; i++) {
- [ReaperVaultV2.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L128)v4.sol#L77](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L77)
        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
- [ReaperStrategyGranarySupplyOnly.sol#L117](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117)
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
- [ReaperStrategyGranarySupplyOnly.sol#L165](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L165)
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {

        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
- [ReaperVaultV2.sol#L264](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L264)
        for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
- [ReaperVaultV2.sol#L372](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L372)
            for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
- [ReaperBaseStrategy

## `require()`/`revert()` strings longer than 32 bytes cost extra gas

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas

- [BorrowerOperations.sol#L563](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L563)

- [ActivePool.sol#L324](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L324)
- [ActivePool.sol#L334](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L334)
- [ActivePool.sol#L341](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L341)

- [LQTYStaking.sol#L257](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L257)

- [LUSDToken.sol#L136](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L136)
- [LUSDToken.sol#L350](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L350)
- [LUSDToken.sol#L371](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L371)
- [LUSDToken.sol#L386](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L386)

- [ReaperBaseStrategyv4.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L193)



## `bytes` constants are more efficient than `string` constants

If data can fit into 32 bytes, then you should use `bytes32` datatype rather than bytes or strings as it is cheaper in solidity.

- [BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L21)
    string constant public NAME = "BorrowerOperations";
- [ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L30)
    string constant public NAME = "ActivePool";
- [StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L150)
    string constant public NAME = "StabilityPool";
- [CommunityIssuance.sol#L19](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)
    string constant public NAME = "CommunityIssuance";
- [LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)
    string constant public NAME = "LQTYStaking";
- [LUSDToken.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L32)
    string constant internal _NAME = "LUSD Stablecoin";
- [LUSDToken.sol#L33](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L33)
    string constant internal _SYMBOL = "LUSD";
- [LUSDToken.sol#L34](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L34)
    string constant internal _VERSION = "1";

## Should use arguments instead of state variable

This removes an extra SLOAD 

- [ReaperVaultV2.sol#L581](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L581)


## Superfluous event fields can be omitted

`block.timestamp` and `block.number` are added to event information by default so adding them manually wastes extra gas.

- [TroveManager.sol#L1505](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1505)
            emit LastFeeOpTimeUpdated(block.timestamp);

## `abi.encode()` is less gas efficient than `abi.encodePacked()`

Changing the `abi.encode` function to `abi.encodePacked` can save gas since the `abi.encode` function pads extra null `bytes` at the end of the call data, which is unnecessary. Also, in general, `abi.encodePacked` is more gas-efficient.

- [LUSDToken.sol#L284](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L284)
                         domainSeparator(), keccak256(abi.encode(
- [LUSDToken.sol#L305](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L305)
        return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

## Splitting `require()` / `assert()` statements that use `&&` saves gas

Instead of using the `&&` operator in a single require statement to check multiple conditions, consider using multiple require statements with 1 condition per require statement (saving 3 gas per `&`).

- [BorrowerOperations.sol#L301](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L301)
        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
- [BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L653)
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
- [TroveManager.sol#L1279](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1279)
        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
- [TroveManager.sol#L1342](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1342)
        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
- [TroveManager.sol#L1539](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L1539)
        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

- [LUSDToken.sol#L348](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L348)
            _recipient != address(0) && 
- [LUSDToken.sol#L353](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L353)
            !stabilityPools[_recipient] &&
- [LUSDToken.sol#L354](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L354)
            !troveManagers[_recipient] &&

## Use of `safeMath` can be avoided to save gas

In other parts of the project solidity version `^0.8` it's being used and has overflow protection by default so it's not needed `safeMath`.

- [ActivePool.sol#L27](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L27)
    using SafeMath for uint256;
- [CommunityIssuance.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L15)
    using SafeMath for uint;
- [LQTYStaking.sol#L20](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L20)
    using SafeMath for uint;
- [LUSDToken.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L29)
    using SafeMath for uint256;


## Use Custom Errors consumes less gas than regular `require` strings

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/) Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

- [BorrowerOperations.sol#L561](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L561)
        require(
- [ActivePool.sol#L314](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L314)
        require(
- [ActivePool.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L321)
        require(
- [ActivePool.sol#L329](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L329)
        require(
- [ActivePool.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L338)
        require(
- [LQTYStaking.sol#L253](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L253)
        require(
- [LUSDToken.sol#L134](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L134)
        require(
- [LUSDToken.sol#L347](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L347)
        require(
- [LUSDToken.sol#L352](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L352)
        require(
- [LUSDToken.sol#L367](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L367)
        require(
- [LUSDToken.sol#L384](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L384)
        require(
- [ReaperVaultV2.sol#L404](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L404)
            require(
- [ReaperBaseStrategyv4.sol#L191](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L191)
        require(
- [LUSDToken.sol#L280](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L280)
            revert('LUSD: Invalid s value');

## Use assembly to check for `address(0)` for gas saves

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

Saves 6 gas per instance

- [ReaperVaultV2.sol#L151](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L151)
        require(_strategy != address(0), "Invalid strategy address");
- [ReaperVaultV2.sol#L629](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L629)
        require(newTreasury != address(0), "Invalid address");
- [ReaperStrategyGranarySupplyOnly.sol#L167](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167)
            require(step[0] != address(0));
- [ReaperStrategyGranarySupplyOnly.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L168)
            require(step[1] != address(0));


## Initializing variables with default value wastes extra gas

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

- [CollateralConfig.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/CollateralConfig.sol#L18)
    bool public initialized = false;
- [ActivePool.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L32)
    bool public addressesSet = false;
- [CommunityIssuance.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21)
    bool public initialized = false;
- [LUSDToken.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L37)
    bool public mintingPaused = false;

## Functions guaranteed to revert when called by normal users can be marked `payable`

`NOTE`: None of these findings where found by [4naly3er output - Gas](https://gist.github.com/Picodes/deafcaead63bb0d725e4e0c125aa10ae)

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are:
CALLVALUE (2), DUP1 (3), ISZERO (3), PUSH2 (3), JUMPI (10), PUSH1 (3), DUP1 (3), REVERT(0), JUMPDEST (1), POP (2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

- [TroveManager.sol#L250](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/TroveManager.sol#L250)
        require(msg.sender == owner);

## `unchecked` block can be used to save gas

Where is not expected to overflow, `unchecked` can be used

- [ReaperStrategyGranarySupplyOnly.sol#L81](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81)
- [ReaperStrategyGranarySupplyOnly.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93)
- [ReaperStrategyGranarySupplyOnly.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100)
- [ReaperStrategyGranarySupplyOnly.sol#L132](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132)
- [ReaperStrategyGranarySupplyOnly.sol#L133](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133)
- [ReaperStrategyGranarySupplyOnly.sol#L136](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136)
- [ReaperVaultERC4626.sol#L82](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82)
- [ReaperVaultERC4626.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125)
- [ReaperVaultERC4626.sol#L273](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L273)
- [ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
- [ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
- [ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
- [ReaperVaultV2.sol#L419](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultV2.sol#L419)

## Explicit named return can be removed

- [ReaperVaultERC4626.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29)
- [ReaperVaultERC4626.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37)
- [ReaperVaultERC4626.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51)
- [ReaperVaultERC4626.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66)
- [ReaperVaultERC4626.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)
- [ReaperVaultERC4626.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96)
- [ReaperVaultERC4626.sol#L122](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)
- [ReaperVaultERC4626.sol#L220](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
- [ReaperVaultERC4626.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/34ba1e7dd2d259f06f9d5fa13f5d96be7829ff34/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)

## decimals can be set as `uint8` for convention and to save 1 storage slot

- [CollateralConfig.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/CollateralConfig.sol#L29)

## value of `distributionPeriod` can be set as default value to save gas

- [CommunityIssuance.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L58)

## Remove testing libs for gas saves

- [LQTYStaking.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9)
- [LUSDToken.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/LUSDToken.sol#L9)
- [StabilityPool.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/StabilityPool.sol#L18)
- [ActivePool.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/ActivePool.sol#L15)
- [BorrowerOperations.sol#L15](https://github.com/code-423n4/2023-02-ethos/blob/891252b407033489140de63ea502b053d5dc860c/Ethos-Core/contracts/BorrowerOperations.sol#L15)