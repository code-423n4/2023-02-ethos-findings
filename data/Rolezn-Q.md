## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | `decimals()` not part of ERC20 standard | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Event is missing parameters | 2 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops | 2 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Possible rounding issue | 2 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Contracts are not using their OZ Upgradeable counterparts | 6 |
| [LOW&#x2011;6](#LOW&#x2011;6) | `require()` should be used instead of `assert()` | 20 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Inability to remove specific array values | 2 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Upgrade OpenZeppelin Contract Dependency | 3 |

Total: 38 contexts over 8 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 15 |
| [NC&#x2011;2](#NC&#x2011;2) | Avoid Floating Pragmas: The Version Should Be Locked | 3 |
| [NC&#x2011;3](#NC&#x2011;3) | Variable Names That Consist Of All Capital Letters Should Be Reserved For Const/immutable Variables | 2 |
| [NC&#x2011;4](#NC&#x2011;4) | Constants Should Be Defined Rather Than Using Magic Numbers | 4 |
| [NC&#x2011;5](#NC&#x2011;5) | Constants in comparisons should appear on the left side | 12 |
| [NC&#x2011;6](#NC&#x2011;6) | Critical Changes Should Use Two-step Procedure | 15 |
| [NC&#x2011;7](#NC&#x2011;7) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 6 |
| [NC&#x2011;8](#NC&#x2011;8) | Duplicate imports | 2 |
| [NC&#x2011;9](#NC&#x2011;9) | `block.timestamp` is already used when emitting events, no need to input timestamp | 1 |
| [NC&#x2011;10](#NC&#x2011;10) | Function writing that does not comply with the Solidity Style Guide  | 11 |
| [NC&#x2011;11](#NC&#x2011;11) | Large or complicated code bases should implement fuzzing tests | 1 |
| [NC&#x2011;12](#NC&#x2011;12) | Use `delete` to Clear Variables | 17 |
| [NC&#x2011;13](#NC&#x2011;13) | Imports can be grouped together | 4 |
| [NC&#x2011;14](#NC&#x2011;14) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;15](#NC&#x2011;15) | No need to initialize uints to zero | 2 |
| [NC&#x2011;16](#NC&#x2011;16) | Contracts should have full test coverage | 1 |
| [NC&#x2011;17](#NC&#x2011;17) | Lines are too long | 2 |
| [NC&#x2011;18](#NC&#x2011;18) | Missing event for critical parameter change | 2 |
| [NC&#x2011;19](#NC&#x2011;19) | Implementation contract may not be initialized | 4 |
| [NC&#x2011;20](#NC&#x2011;20) | NatSpec comments should be increased in contracts | 1 |
| [NC&#x2011;21](#NC&#x2011;21) | Non-usage of specific imports | 96 |
| [NC&#x2011;22](#NC&#x2011;22) | Use a more recent version of Solidity | 11 |
| [NC&#x2011;23](#NC&#x2011;23) | Omissions in Events | 14 |
| [NC&#x2011;24](#NC&#x2011;24) | `type(uint256).max` Should Be Used Instead Of `uint256(-1)` | 1 |
| [NC&#x2011;25](#NC&#x2011;25) | Use `bytes.concat()` | 1 |
| [NC&#x2011;26](#NC&#x2011;26) | Interchangeable usage of `uint` and `uint256` | 11 |
| [NC&#x2011;27](#NC&#x2011;27) | Use Underscores for Number Literals  | 2 |

Total: 242 contexts over 27 issues

## Low Risk Issues



### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
63: uint256 decimals = IERC20(collateral).decimals()
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63






### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Event is missing parameters

The following functions are missing critical parameters when emitting an event.
When dealing with source address which uses the value of `msg.sender`, the `msg.sender` value must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose. Basically, this event cannot be used.

#### <ins>Proof Of Concept</ins>


```solidity

function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {
        ...
        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
        ...
    }

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L208


```solidity

function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {
        ...
        emit StabilityPoolCollateralBalanceUpdated(_collateral, newCollAmount);
        ...

        IERC20(_collateral).safeTransfer(msg.sender, _amount);
    }

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L787


#### <ins>Recommended Mitigation Steps</ins>
Add `msg.sender` parameter in event-emit



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops

The functions `_updateDepositAndSnapshots` and `_updateUserSnapshots` contains loops over arrays that the user cannot influence. These functions are called in `StabilityPool.provideToSP`, `StabilityPool.withdrawFromSP`, `LQTYStaking.stake` and `LQTYStaking.unstake`.
In any call for the following functions, users spend gas to iterate over the array. There are loops in the following functions that iterate all array values of `collaterals`. Which could risk gas exhaustion on large array calls.

This risk is small, but could be lessened somewhat by allowing separate calls for small loops.

#### <ins>Proof Of Concept</ins>


```solidity
803: function _updateDepositAndSnapshots(address _depositor, uint _newValue) internal {
        ...

        address[] memory collaterals = collateralConfig.getAllowedCollaterals();
        uint[] memory amounts = new uint[](collaterals.length);

        if (_newValue == 0) {
            for (uint i = 0; i < collaterals.length; i++) {
                delete depositSnapshots[_depositor].S[collaterals[i]];
            }
            delete depositSnapshots[_depositor];
            emit DepositSnapshotUpdated(_depositor, 0, collaterals, amounts, 0);
            return;
        }
        ...
        
        for (uint i = 0; i < collaterals.length; i++) {
            address collateral = collaterals[i];
            uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
            depositSnapshots[_depositor].S[collateral] = currentS;
            amounts[i] = currentS;
        }
        emit DepositSnapshotUpdated(_depositor, currentP, collaterals, amounts, currentG);
    }

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L803

```solidity
225: function _updateUserSnapshots(address _user) internal {
        address[] memory collaterals = collateralConfig.getAllowedCollaterals();
        uint[] memory amounts = new uint[](collaterals.length);
        for (uint i = 0; i < collaterals.length; i++) {
            address collateral = collaterals[i];
            snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];
            amounts[i] = F_Collateral[collateral];
        }
        ...
    }

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L225







### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Possible rounding issue

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

#### <ins>Proof Of Concept</ins>


```solidity
421: return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421

```solidity
440: uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L440













### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
14: import "@openzeppelin/contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L9-L14



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts








### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> `require()` Should Be Used Instead Of `assert()`

Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction's available gas rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its <a href="https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require">documentation</a> states that "The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix".

#### <ins>Proof Of Concept</ins>


```solidity
128: assert(MIN_NET_DEBT > 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128

```solidity
197: assert(vars.compositeDebt > 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197

```solidity
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331: assert(_collWithdrawal <= vars.coll);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331



```solidity
312: assert(sender != address(0));
313: assert(recipient != address(0));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313

```solidity
321: assert(account != address(0));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321

```solidity
329: assert(account != address(0));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329

```solidity
337: assert(owner != address(0));
338: assert(spender != address(0));

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337-L338

```solidity
526: assert(_debtToOffset <= _totalLUSDDeposits);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526

```solidity
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591



```solidity
417: assert(_LUSDInStabPool != 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417

```solidity
1224: assert(totalStakesSnapshot[_collateral] > 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224

```solidity
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279

```solidity
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348



```solidity
1414: assert(newBaseRate > 0);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414

```solidity
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489





### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Inability to remove specific array values

New items are pushed into the following arrays:

`collaterals`, `withdrawalQueue`

There's no functionality to remove specific array values. It is only possible by completely deleting the entire array using `delete` calls.

If the array grows too large, it would only be possible to remove array values by completely recreating the `withdrawalQueue` array via `setWithdrawalQueue()`

#### <ins>Proof Of Concept</ins>

```solidity
59: collaterals.push(collateral);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L59


```solidity
169: withdrawalQueue.push(_strategy);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L169




#### <ins>Recommended Mitigation Steps</ins>
Add a functionality to delete array values or add a maximum size limit for arrays.



### <a href="#Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Upgrade OpenZeppelin Contract Dependency

An outdated OZ version is used (which has known vulnerabilities, see: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).

#### <ins>Proof Of Concept</ins>

Require dependencies to be at least version of 4.8.1 to include critical vulnerability patches.

```solidity
    "@openzeppelin/contracts": "^3.3.0"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Core/package.json#L34

```solidity
    "@openzeppelin/contracts": "^4.7.3"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Vault/package.json#L41

```solidity
    "@openzeppelin/contracts-upgradeable": "^4.7.3"
```

https://github.com/code-423n4/2023-02-ethos/blob/main/../2023-02-ethos/blob/main/Ethos-Vault/package.json#L42



#### <ins>Recommended Mitigation Steps</ins>

Update OpenZeppelin Contracts Usage in package.json and require at least version of 4.8.1 via `>=4.8.1`







## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

#### <ins>Proof Of Concept</ins>


```solidity
71: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71

```solidity
125: function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125

```solidity
132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132

```solidity
138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138

```solidity
144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144

```solidity
110: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110

```solidity
261: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261

```solidity
232: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232

```solidity
1562: function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562

```solidity
61: function setAddresses
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61

```solidity
66: function setAddresses
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66

```solidity
160: function setHarvestSteps(address[2][] calldata _newSteps) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160

```solidity
258: function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258

```solidity
603: function setEmergencyShutdown(bool _active) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L603

```solidity
617: function setLockedProfitDegradation(uint256 degradation) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617





### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Variable names that consist of all capital letters should be reserved for `constant`/`immutable` variables

Variable names with all capital letters should be restricted to be used only with `constant` or `immutable` variables.
If the variable needs to be different based on which class it comes from, a `view`/`pure` function should be used instead (e.g. like <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59">this</a>).

#### <ins>Proof Of Concept</ins>

```solidity
195: uint public P = DECIMAL_PRECISION;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L195

```solidity
29: uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L29





### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Variable Names That Consist Of All Capital Letters Should Be Reserved For Const/immutable Variables

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.

#### <ins>Proof Of Concept</ins>

```solidity
591: assert(newP > 0);
        P = newP;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591

```solidity
864: uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L864

```solidity
819: uint TCR = LiquityMath._computeCR(vars.entireSystemColl, vars.entireSystemDebt, _price, vars.collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L819

```solidity
1049: uint NICR = LiquityMath._computeNominalCR(currentCollateral, currentLUSDDebt, collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1049

```solidity
1062: uint ICR = LiquityMath._computeCR(currentCollateral, currentLUSDDebt, _price, collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1062

```solidity
1384: uint TCR = LiquityMath._computeCR(_entireSystemColl, _entireSystemDebt, _price, _collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1384






### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Constants Should Be Defined Rather Than Using Magic Numbers

#### <ins>Proof Of Concept</ins>

For example, define different maxBPS values for `_bps` and `_driftBPs`.

```solidity
127: require(_bps <= 10_000, "Invalid BPS value");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127

```solidity
133: require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L133

```solidity
145: require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L145

```solidity
768: if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L768




### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Constants in comparisons should appear on the left side

Doing so will prevent <a href="https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html">typo bugs</a>

#### <ins>Proof Of Concept</ins>

```solidity
428: if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L428

```solidity
469: if (totalLUSD == 0 || _debtToOffset == 0) { return; }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L469

```solidity
489: if (totalLUSD == 0) { return; }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L489

```solidity
575: if (newProductFactor == 0) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L575

```solidity
685: if (initialDeposit == 0) {return 0;}
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L685

```solidity
720: if (initialDeposit == 0) { return 0; }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L720

```solidity
751: if (scaleDiff == 0) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L751

```solidity
1127: if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1127

```solidity
1142: if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1142

```solidity
121: if (amount == 0) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L121

```solidity
210: if (strategies[_strategy].allocBPS == 0) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L210

```solidity
380: if (strategyBal == 0) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L380




### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
71: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71

```solidity
125: function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125

```solidity
132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132

```solidity
138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138

```solidity
144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144

```solidity
110: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L110

```solidity
261: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261

```solidity
232: function setAddresses(
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L232

```solidity
1562: function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562

```solidity
61: function setAddresses
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61

```solidity
66: function setAddresses
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66

```solidity
160: function setHarvestSteps(address[2][] calldata _newSteps) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160

```solidity
258: function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258

```solidity
603: function setEmergencyShutdown(bool _active) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L603

```solidity
617: function setLockedProfitDegradation(uint256 degradation) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



#### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
551: require(totals.totalDebtInSequence > 0);
754: require(totals.totalDebtInSequence > 0);
```


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754


```solidity
155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
181: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
180: require(strategies[_strategy].activation != 0, "Invalid strategy address");
193: require(strategies[_strategy].activation != 0, "Invalid strategy address");

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193








### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Duplicate imports

There are several occasions where there several imports of the same file

#### <ins>Proof Of Concept</ins>


```solidity
5: import './Interfaces/IBorrowerOperations.sol';
8: import './Interfaces/IBorrowerOperations.sol';

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8








### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> block.timestamp is already used when emitting events, no need to input timestamp

#### <ins>Proof Of Concept</ins>

```solidity
1505: emit LastFeeOpTimeUpdated(block.timestamp);

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1505









### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private
- within a grouping, place the view and pure functions last

#### <ins>Proof Of Concept</ins>

All in-scope contracts

See `BorrowerOperations.sol` for example.



### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Large or complicated code bases should implement fuzzing tests

Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement <a href="https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05">fuzzing tests</a>. Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

#### <ins>Proof Of Concept</ins>

Apply this to:
- `BorrowerOperations.sol`
- `TroveManager.sol`
- `StabilityPool.sol`
- `ReaperVaultV2.sol`




### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
253: vars.profit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L253

```solidity
529: lastLUSDLossError_Offset = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L529

```solidity
578: currentScale = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L578

```solidity
756: compoundedDeposit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L756

```solidity
387: singleLiquidation.debtToOffset = 0;
388: singleLiquidation.collToSendToSP = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L387-L388

```solidity
468: debtToOffset = 0;
469: collToSendToSP = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L468-L469

```solidity
505: singleLiquidation.debtToRedistribute = 0;
506: singleLiquidation.collToRedistribute = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L505-L506

```solidity
1192: Troves[_borrower][_collateral].stake = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1192

```solidity
1285: Troves[_borrower][_collateral].coll = 0;
1286: Troves[_borrower][_collateral].debt = 0;
1288: rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289: rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1285

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1286

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1288

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1289



```solidity
215: strategies[_strategy].allocBPS = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215

```solidity
537: lockedProfit = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L537





### <a href="#Summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Imports can be grouped together

Consider importing OZ first, then all dependencies, then all interfaces and lastly all utils.

#### <ins>Proof Of Concept</ins>


```solidity
5: import "../Dependencies/IERC20.sol";
6: import "../Interfaces/ICommunityIssuance.sol";
7: import "../Dependencies/BaseMath.sol";
8: import "../Dependencies/LiquityMath.sol";
9: import "../Dependencies/Ownable.sol";
10: import "../Dependencies/CheckContract.sol";
11: import "../Dependencies/SafeMath.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11

```solidity
5: import "../Dependencies/BaseMath.sol";
6: import "../Dependencies/SafeMath.sol";
7: import "../Dependencies/Ownable.sol";
8: import "../Dependencies/CheckContract.sol";
9: import "../Dependencies/console.sol";
10: import "../Dependencies/IERC20.sol";
11: import "../Interfaces/ICollateralConfig.sol";
12: import "../Interfaces/ILQTYStaking.sol";
13: import "../Interfaces/ITroveManager.sol";
14: import "../Dependencies/LiquityMath.sol";
15: import "../Interfaces/ILUSDToken.sol";
16: import "../Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16

```solidity
5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
9: import "./interfaces/ILendingPoolAddressesProvider.sol";
10: import "./interfaces/IRewardsController.sol";
11: import "./libraries/ReaperMathUtils.sol";
12: import "./mixins/VeloSolidMixin.sol";
13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L14


```solidity
5: import "./interfaces/IERC4626Events.sol";
6: import "./interfaces/IStrategy.sol";
7: import "./libraries/ReaperMathUtils.sol";
8: import "./mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
14: import "@openzeppelin/contracts/utils/math/Math.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L14







### <a href="#Summary">[NC&#x2011;14]</a><a name="NC&#x2011;14"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


All in-scope contracts

See `BorrowOperations.sol` for example

#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```



### <a href="#Summary">[NC&#x2011;15]</a><a name="NC&#x2011;15"> No need to initialize uints to zero

There is no need to initialize `uint` variables to zero as their default value is `0`

#### <ins>Proof Of Concept</ins>

```solidity
369: uint256 totalLoss = 0;
371: uint256 vaultBalance = 0;

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L369

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L371








### <a href="#Summary">[NC&#x2011;16]</a><a name="NC&#x2011;16"> Contracts should have full test coverage

The test coverage rate of the project is 97%. Testing all functions is best practice in terms of security criteria.
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

It is stated in the docs:
```
What is the overall line coverage percentage provided by your tests?:  93
```
Due to its capacity, test coverage is expected to be 100%.



### <a href="#Summary">[NC&#x2011;17]</a><a name="NC&#x2011;17"> Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### <ins>Proof Of Concept</ins>

```solidity
1044: // Return the nominal collateral ratio (ICR) of a given Trove, without the price. Takes a trove's pending coll and debt rewards from redistributions into account.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1044

```solidity
1053: // Return the current collateral ratio (ICR) of a given Trove. Takes a trove's pending coll and debt rewards from redistributions into account.
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1053








### <a href="#Summary">[NC&#x2011;18]</a><a name="NC&#x2011;18"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
1562: function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562

```solidity
160: function setHarvestSteps(address[2][] calldata _newSteps) external {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160





### <a href="#Summary">[NC&#x2011;19]</a><a name="NC&#x2011;19"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
225: constructor() public
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225

```solidity
57: constructor() public
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57

```solidity
16: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16

```solidity
111: constructor(
        address _token,
        string memory _name,
        string memory _symbol,
        uint256 _tvlCap,
        address _treasury,
        address[] memory _strategists,
        address[] memory _multisigRoles
    ) ERC20(string(_name), string(_symbol))
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111





### <a href="#Summary">[NC&#x2011;20]</a><a name="NC&#x2011;20"> NatSpec comments should be increased in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

#### <ins>Proof Of Concept</ins>


All in-scope contracts

See `BorrowerOperations.sol` for example.

#### <ins>Recommended Mitigation Steps</ins>

NatSpec comments should be increased in contracts



### <a href="#Summary">[NC&#x2011;21]</a><a name="NC&#x2011;21"> Non-usage of specific imports

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace.
Instead, the Solidity docs recommend specifying imported symbols explicitly.
https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

A good example:
```solidity
import {OwnableUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/OwnableUpgradeable.sol";
import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
import {SafeCastLib} from "solmate/utils/SafeCastLib.sol";
import {ERC20} from "solmate/tokens/ERC20.sol";
import {IProducer} from "src/interfaces/IProducer.sol";
import {GlobalState, UserState} from "src/Common.sol";
```

#### <ins>Proof Of Concept</ins>


```solidity
5: import './Interfaces/IActivePool.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IDefaultPool.sol';
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILQTYStaking.sol";
10: import "./Interfaces/IStabilityPool.sol";
11: import "./Interfaces/ITroveManager.sol";
12: import "./Dependencies/SafeMath.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";
17: import "./Dependencies/IERC4626.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L5-L17

```solidity
5: import "./Interfaces/IBorrowerOperations.sol";
6: import "./Interfaces/ICollateralConfig.sol";
7: import "./Interfaces/ITroveManager.sol";
8: import "./Interfaces/ILUSDToken.sol";
9: import "./Interfaces/ICollSurplusPool.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Dependencies/LiquityBase.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5-L16

```solidity
5: import "./Dependencies/CheckContract.sol";
6: import "./Dependencies/Ownable.sol";
7: import "./Dependencies/SafeERC20.sol";
8: import "./Interfaces/ICollateralConfig.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8

```solidity
5: import "./Interfaces/ILUSDToken.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Dependencies/SafeMath.sol";
8: import "./Dependencies/CheckContract.sol";
9: import "./Dependencies/console.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L5-L9

```solidity
5: import './Interfaces/IBorrowerOperations.sol';
8: import './Interfaces/IBorrowerOperations.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IStabilityPool.sol';
5: import './Interfaces/IBorrowerOperations.sol';
8: import './Interfaces/IBorrowerOperations.sol';
9: import './Interfaces/ITroveManager.sol';
10: import './Interfaces/ILUSDToken.sol';
11: import './Interfaces/ISortedTroves.sol';
12: import "./Interfaces/ICommunityIssuance.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/SafeMath.sol";
15: import "./Dependencies/LiquitySafeMath128.sol";
16: import "./Dependencies/Ownable.sol";
17: import "./Dependencies/CheckContract.sol";
18: import "./Dependencies/console.sol";
19: import "./Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L6

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L7

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L9

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L10

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L11

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L12

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L13

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L14

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L15

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L16

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L17

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L19



```solidity
5: import "./Interfaces/ICollateralConfig.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Interfaces/IStabilityPool.sol";
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILUSDToken.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Interfaces/IRedemptionHelper.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/Ownable.sol";
15: import "./Dependencies/CheckContract.sol";
16: import "./Dependencies/IERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L5-L16

```solidity
5: import "../Dependencies/IERC20.sol";
6: import "../Interfaces/ICommunityIssuance.sol";
7: import "../Dependencies/BaseMath.sol";
8: import "../Dependencies/LiquityMath.sol";
9: import "../Dependencies/Ownable.sol";
10: import "../Dependencies/CheckContract.sol";
11: import "../Dependencies/SafeMath.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11

```solidity
5: import "../Dependencies/BaseMath.sol";
6: import "../Dependencies/SafeMath.sol";
7: import "../Dependencies/Ownable.sol";
8: import "../Dependencies/CheckContract.sol";
9: import "../Dependencies/console.sol";
10: import "../Dependencies/IERC20.sol";
11: import "../Interfaces/ICollateralConfig.sol";
12: import "../Interfaces/ILQTYStaking.sol";
13: import "../Interfaces/ITroveManager.sol";
14: import "../Dependencies/LiquityMath.sol";
15: import "../Interfaces/ILUSDToken.sol";
16: import "../Dependencies/SafeERC20.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16

```solidity
5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
9: import "./interfaces/ILendingPoolAddressesProvider.sol";
10: import "./interfaces/IRewardsController.sol";
11: import "./libraries/ReaperMathUtils.sol";
12: import "./mixins/VeloSolidMixin.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L12

```solidity
5: import "./ReaperVaultV2.sol";
6: import "./interfaces/IERC4626Functions.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6

```solidity
5: import "./interfaces/IERC4626Events.sol";
6: import "./interfaces/IStrategy.sol";
7: import "./libraries/ReaperMathUtils.sol";
8: import "./mixins/ReaperAccessControl.sol";

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L8




#### <ins>Recommended Mitigation Steps</ins>

Use specific imports syntax per solidity docs recommendation.



### <a href="#Summary">[NC&#x2011;22]</a><a name="NC&#x2011;22"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/">0.8.4</a>:
bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<a href="https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/">0.8.19</a>:
SMTChecker: New trusted mode that assumes that any compile-time available code is the actual used code, even in external calls. 
Bug Fixes: 
- Assembler: Avoid duplicating subassembly bytecode where possible.
- Code Generator: Avoid including references to the deployed label of referenced functions if they are called right away.
- ContractLevelChecker: Properly distinguish the case of missing base constructor arguments from having an unimplemented base function.
- SMTChecker: Fix internal error caused by unhandled z3 expressions that come from the solver when bitwise operators are used.
- SMTChecker: Fix internal error when using the custom NatSpec annotation to abstract free functions.
- TypeChecker: Also allow external library functions in using for.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3

```solidity
pragma solidity 0.6.11;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3

```solidity
pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.



### <a href="#Summary">[NC&#x2011;23]</a><a name="NC&#x2011;23"> Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters
The following events should also add the previous original value in addition to the new value.

#### <ins>Proof Of Concept</ins>


```solidity
65: event YieldingPercentageDriftUpdated(uint256 _driftBps);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L65

```solidity
234: event StabilityPoolLUSDBalanceUpdated(uint _newBalance);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L234

```solidity
246: event P_Updated(uint _P);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L246

```solidity
249: event EpochUpdated(uint128 _currentEpoch);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L249

```solidity
250: event ScaleUpdated(uint128 _currentScale);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L250

```solidity
203: event BaseRateUpdated(uint _baseRate);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L203

```solidity
204: event LastFeeOpTimeUpdated(uint _lastFeeOpTime);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L204

```solidity
53: event TotalOATHIssuedUpdated(uint _totalOATHIssued);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53

```solidity
59: event F_LUSDUpdated(uint _F_LUSD);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59

```solidity
60: event TotalLQTYStakedUpdated(uint _totalLQTYStaked);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60

```solidity
84: event UpdateWithdrawalQueue(address[] withdrawalQueue);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L84

```solidity
85: event WithdrawMaxLossUpdated(uint256 withdrawMaxLoss);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L85

```solidity
88: event TvlCapUpdated(uint256 newTvlCap);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L88

```solidity
89: event LockedProfitDegradationUpdated(uint256 degradation);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L89




#### <ins>Recommended Mitigation Steps</ins>
The events should include the new value and old value where possible.













### <a href="#Summary">[NC&#x2011;24]</a><a name="NC&#x2011;24"> `type(uint256).max` Should Be Used Instead Of `uint256(-1)`

#### <ins>Proof Of Concept</ins>


```solidity
258: vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258








### <a href="#Summary">[NC&#x2011;25]</a><a name="NC&#x2011;25"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
283: bytes32 digest = keccak256(abi.encodePacked('/x19/x01', 
                         domainSeparator(), keccak256(abi.encode(
                         _PERMIT_TYPEHASH, owner, spender, amount, 
                         _nonces[owner]++, deadline))))

```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283




#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 








### <a href="#Summary">[NC&#x2011;26]</a><a name="NC&#x2011;26"> Interchangeable usage of `uint` and `uint256`

Some developers prefer to use `uint256` because it is consistent with other `uint` data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example, if doing bytes4(keccak('transfer(address, uint)’)), you'll get a different method sig ID than bytes4(keccak('transfer(address, uint256)’)) and smart contracts will only understand the latter when comparing method sig IDs

#### <ins>Proof Of Concept</ins>


```solidity
_getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432

```solidity
_requireNotInRecoveryMode(
        address _collateral,
        uint _price,
        uint256 _CCR,
        uint256 _collateralDecimals
    ) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L555

```solidity
_getNewNominalICRFromTroveChange
    (
        uint _coll,
        uint _debt,
        uint _collChange,
        bool _isCollIncrease,
        uint _debtChange,
        bool _isDebtIncrease,
        uint256 _collateralDecimals
    )
        pure
        internal
        returns (uint)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661

```solidity
_getNewICRFromTroveChange
    (
        uint _coll,
        uint _debt,
        uint _collChange,
        bool _isCollIncrease,
        uint _debtChange,
        bool _isDebtIncrease,
        uint _price,
        uint256 _collateralDecimals
    )
        pure
        internal
        returns (uint)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L682

```solidity
_getNewTCRFromTroveChange
    (
        address _collateral,
        uint _collChange,
        bool _isCollIncrease,
        uint _debtChange,
        bool _isDebtIncrease,
        uint _price,
        uint256 _collateralDecimals
    )
        internal
        view
        returns (uint)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L724

```solidity
_liquidateRecoveryMode(
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        address _borrower,
        uint _ICR,
        uint _LUSDInStabPool,
        uint _TCR,
        uint _price,
        uint256 _MCR
    )
        internal
        returns (LiquidationValues memory singleLiquidation)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L357

```solidity
_getCappedOffsetVals
    (
        uint _entireTroveDebt,
        uint _entireTroveColl,
        uint _price,
        uint256 _MCR,
        uint _collDecimals
    )
        internal
        pure
        returns (LiquidationValues memory singleLiquidation)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478

```solidity
_getTotalsFromLiquidateTrovesSequence_NormalMode
    (
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        uint _price,
        uint256 _MCR,
        uint _LUSDInStabPool,
        uint _n
    )
        internal
        returns(LiquidationTotals memory totals)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L671

```solidity
_getTotalsFromBatchLiquidate_NormalMode
    (
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        uint _price,
        uint256 _MCR,
        uint _LUSDInStabPool,
        address[] memory _troveArray
    )
        internal
        returns(LiquidationTotals memory totals)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L864

```solidity
_redistributeDebtAndColl(
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        uint _debt,
        uint _coll,
        uint256 _collDecimals
    ) internal {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1230

```solidity
_checkPotentialRecoveryMode(
        uint _entireSystemColl,
        uint _entireSystemDebt,
        uint _price,
        uint256 _collDecimals,
        uint256 _CCR
    )
        internal
        pure
    returns (bool)
    {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1373

```solidity
updateBaseRateFromRedemption(
        uint _collateralDrawn,
        uint _price,
        uint256 _collDecimals,
        uint _totalLUSDSupply
    ) external override returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1397



### <a href="#Summary">[NC&#x2011;27]</a><a name="NC&#x2011;27"> Use Underscores for Number Literals

#### <ins>Proof Of Concept</ins>


```solidity
41: uint256 public constant PERCENT_DIVISOR = 10000;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41






