# Report
## Gas Optimizations ##
### [G-1]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```return -int256(stratCurrentAllocation - stratMaxAllocation);``` [L235](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L235) (stratCurrentAllocation - stratMaxAllocation)
1. ```token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);``` [L526](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L526) (vars.credit - vars.freeWantInStrat)
1. ```token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);``` [L528](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L528) (vars.freeWantInStrat - vars.credit)
1. ```lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;``` [L535](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L535) 
1. ```return tvlCap - balance();``` [L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82) (tvlCap - balance())
1. ```return convertToShares(tvlCap - balance());``` [L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125) (tvlCap - balance())
1. ```roi = -int256(debt - amountFreed);``` [L121](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121) 
1. ```roi = int256(amountFreed - debt);``` [L123](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L123) 
1. ```uint256 toReinvest = wantBalance - _debt;``` [L81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81) 
1. ```_withdraw(_amountNeeded - wantBal);``` [L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93) 
1. ```loss = _amountNeeded - liquidatedAmount;``` [L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100) 
1. ```uint256 profit = totalAssets - allocated;``` [L132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132) 
1. ```roi = -int256(allocated - totalAssets);``` [L136](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136)

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.
