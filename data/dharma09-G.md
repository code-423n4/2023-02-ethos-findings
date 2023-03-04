### **[G-01] No need for type conversion as `token` is already an address**

```diff
File : /contracts/ReaperVaultERC4626.sol
	29:  function asset() external view override returns (address assetTokenAddress) {
  -30:      return address(token);
	+30:      return token;
  31:  }
```
- [ReaperVaultERC4626.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29)

### **[G-02] Duplicated require()/revert() checks should be refactored to a modifier or function**

This saves deployement gas

```solidity
File: /contracts/TroveManager.sol
	551: require(totals.totalDebtInSequence > 0);
...
	754: require(totals.totalDebtInSequence > 0);
```

- [TroveManager.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L)

### [G-03] Using `delete` instead of setting state variable/mapping to `0` saves gas

5 results - 1 file:

```solidity
File: /contracts/ActivePool.sol:
	253:  vars.profit = 0;

File: /contracts/TroveManager.sol:
	387:  singleLiquidation.debtToOffset = 0;
	388:  singleLiquidation.collToSendToSP = 0;
	468:  debtToOffset = 0;
	469:  collToSendToSP = 0;
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
- [TroveManager.sol#L387](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L387)

```diff
File: /contracts/ActivePool.sol#L253
	- 253:     vars.profit = 0;
	+            delete vars.profit
```

### [G-04] Use double `require` instead of using `&&`

Using double require instead of operator && can save more gas When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.

3 results - 3 files:

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

```solidity
File: /contracts/ReaperVaultV2.sol
	581:   emit TvlCapUpdated(tvlCap);
```

- [ReaperVaultV2.sol#L581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581)

### **[G‑06] The result of function calls should be cached rather than re-calling the function**

The instances below point to the second+ call of the function within a single function

*There is 2 instance of this issue:*

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

### [G-07] StabilityPool.sol**.**updateRewardSum**():** totalLUSD **should be cached in local storage**

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

### [G-08] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: /contracts/TroveManager.sol	
	313: address[] memory borrowers = new address[](1);

```

- [TroveManager.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L313)

```solidity
File : /contracts/ActivePool.sol	
	105: address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollatera
```

- [ActivePool.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L105)

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