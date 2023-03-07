# Gas Optimizations Report for Ethos Reserve contest
## Overview
During the audit, 2 gas issues were found.  

№ | Title | Instance Count 
--- | --- | --- 
G-1 | Use unchecked blocks for subtractions where underflow is impossible | 14 
G-2 | Using ```storage``` pointer to ```struct```/```array```/```mapping``` is cheaper than using ```memory``` | 2 

## Gas Optimizations Findings(2)
### G-1. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas (~35 per instance) can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- [```return -int256(stratCurrentAllocation - stratMaxAllocation);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L235) 
- [```uint256 remaining = value - vaultBalance;```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L384) 
- [```token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L526) 
- [```token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L528) 
- [```lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L535) 
- [```return tvlCap - balance();```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82) 
- [```return convertToShares(tvlCap - balance());```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125) 
- [```roi = -int256(debt - amountFreed);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121) 
- [```roi = int256(amountFreed - debt);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L123) 
- [```uint256 toReinvest = wantBalance - _debt;```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81) 
- [```_withdraw(_amountNeeded - wantBal);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93) 
- [```loss = _amountNeeded - liquidatedAmount;```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100) 
- [```uint256 profit = totalAssets - allocated;```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132) 
- [```roi = -int256(allocated - totalAssets);```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136) 

#
### G-2. Using ```storage``` pointer to ```struct```/```array```/```mapping``` is cheaper than using ```memory```
##### Description
When you copy a storage ```struct```/```array```/```mapping``` to a memory variable, you are copying each member by reading it from the storage, which is expensive, but when you use the ```storage``` keyword, you are just storing a pointer to the storage, which is much cheaper.
##### Instances
- [```Snapshots memory snapshots = depositSnapshots[_depositor];```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687) 
- [```Snapshots memory snapshots = depositSnapshots[_depositor];```](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L722) 

##### Recommendation
For example, change: 
```
Snapshots memory snapshots = depositSnapshots[_depositor];
```  
to:  
```
Snapshots storage snapshots = depositSnapshots[_depositor];
```