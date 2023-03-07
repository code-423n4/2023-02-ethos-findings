| S No. | Issue | Instances |
|-----|-----|-----|
| [01] | Best practice is to prevent signature malleability | 1
| [02] | Large multiples of ten should use scientific notation | 6
| [03] | Require() should be used instead of assert() | 20
| [04] | Upgradeable contract is missing a \_\_gap[50] storage variable to allow for new storage variables in later versions | 3
| [05] | Adding a return statement when the function defines a named return variable, is redundant | 18
| [06] | Use a more recent version of solidity | 12 
| [07] | Non-library or interface files should use fixed compiler versions, not floating ones | 4
| [08] | Use named imports instead of plain 'import file.sol' | 110

-------------

## 01 Best practice is to prevent signature malleability

Use OpenZeppelin’s `ECDSA` contract rather than calling `ecrecover()` directly

_There is 1 instance of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
File: Ethos-Core/contracts/LUSDToken.sol

287: address recoveredAddress = ecrecover(digest, v, r, s);
```

-----

## 02 Large multiples of ten should use scientific notation

Use (e.g. 1e12) rather than decimal literals (e.g. 1000000000000), for better code readability

_There are 6 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
File: Ethos-Core/contracts/ActivePool.sol

127: require(_bps <= 10_000, "Invalid BPS value");
145: require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
258: vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
262: vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
291: vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);
296: vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);
```

----------

## 03 Require() should be used instead of assert()

Assert should not be used except for tests, `require` should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states “The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there’s a mistake”.

_There are 20 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
File: Ethos-Core/contracts/BorrowerOperations.sol

128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331: assert(_collWithdrawal <= vars.coll);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
File: Ethos-Core/contracts/LUSDToken.sol

312: assert(sender != address(0));
313: assert(recipient != address(0));
321: assert(account != address(0));
329: assert(account != address(0));
337: assert(owner != address(0));
338: assert(spender != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

417: assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);
```

------------

## 04 Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

_There are 3 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

21: using SafeERC20Upgradeable for IERC20Upgradeable;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

17: UUPSUpgradeable,
18: AccessControlEnumerableUpgradeable
```

--------

## 05 Adding a return statement when the function defines a named return variable, is redundant

_There are 18 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
File: Ethos-Core/contracts/StabilityPool.sol

504: function _computeRewardsPerUnitStaked(
626: function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
File: Ethos-Core/contracts/TroveManager.sol

321: function _liquidateNormalMode(
357: function _liquidateRecoveryMode(
898: function _addLiquidationValuesToTotals(LiquidationTotals memory oldTotals, LiquidationValues memory singleLiquidation)
1316: function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
1321: function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

104: function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

29: function asset() external view override returns (address assetTokenAddress) {
37: function totalAssets() external view override returns (uint256 totalManagedAssets) {
51: function convertToShares(uint256 assets) public view override returns (uint256 shares) {
66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
79: function maxDeposit(address) external view override returns (uint256 maxAssets) {
96: function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
122: function maxMint(address) external view override returns (uint256 maxShares) {
165: function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {
220: function maxRedeem(address owner) external view override returns (uint256 maxShares) {
240: function previewRedeem(uint256 shares) external view override returns (uint256 assets) {
```

--------

## 06 Use a more recent version of solidity

_This issue exists in all the In-scope contracts. There are 12 instances of this issue_

[CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3) , [BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3) , [TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)] , [ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3) , [StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3) , [CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3) , [LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3) , [LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3) , [ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3) , [ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3) , [ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3) , [ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)

-----------

## 07 Non-library or interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

_There are 4 instances of this issue_

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

3: pragma solidity ^0.8.0;
```

-----

## 08 Use named imports instead of plain 'import file.sol'

For instance, you use regular imports such as:

[CollateralConfig.sol#L5](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5)

```
import "./Dependencies/CheckContract.sol";
```

Instead of this, use named imports such as:

```
import {CheckContract.sol} from "./Dependencies/CheckContract.sol";
```

_This issue exists in all the In-scope contracts. There are 110 instances of this issue_

[CollateralConfig.sol#L5-L8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8) , [BorrowerOperations.sol#L5-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L5-L16) , [TroveManager.sol#L5-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L5-L16) , [ActivePool.sol#L5-L17](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L5-L17) , [StabilityPool.sol#L5-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5-L19) , [CommunityIssuance.sol#L5-L11](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11) , [LQTYStaking.sol#L5-L16](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16) , [LUSDToken.sol#L5-L9](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L5-L9) , [ReaperVaultV2.sol#L5-L14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L14) , [ReaperVaultERC4626.sol#L5-L6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6) , [ReaperBaseStrategyv4.sol#L5-L12](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L5-L12) , [ReaperStrategyGranarySupplyOnly.sol#L5-L14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L14)

--------------

