## Summary

**Gas Optimizations**

|  | Issue | Instances |
| --- | --- | --- |
| [G‑01] | No need for type conversion as token is already an address | 1 |
| [G‑02] | Duplicated require()/revert() checks should be refactored to a modifier or function | 1 |
| [G-03] | Using delete instead of setting state variable/mapping to 0 saves gas | 17 |
| [G-04] | Use double require instead of using && | 4 |
| [G-05] | Emitting storage values instead of the memory one | 1 |
| [G-06] | The result of function calls should be cached rather than re-calling the function | 6 |
| [G-07] | Using storage instead of memory for structs/arrays saves gas | 4 |
| [G-08] | StabilityPool.sol.updateRewardSum(): totalLUSD should be cached in local storage | 2 |
| [G-09] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 1 |
| [G-10] | Not using the named return variables when a function returns, wastes deployment gas | 1 |
| [G-11] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or ifstatement | 2 |
| [G-12] | Multiple accesses of a mapping/array should use a local variable cache | 2 |

Total: 42 instances over 12 issues

### **[G-01] No need for type conversion as `token` is already an address**

```diff
	29:  function asset() external view override returns (address assetTokenAddress) {
  -30:      return address(token);
	+30:      return token;
  31:  }
```

### **[G-02] Duplicated require()/revert() checks should be refactored to a modifier or function**

This saves deployement gas

2 results- 1 file:

```solidity
File: /contracts/TroveManager.sol
	551: require(totals.totalDebtInSequence > 0);
...
	754: require(totals.totalDebtInSequence > 0);
```

- [TroveManager.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L)

### [G-03] Using `delete` instead of setting state variable/mapping to `0` saves gas

17 results - 4 file:

```solidity
File: /contracts/ActivePool.sol:
	253:  vars.profit = 0;

File: /contracts/TroveManager.sol:
	387:  singleLiquidation.debtToOffset = 0;
	388:  singleLiquidation.collToSendToSP = 0;
	468:  debtToOffset ****= 0;
****	469:  ****collToSendToSP = 0;
	505:  singleLiquidation.debtToRedistribute = 0;
	506:  singleLiquidation.collToRedistribute = 0;
	1192: Troves[_borrower][_collateral].stake = 0;
	1285: Troves[_borrower][_collateral].coll = 0;
	1286: Troves[_borrower][_collateral].debt = 0;
	1288: rewardSnapshots[_borrower][_collateral].collAmount = 0;
	1289: rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

File: /contracts/StabilityPool.sol:
	529: lastLUSDLossError_Offset = 0;
	578: currentScale = 0;
	756: compoundedDeposit = 0;

File: /contracts/ReaperVaultV2.sol
	215: strategies[_strategy].allocBPS = 0;
	537: lockedProfit = 0;
```

- [ActivePool.sol#L253](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L253)
- [StabilityPool.sol#L529](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L529)
- [ReaperVaultV2.sol#L215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215)

### Mitigation

```jsx
File: /contracts/ActivePool.sol#L253
	- 253:     vars.profit = 0;
	+            delete vars.profit
```

### [G-04] Use double `require` instead of using `&&`

Using double require instead of operator && can save more gas When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.

4 results - 3 files:

```solidity
File: /contracts/BorrowerOperations.sol
	653:   require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
	654:	              "Max fee percentage must be between 0.5% and 100%");
```

- [BorrowerOperations.sol#L652](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L652)

```solidity
File: /contracts/TroveManager.sol
	1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

- [TroveManager.sol#L1332](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1332)

```solidity
File: /contracts/LUSDToken.sol
	347:     require(
	348:        _recipient != address(0) && 
	349:        _recipient != address(this),
	350:        "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
	351:      );
	352:     require(
	353:        !stabilityPools[_recipient] &&
	354:        !troveManagers[_recipient] &&
	355:        !borrowerOperations[_recipient],
	356:        "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
	357:      );
 
```

- [LUSDToken.sol#L347](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347)

### **[G-05] Emitting storage values instead of the memory one.**

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

1 results - 1 files:

```solidity
File: /contracts/ReaperVaultV2.sol
	581:   emit TvlCapUpdated(tvlCap);
```

- [ReaperVaultV2.sol#L581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581)

### **[G‑06] The result of function calls should be cached rather than re-calling the function**

The instances below point to the second+ call of the function within a single function

6 results - 1 files:

```solidity
File: /contracts/ReaperVaultERC4626.sol
		79: function maxDeposit(address) external view override returns (uint256 maxAssets) {
	  80:      if (emergencyShutdown || balance() >= tvlCap) return 0;  //@audit:cache function here (e.g. uint256 pool = balance(); )
	  81:      if (tvlCap == type(uint256).max) return type(uint256).max;
	  82:    return tvlCap - balance(); //@audit:cache function here
   ...
		122: function maxMint(address) external view override returns (uint256 maxShares) {
		123:   if (emergencyShutdown || balance() >= tvlCap) return 0; //@audit:cache function here
	  124:    if (tvlCap == type(uint256).max) return type(uint256).max;
	  125:    return convertToShares(tvlCap - balance()); //@audit:cache function here
```

- [ReaperVaultERC4626.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)

```solidity
File: /contracts/ReaperVaultERC4626.sol
	66: function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
	68:        if (totalSupply() == 0) return shares; //@audit: Cache totalsupply() function
	69:        return (shares * _freeFunds()) / totalSupply(); //@audit: Cache totalsupply() function
	70:    }
...
	139: function previewMint(uint256 shares) public view override returns (uint256 assets) {
	140:        if (totalSupply() == 0) return shares; //@audit: Cache totalsupply() function
	141:        assets = roundUpDiv(shares * _freeFunds(), totalSupply()); //@audit: Cache totalsupply() function
	142:    }
```

- [ReaperVaultERC4626.sol#L67](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L67)

```solidity
File: /contracts/ReaperVaultERC4626.sol
	51: function convertToShares(uint256 assets) public view override returns (uint256 shares) {
	52:        if (totalSupply() == 0 || _freeFunds() == 0) return assets; //@audit:Cache _freeFunds() and totalsupply() function
	53:        return (assets * totalSupply()) / _freeFunds(); //@audit: Cache _freeFunds() and totalsupply() function
	54:    }
...
	183: function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
	184:        if (totalSupply() == 0 || _freeFunds() == 0) return 0; //@audit: Cache _freeFunds() and totalsupply() function
	185:        shares = roundUpDiv(assets * totalSupply(), _freeFunds()); //@audit: Cache _freeFunds() and totalsupply() function
	186:    }
```

### [G-07] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

5 results - 4 files:

```solidity
File: /contracts/TroveManager.sol	
	313: address[] memory borrowers = new address[](1);

```

- [TroveManager.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L313)

```solidity
File: /contracts/StabilityPool.sol
	687: Snapshots memory snapshots = depositSnapshots[_depositor];
	722: Snapshots memory snapshots = depositSnapshots[_depositor];
```

- [StabilityPool.sol#L687](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687)

```solidity
File: /contracts/ReaperStrategyGranarySupplyOnly.sol
	166: address[2] memory step = _newSteps[i];
```

- [ReaperStrategyGranarySupplyOnly.sol#L166](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166)

### [G-08] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

2 results - 2 files:

### [**1] `StabilityPool.sol.updateRewardSum(): totalLUSD` should be cached in local storage**

```solidity
File: /contracts/StabilityPool.sol
	488: uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD
	489:  if (totalLUSD == 0) { return; } //@audit: use cache value (line 488)
	490:
	491: _triggerLQTYIssuance(communityIssuance);
	492:
	493: (uint collGainPerUnitStaked, ) = _computeRewardsPerUnitStaked(_collateral, _collToAdd, 0, totalLUSD); //@audit: use cache value (line 488)
```

- [StabilityPool.sol#L488](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L488)

### [2]  `LQTYStaking.sol._sendCollGainToUser() : amounts[i]` should be cached in local storage

```diff
File: **/contracts/LQTY/LQTYStaking.sol**
	240: for (uint i = 0; i < numCollaterals; i++) {
+ 241: uint amount = amounts[i];
+	242:            if (amount != 0) {
+	243:                address collateral = assets[i];
+	244:                emit CollateralSent(msg.sender, collateral, amount);
+	245:                IERC20(collateral).safeTransfer(msg.sender, amount);
	246:            }
```

- [LQTYStaking.sol#L238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238)

### **[G-09] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead**

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

**[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)Use a larger size then downcast where needed**

1 results - 1 files:

```solidity
File: /contracts/LUSDToken.sol
	35: uint8 constant internal _DECIMALS = 18;
```

- [LUSDToken.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L35)

### **[G-10] Not using the named return variables when a function returns, wastes deployment gas**

1 results - 1 files:

```solidity
File: **/contracts/LQTY/LQTYStaking.sol**
	213: function getPendingLUSDGain(address _user) external view override returns (uint) {
```

- [LQTYStaking.sol#L213](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L213)

### [G-11]Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`statement

`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`.

2 results - 1 files:

```solidity
File: /contracts/ReaperStrategyGranarySupplyOnly.sol
	81: uint256 toReinvest = wantBalance - _debt; //@audit: Line 80
	100: loss = _amountNeeded - liquidatedAmount; //@audit: Line 99
```

- [ReaperStrategyGranarySupplyOnly.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L80)

### [G-12] Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. **Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

2 results - 2 files:

```solidity

File: /contracts/ReaperVaultV2.sol
	205: function revokeStrategy(address _strategy) external {
      
	210:       if (strategies[_strategy].allocBPS == 0) { //@audit: Initial access
  211:          return;
  212:      }

  214:      totalAllocBPS -= strategies[_strategy].allocBPS; //@audit: 2nd access
  215:      strategies[_strategy].allocBPS = 0;  //@audit: 3rd access
  216:      emit StrategyRevoked(_strategy);
    
```

- [ReaperVaultV2.sol#L205](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L205)

```solidity
File: /contracts/TroveManager.sol 
	1082: function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

	1091: Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward); //@audit: Initial access
	1092: Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward); //@audit: Initial access

	1102: Troves[_borrower][_collateral].debt, //@audit: 2nd access
  1103: Troves[_borrower][_collateral].coll, //@audit: 2nd access
```

- [TroveManager.sol#L1082](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1082)