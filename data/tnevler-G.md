# Report
## Gas Optimizations ##
### [G-1]: The increment in for loop post condition can be made unchecked
**Context:**

1. ```for(uint256 i = 0; i < _collaterals.length; i++) {``` [L56](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56) 
1. ```for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {``` [L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608) 
1. ```for (vars.i = 0; vars.i < _n; vars.i++) {``` [L690](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690) 
1. ```for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {``` [L808](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808) 
1. ```for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {``` [L882](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882) 
1. ```for(uint256 i = 0; i < numCollaterals; i++) {``` [L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108) 
1. ```for (uint i = 0; i < numCollaterals; i++) {``` [L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351) 
1. ```for (uint i = 0; i < numCollaterals; i++) {``` [L397](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397) 
1. ```for (uint i = 0; i < assets.length; i++) {``` [L640](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640) 
1. ```for (uint i = 0; i < collaterals.length; i++) {``` [L810](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810) 
1. ```for (uint i = 0; i < collaterals.length; i++) {``` [L831](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831) 
1. ```for (uint i = 0; i < numCollaterals; i++) {``` [L859](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859) 
1. ```for (uint i = 0; i < assets.length; i++) {``` [L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206) 
1. ```for (uint i = 0; i < collaterals.length; i++) {``` [L228](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228) 
1. ```for (uint i = 0; i < numCollaterals; i++) {``` [L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240) 

**Description:**

[This](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) can save 30-40 gas per loop iteration.

**Recommendation:**

Example how to fix. Change:
```
for (uint256 i = 0; i < orders.length; ++i) {
   // Do the thing
}
```

To:
```
for (uint256 i = 0; i < orders.length;) {
   // Do the thing
   unchecked { ++i; }
}
```
### [G-2]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));``` [L494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494) (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals)
1. ```cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));``` [L496](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496) (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS)
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
