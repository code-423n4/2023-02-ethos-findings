## Summary

### Gas Optimizations
|               | Issue                                                                                        | Instances | Total Gas Saved |
| ------------- |:-------------------------------------------------------------------------------------------- |:---------:|:---------------:|
| [G-01] | `require()`  statements that check input arguments should be at the top of the function      |     11     |      >2100       |
| [G-02] | Use  `require` instead of `assert`                                                           |    20     |        -      |
| [G-03] | `internal` functions only called once can be inlined to save gas                        |     28    |       560       |
| [G-04] |  Structs can be packed into fewer storage slots                                               |     6     |       12000       |
| [G-05] | Using fixed `bytes` is cheaper than using `string` |    6       |  -    |
| [G-06] |   State variables should be cached in stack variables rather than re-reading them from storage |    12     |        ~1200        |
| [G-07] | Splitting require() statements that use && saves gas                                         |     7     |       21       |
| [G-08] |  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate     |     3     |        -        |

Total: 93 instances over 8 issues with >15881 **gas** saved.

## Gas Optimizations:

### [G-01] `require()`  statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (**2100 gas**) in a function that may ultimately revert in the unhappy case.

_There are 11 instance of this issue:_

```Solidity
File: contracts/CollateralConfig.sol

66:             require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

69:             require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

94:             require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

97:             require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66

```
File: contracts/ActivePool.sol

93:     require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

107:    require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

111:    require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

127:    require(_bps <= 10_000, "Invalid BPS value");

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L93

```solidity
File: contracts/abstract/ReaperBaseStrategyv4.sol

97:        require(_amount != 0, "Amount cannot be zero");
98:        require(_amount <= balanceOf(), "Ammount must be less than balance")
       
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L97-L98

```solidity
File: contracts/ReaperVaultV2.sol

/// @audit change order of requires
125:    require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
        require(_strategy != address(0), "Invalid strategy address");
        require(strategies[_strategy].activation == 0, "Strategy already added");
        require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
        require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

// @audit-ok change order of requires
321:    require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
        require(_amount != 0, "Invalid amount");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150-L156


### [G-02] Use  `require` instead of `assert`

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, **uses up all the remaining gas and reverts all the changes made.**

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but **does refund all the remaining gas fees we offered to pay**. This is the most common Solidity function used by developers for debugging and error handling.

_There is 20 instance of this issue:_

```solidity
File: contracts/TroveManager.sol

417:             assert(_LUSDInStabPool != 0);

1224:             assert(totalStakesSnapshot[_collateral] > 0);

1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1348:         assert(index <= idxLast);

1414:         assert(newBaseRate > 0);

1480:         assert(decayedBaseRate <= DECIMAL_PRECISION);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417


```solidity
File: contracts/BorrowerOperations.sol

128:        assert(MIN_NET_DEBT > 0);

197:        assert(vars.compositeDebt > 0);

301:        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331:        assert(_collWithdrawal <= vars.coll); 

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128

```solidity
File: contracts/LUSDToken.sol

312:        assert(sender != address(0));
313:        assert(recipient != address(0));

321:        assert(account != address(0));

329:        assert(account != address(0));

337:        assert(owner != address(0));
338:        assert(spender != address(0));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312

```solidity
File: contracts/StabilityPool.sol

526:        assert(_debtToOffset <= _totalLUSDDeposits);

551:        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

591:        assert(newP > 0);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526
### [G-03]  `internal` functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

I think it is advisable to replace only `_require` functions. 

_There are 28 instance of this issue:_

```solidity
File: contracts/BorrowerOperations.sol

524: function _requireValidCollateralAddress

533: function _requireSingularCollChange

537: function _requireNonZeroAdjustment

546: function _requireTroveisNotActive

551: function _requireNonZeroDebtChange

555: function _requireNotInRecoveryMode

571: function _requireValidAdjustmentInCurrentMode

624: function _requireNewICRisAboveOldICR

636: function _requireValidLUSDRepayment

640: function _requireCallerIsStabilityPool

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438

```solidity
File: contracts/TroveManager.sol

1538: function _requireMoreThanOneTroveInSystem
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1538

```solidity
File: contracts/ActivePool.sol

320: function _requireCallerIsBorrowerOperationsOrDefaultPool

337: function _requireCallerIsBOorTroveM

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320

```solidity
File: contracts/StabilityPool.sol

848: function _requireCallerIsActivePool

852: function _requireCallerIsTroveManager

856: function _requireNoUnderCollateralizedTroves

869: function _requireUserHasDeposit

873: function _requireNonZeroAmount
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L848

```solidity
File: contracts/LQTY/LQTYStaking.sol

251: function _requireCallerIsTroveManagerOrActivePool

261: function _requireCallerIsBorrowerOperations

265: function _requireUserHasStake

269: function _requireNonZeroAmount

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251

```solidity
File: contracts/LUSDToken.sol

360: function _requireCallerIsBorrowerOperations

365: function _requireCallerIsBOorTroveMorSP

375: function _requireCallerIsStabilityPool

380: function _requireCallerIsTroveMorSP

393: function _requireMintingNotPaused
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```solidity
File: contracts/ReaperStrategyGranarySupplyOnly.sol

206: function _claimRewards() internal {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206

### [G‑04] Structs can be packed into fewer storage slots

_There are 6 instances of this issue:_

***Total slots saved:* 6**

```solidity
File: contracts/CollateralConfig.sol
/// @audit Variable ordering with 3 slots instead of the current 4:
///        bool(1): allowed, uint128(16): decimals, uint256(32): MCR, uint256(32) CCR
27: struct Config {
        bool allowed;
        uint256 decimals;
        uint256 MCR;
        uint256 CCR;
    }
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L27-L32

```Solidity
File: contracts/TroveManager.sol
    
/// @audit Variables ordering with 7 slots instead of 8:
///        uint128(16) collDecimals, bool(1) recoveryModeAtStart, uint256(32) collCCR, uint256(32) collMCR, 
///        uint(32) price, uint(32) LUSDInStabPool, uint(32) liquidatedDebt, uint liquidatedColl;
struct LocalVariables_OuterLiquidationFunction {
	uint256 collDecimals;
	uint256 collCCR;
	uint256 collMCR;
	uint price;
	uint LUSDInStabPool;
	bool recoveryModeAtStart;
	uint liquidatedDebt;
	uint liquidatedColl;
}

/// @audit Variables ordering with 9 slots instead of 10:
/// uint remainingLUSDInStabPool, uint i, uint ICR, uint TCR, address(20) user, bool(1) backToNormalMode, uint88(11) collDecimals
///	uint entireSystemDebt, uint entireSystemColl, uint256 collCCR, uint256 collMCR;
struct LocalVariables_LiquidationSequence {
	uint remainingLUSDInStabPool;
	uint i;
	uint ICR;
	uint TCR;
	address user;
	bool backToNormalMode;
	uint entireSystemDebt;
	uint entireSystemColl;
	uint256 collDecimals;
	uint256 collCCR;
	uint256 collMCR;
}

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L129

```solidity
File: contracts/BorrowerOperations.sol

/// @audit Variables ordering with 15 slots instead of 16:
//         uint256 collCCR, uint256 collMCR, uint128(16) collDecimals, bool(1) isCollIncrease, uint price, uint collChange, uint netDebtChange, 
//         uint debt, uint coll, uint oldICR, uint newICR, uint newTCR, uint LUSDFee, uint newDebt, uint newColl , uint stake;

47: struct LocalVariables_adjustTrove {
        uint256 collCCR;
        uint256 collMCR;
        uint256 collDecimals;
        uint price;
        uint collChange;
        uint netDebtChange;
        bool isCollIncrease;
        uint debt;
        uint coll;
        uint oldICR;
        uint newICR;
        uint newTCR;
        uint LUSDFee;
        uint newDebt;
        uint newColl;
        uint stake;
    }

/// @audit Variables ordering with 15 slots instead of 16:
//         uint256 collCCR, uint256 collMCR, uint128(16) collDecimals, uint128(16) arrayIndex, uint price, 
//         uint LUSDFee, uint netDebt, uint compositeDebt, uint ICR, uint NICR, uint stake
    
66: struct LocalVariables_openTrove {
        uint256 collCCR;
        uint256 collMCR;
        uint256 collDecimals;
        uint price;
        uint LUSDFee;
        uint netDebt;
        uint compositeDebt;
        uint ICR;
        uint NICR;
        uint stake;
        uint arrayIndex;
    }
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47

```solidity
File: contracts/ReaperVaultV2.sol

/// @audit Variables ordering with 6 slots instead of 7:
//         uint128(16) activation, uint128(16) lastReport, uint256 feeBPS, uint256 allocBPS,  
//         uint256 allocated, uint256 gains, uint256 losses, 
  
25: struct StrategyParams {
        uint256 activation; // Activation block.timestamp
        uint256 feeBPS; // Performance fee taken from profit, in BPS
        uint256 allocBPS; // Allocation in BPS of vault's total assets
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
        uint256 lastReport; // block.timestamp of the last time a report occured
    }
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L25-L33


#### **Reference:**
-   [https://ethdebug.github.io/solidity-data-representation](https://ethdebug.github.io/solidity-data-representation)

> Enums are represented by integers; the possibility listed first by 0, the next by 1, and so forth. An enum type just acts like uintN, where N is the smallest legal value large enough to accomodate all the possibilities.

-   [https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding)


### [G-05] Using fixed `bytes` is cheaper than using `string`

_There are 6 instance of this issue:_

```solidity
File: contracts/LQTY/CommunityIssuance.sol

19:    string constant public NAME = "CommunityIssuance";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19

```solidity
File: contracts/LQTY/CommunityIssuance.sol

19:        string constant internal _NAME = "LUSD Stablecoin";
20:        string constant internal _SYMBOL = "LUSD";
21:        string constant internal _VERSION = "1";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34

```solidity
File: /contracts/ActivePool.sol

30:         string constant public NAME = "ActivePool";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30

```
File : contracts/StabilityPool.sol

150: string public constant NAME = "StabilityPool";
```


### [G-06] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

_There are 12 instance of this issue:_

```solidity
File: contracts/LQTY/CommunityIssuance.sol

/// @audit cache variables: lastIssuanceTimestamp, lastDistributionTime
84:     function issueOath() external override returns (uint issuance) {

/// @audit cache variables: lastIssuanceTimestamp, amount.div(distributionPeriod) to use in event
101:    function fund(uint amount) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84

```solidity
File: contracts/ActivePool.sol

/// @audit cache variables: collAmount[_collateral].sub(_amount), defaultPoolAddress, collSurplusPoolAddress
171: function sendCollateral(address _collateral, address _account, uint _amount) external override {

/// @audit cache variables:  LUSDDebt[_collateral].add(_amount)
190: function increaseLUSDDebt(address _collateral, uint _amount) external override {

/// @audit cache variables: LUSDDebt[_collateral].sub(_amount) 
197: function decreaseLUSDDebt(address _collateral, uint _amount) external override {

/// @audit cache variable: collAmount[_collateral].add(_amount)
204: function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L204

```solidity
File: contracts/ReaperVaultERC4626.sol

/// @audit cache variables: tvlCap
79: function maxDeposit(address) external view override returns (uint256 maxAssets) {

/// @audit cache variables: tvlCap
122: function maxMint(address) external view override returns (uint256 maxShares) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79

```solidity
File: contracts/ReaperStrategyGranarySupplyOnly.sol

/// @audit cache variables: want
176:    function _deposit(uint256 toReinvest) internal {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176

```solidity
File: contracts/ReaperVaultV2.sol

/// @audit cache variables: strategies[_strategy].allocBPS
205:    function revokeStrategy(address _strategy) external {

/// @audit cache variable: lockedProfit
418:    function _calculateLockedProfit() internal view returns (uint256) {

581:        emit TvlCapUpdated(tvlCap); // @audit use _newTvlCap
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L205


### [G-07] Splitting require() statements that use && saves gas

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by **3 gas**

_There are 7 instances of this issue:_

```solidity
File: contracts/PairsContract.sol

653:            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653


```solidity
File: contracts/PairsContract.sol

347:    require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
        );
        require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
        );
    }


```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347


```solidity
/// @audit replace with `require`
1279:    assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342:    assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1539:    require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279


### [G-08] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

_There are 3 instance of this issue:_

```solidity
File: contracts/ActivePool.sol

41:   mapping (address => uint256) internal collAmount; // collateral => amount tracker
42:   mapping (address => uint256) internal LUSDDebt; // collateral => corresponding debt tracker


44: mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k
45: mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
46: mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
47: mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41-L42

```solidity
File: contracts/LUSDToken.sol

62:    mapping(address => bool) public troveManagers;
63:    mapping(address => bool) public stabilityPools;
64:    mapping(address => bool) public borrowerOperations;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62

