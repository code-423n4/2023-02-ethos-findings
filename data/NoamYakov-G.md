
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate | 1 |  87,330 |
| [G&#x2011;02] | State variables can be packed into fewer storage slots | 2 |  21,200 |
| [G&#x2011;03] | Avoid contract existence checks by using low level calls | 101 |  18,500 |
| [G&#x2011;04] | Functions guaranteed to revert when called by normal users can be marked `payable` | 56 |  1,260 |

Total: 160 instances over 4 issues with **128,290 gas** saved.

Gas totals are only calculated for `public`/`external` functions, use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions.

## Gas Optimizations

### [G&#x2011;01]  Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There is 1 instance of this issue:*

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L62-L64)
```solidity
/// @audit can be a mapping of an address to a struct of 3 bools
62      mapping (address => bool) public troveManagers;
63      mapping (address => bool) public stabilityPools;
64      mapping (address => bool) public borrowerOperations;
```

- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`LUSDToken.transfer()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L221) (see [`LUSDToken._requireValidRecipient()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L353-L355))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`LUSDToken.transferFrom()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L236) (see [`LUSDToken._requireValidRecipient()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L353-L355))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`LUSDToken.burn()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L192) (see [`LUSDToken._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L368-L370))
- Saves 1 Gcoldsload + 1 Gkeccak256 (**2130 gas**) on [`LUSDToken.returnFromPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L202) (see [`LUSDToken._requireCallerIsTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L385))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`LQTYStaking.stake()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135)
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`LQTYStaking.unstake()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171)
- Saves 4 Gcoldsload + 4 Gkeccak256 (**8520 gas**) on [`BorrowerOperations.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L402-L403) (see [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L493) + [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L493) + [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L493) + [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L493) + [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L493) + [`BorrowerOperations._repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L519))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`StabilityPool.offset()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L478) (see [`StabilityPool._moveOffsetCollAndDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L605))
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`TroveManager.redeemCloseTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L946)
- Saves 2 Gcoldsload + 2 Gkeccak256 (**4260 gas**) on [`TroveManager.burnLUSDAndEmitRedemptionEvent()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L991)
- Saves 1 Gcoldsload + 1 Gkeccak256 (**2130 gas**) on [`StabilityPool.withdrawFromSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L389) (see [`StabilityPool._sendLUSDToDepositor()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L797))
- Saves 3 Gcoldsload + 3 Gkeccak256 (**6390 gas**) on [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L575) (see [`StabilityPool._sendGasCompensation()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L917)) and [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L554)
- Saves 3 Gcoldsload + 3 Gkeccak256 (**6390 gas**) on [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L778) (see [`TroveManager._sendGasCompensation()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L917)) and [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L757)
- Saves 3 Gcoldsload + 3 Gkeccak256 (**6390 gas**) on [`TroveManager.liquidate()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L315)

### [G&#x2011;02]  State variables can be packed into fewer storage slots
If variables occupying the same slot are both written by the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper.

*There are 2 instances of this issue:*

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L37-L69)
```solidity
/// @audit can be packed into 1 storage slot
37      bool public mintingPaused = false;
69      address public borrowerOperationsAddress;
```

- Saves 1 Gcoldsload (**2100 gas**) on [`LUSDToken.mint()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L186-L187) (see [`LUSDToken._requireMintingNotPaused()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L394) + [`LUSDToken._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L362))
- Saves 3 Gcoldsload (**6300 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L190) (see [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L427)) and [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L233-L235) (see [`BorrowerOperations._withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L513))
- Saves 2 Gcoldsload (**4200 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L312) + [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L427)) and [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L491) + [`BorrowerOperations._withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L513))
- Saves 2 Gcoldsload (**4200 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L312) + [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L427)) and [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L491) + [`BorrowerOperations._withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L513))

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L49-L78)
```solidity
/// @audit can be packed into 1 storage slot
49      bool public emergencyShutdown;
78      address public treasury; // address to whom performance fee is remitted in the form of vault shares
```

- Saves 1 Gcoldsload + 1 Gwarmaccess (**2200 gas**) on [`ReaperVaultV2.report()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L504-L555) (see [`ReaperVaultV2._chargeFees()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L467) + [`ReaperVaultV2.availableCapital()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L227))
- Saves 1 Gcoldsload + 1 Gwarmaccess (**2200 gas**) on [`ReaperStrategyGranarySupplyOnly.harvest()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L134)

### [G&#x2011;03]  Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

*There are 101 instances of this issue:*

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L105-L111)
```solidity
105         address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

111             require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
```

- Saves 3 Gwarmaccess (**300 gas**) on [`ActivePool.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L105)

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L288)
```solidity
288             vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]);
```

- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.sendCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L174) (see [`ActivePool._rebalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L288))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.pullCollateralFromBorrowerOperationsOrDefaultPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L211) (see [`ActivePool._rebalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L288))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.manualRebalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L216) (see [`ActivePool._rebalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L288))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L406)
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500)) and [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L498) + [`BorrowerOperations._activePoolAddColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L507))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500)) and [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L498) + [`BorrowerOperations._activePoolAddColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L507))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L500)) and [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L362-L372) + [`BorrowerOperations._moveTokensAndCollateralfromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L498) + [`BorrowerOperations._activePoolAddColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L507))
- Saves 1 Gwarmaccess (**100 gas**) on [`StabilityPool.offset()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L478) (see [`StabilityPool._moveOffsetCollAndDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L607))
- Saves 4 Gwarmaccess (**400 gas**) on [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L564), [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L575) (see [`TroveManager._sendGasCompensation()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L921)), [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L555-L562) (see [`TroveManager._redistributeDebtAndColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1270)) and [`TroveManager.liquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L554)
- Saves 4 Gwarmaccess (**400 gas**) on [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L767), [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L778) (see [`TroveManager._sendGasCompensation()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L921)), [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L758-L765) (see [`TroveManager._redistributeDebtAndColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1270)) and [`TroveManager.batchLiquidateTroves()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L757)
- Saves 1 Gwarmaccess (**100 gas**) on [`TroveManager.redeemCloseTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L952)
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L231) (see [`BorrowerOperations._activePoolAddColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L507))
- Saves 4 Gwarmaccess (**400 gas**) on [`TroveManager.liquidate()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L315)

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315)
```solidity
315             ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),
```

- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.setYieldingPercentage()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L126) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.setYieldClaimThreshold()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L139) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.getCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L160) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.getLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L165) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.sendCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L172) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.increaseLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L191) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.decreaseLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L198) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.pullCollateralFromBorrowerOperationsOrDefaultPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L205) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.manualRebalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L215) (see [`ActivePool._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L315))
- ...

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L328)
```solidity
328         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
```

- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.sendCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L173) (see [`ActivePool._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L328))
- Saves 1 Gwarmaccess (**100 gas**) on [`ActivePool.decreaseLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L199) (see [`ActivePool._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L328))
- ...

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L178-L181)
```solidity
178         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

179         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

180         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

181         vars.price = priceFeed.fetchPrice(_collateral);
```

- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L178-L181)

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289)
```solidity
286         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

287         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

288         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

289         vars.price = priceFeed.fetchPrice(_collateral);
```

- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))
- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))
- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))
- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))
- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))
- Saves 4 Gwarmaccess (**400 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L286-L289))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L381-L389)
```solidity
381         uint256 collCCR = collateralConfig.getCollateralCCR(_collateral);

382         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

383         uint price = priceFeed.fetchPrice(_collateral);

388         uint coll = troveManagerCached.getTroveColl(msg.sender, _collateral);

389         uint debt = troveManagerCached.getTroveDebt(msg.sender, _collateral);
```

- Saves 5 Gwarmaccess (**500 gas**) on [`BorrowerOperations.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L381-L389)

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L421)
```solidity
421         uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);
```

- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L190) (see [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L421))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L312) + [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L421))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L312) + [`BorrowerOperations._triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L421))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471)
```solidity
468         uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)

469                                         : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);

470         uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)

471                                         : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);
```

- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343) + [`BorrowerOperations._updateTroveFromAdjustment()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L468-471))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L525)
```solidity
525         require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
```

- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L173) (see [`BorrowerOperations._requireValidCollateralAddress()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L525))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530)
```solidity
529         require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

530         require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
```

- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L174) (see [`BorrowerOperations._requireSufficientCollateralBalanceAndAllowance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L243) (see [`BorrowerOperations._requireSufficientCollateralBalanceAndAllowance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530))
- Saves 2 Gwarmaccess (**200 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L270) (see [`BorrowerOperations._requireSufficientCollateralBalanceAndAllowance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542)
```solidity
542         uint status = _troveManager.getTroveStatus(_borrower, _collateral);
```

- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L260) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L298) + [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L380) (see [`BorrowerOperations._requireTroveisActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L542))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L547)
```solidity
547         uint status = _troveManager.getTroveStatus(_borrower, _collateral);
```

- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L185) (see [`BorrowerOperations._requireTroveisNotActive()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L547))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645)
```solidity
645         require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
```

- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L185) (see [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.addColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L244) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L340) + [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.moveCollateralGainToTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L250) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L340) + [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.withdrawColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L255) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L340) + [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.repayLUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L265) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L340) + [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L272) (see [`BorrowerOperations._adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L340) + [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))
- Saves 1 Gwarmaccess (**100 gas**) on [`BorrowerOperations.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L391) (see [`BorrowerOperations._requireSufficientLUSDBalance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645))

[Ethos-Core\contracts\CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63)
```solidity
63              uint256 decimals = IERC20(collateral).decimals();
```

- Saves 1 Gwarmaccess (**100 gas**) on [`CollateralConfig.initialize()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L417)
```solidity
417         uint LQTYIssuance = _communityIssuance.issueOath();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L638)
```solidity
638         assets = collateralConfig.getAllowedCollaterals();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L664)
```solidity
664         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L806)
```solidity
806         address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L857-L864)
```solidity
857         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

861             uint price = priceFeed.fetchPrice(collateral);

862             address lowestTrove = sortedTroves.getLast(collateral);

863             uint256 collMCR = collateralConfig.getCollateralMCR(collateral);

864             uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price);
```

- Saves at least 5 Gwarmaccess (**500 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L420)
```solidity
420             uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L521-L525)
```solidity
521         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

522         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

523         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

524         vars.price = priceFeed.fetchPrice(_collateral);

525         vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();
```

- Saves at least 5 Gwarmaccess (**500 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L596-L610)
```solidity
596         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

597         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

598         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

606         vars.user = _sortedTroves.getLast(_collateral);

607         address firstUser = _sortedTroves.getFirst(_collateral);

610             address nextUser = _sortedTroves.getPrev(_collateral, vars.user);
```

- Saves at least 6 Gwarmaccess (**600 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L691)
```solidity
691             vars.user = sortedTrovesCached.getLast(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L725-L729)
```solidity
725         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

726         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

727         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

728         vars.price = priceFeed.fetchPrice(_collateral);

729         vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();
```

- Saves at least 5 Gwarmaccess (**500 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L798-L800)
```solidity
798         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

799         vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

800         vars.collMCR = collateralConfig.getCollateralMCR(_collateral);
```

- Saves at least 3 Gwarmaccess (**300 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1048)
```solidity
1048            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1061)
```solidity
1061            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1131)
```solidity
1131            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1146)
```solidity
1146            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1308-L1309)
```solidity
1308            uint activeColl = _activePool.getCollateral(_collateral);

1309            uint liquidatedColl = defaultPool.getCollateral(_collateral);
```

- Saves at least 2 Gwarmaccess (**200 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1362)
```solidity
1362            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1367-L1368)
```solidity
1367            uint256 collCCR = collateralConfig.getCollateralCCR(_collateral);

1368            uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

- Saves at least 2 Gwarmaccess (**200 gas**)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1539)
```solidity
1539            require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103)
```solidity
103         OathToken.transferFrom(msg.sender, address(this), amount);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127)
```solidity
127         OathToken.transfer(_account, _OathAmount);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135)
```solidity
135             lusdToken.transfer(msg.sender, LUSDGain);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171)
```solidity
171         lusdToken.transfer(msg.sender, LUSDGain);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L204)
```solidity
204         assets = collateralConfig.getAllowedCollaterals();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L226)
```solidity
226         address[] memory collaterals = collateralConfig.getAllowedCollaterals();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L252)
```solidity
252         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L69)
```solidity
69          want = _gWant.UNDERLYING_ASSET_ADDRESS();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120-L127)
```solidity
120             uint256 amount = startToken.balanceOf(address(this));

127         uint256 allocated = IVault(vault).strategies(address(this)).allocated;
```

- Saves at least 2 Gwarmaccess (**200 gas**)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L178)
```solidity
178             address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L227)
```solidity
227         return IERC20Upgradeable(want).balanceOf(address(this));
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L231-L234)
```solidity
231         (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
232             address(want),
233             address(this)
234         );
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L153-L154)
```solidity
153         require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");

154         require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
```

- Saves at least 2 Gwarmaccess (**200 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L246)
```solidity
246             available = Math.min(available, token.balanceOf(address(this)));
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L279)
```solidity
279         return token.balanceOf(address(this)) + totalAllocated;
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L303)
```solidity
303         _deposit(token.balanceOf(msg.sender), msg.sender);
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L327-L329)
```solidity
327         uint256 balBefore = token.balanceOf(address(this));

329         uint256 balAfter = token.balanceOf(address(this));
```

- Saves at least 2 Gwarmaccess (**200 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L368-L399)
```solidity
368         if (value > token.balanceOf(address(this))) {

373                 vaultBalance = token.balanceOf(address(this));

385                 uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));

386                 uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;

399             vaultBalance = token.balanceOf(address(this));
```

- Saves at least 5 Gwarmaccess (**500 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L556)
```solidity
556             return IStrategy(vars.stratAddr).balanceOf();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L641)
```solidity
641         uint256 amount = IERC20Metadata(_token).balanceOf(address(this));
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L651)
```solidity
651         return token.decimals();
```

- Saves at least 1 Gwarmaccess (**100 gas**)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L111-L134)
```solidity
111         int256 availableCapital = IVault(vault).availableCapital();

134         debt = IVault(vault).report(roi, repayment);
```

- Saves at least 2 Gwarmaccess (**200 gas**)

### [G&#x2011;04]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost.

*There are 56 instances of this issue:*

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L71-L83)
```solidity
71      function setAddresses(
72          address _collateralConfigAddress,
73          address _borrowerOperationsAddress,
74          address _troveManagerAddress,
75          address _stabilityPoolAddress,
76          address _defaultPoolAddress,
77          address _collSurplusPoolAddress,
78          address _treasuryAddress,
79          address _lqtyStakingAddress,
80          address[] calldata _erc4626vaults
81      )
82          external
83          onlyOwner
```

- Saves **21 gas** on [`ActivePool.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L83)

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L171-L173)
```solidity
171     function sendCollateral(address _collateral, address _account, uint _amount) external override {
173         _requireCallerIsBOorTroveMorSP();
```

- Saves **21 gas** on [`ActivePool.sendCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L173) (see [`ActivePool._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L329-L334))

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190-L192)
```solidity
190     function increaseLUSDDebt(address _collateral, uint _amount) external override {
192         _requireCallerIsBOorTroveM();
```

- Saves **21 gas** on [`ActivePool.increaseLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L192) (see [`ActivePool._requireCallerIsBOorTroveM()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L338-L341))

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L197-L199)
```solidity
197     function decreaseLUSDDebt(address _collateral, uint _amount) external override {
199         _requireCallerIsBOorTroveMorSP();
```

- Saves **21 gas** on [`ActivePool.decreaseLUSDDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L199) (see [`ActivePool._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L329-L334))

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L204-L206)
```solidity
204     function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {
206         _requireCallerIsBorrowerOperationsOrDefaultPool();
```

- Saves **21 gas** on [`ActivePool.pullCollateralFromBorrowerOperationsOrDefaultPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L306) (see [`ActivePool._requireCallerIsBorrowerOperationsOrDefaultPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L321-L324))

[Ethos-Core\contracts\BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L110-L125)
```solidity
110     function setAddresses(
111         address _collateralConfigAddress,
112         address _troveManagerAddress,
113         address _activePoolAddress,
114         address _defaultPoolAddress,
115         address _stabilityPoolAddress,
116         address _gasPoolAddress,
117         address _collSurplusPoolAddress,
118         address _priceFeedAddress,
119         address _sortedTrovesAddress,
120         address _lusdTokenAddress,
121         address _lqtyStakingAddress
122     )
123         external
124         override
125         onlyOwner
```

- Saves **21 gas** on [`BorrowerOperations.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L125)

[Ethos-Core\contracts\CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L46-L50)
```solidity
46      function initialize(
47          address[] calldata _collaterals,
48          uint256[] calldata _MCRs,
49          uint256[] calldata _CCRs
50      ) external override onlyOwner {
```

- Saves **21 gas** on [`CollateralConfig.initialize()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L50)

[Ethos-Core\contracts\CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L85-L89)
```solidity
85      function updateCollateralRatios(
86          address _collateral,
87          uint256 _MCR,
88          uint256 _CCR
89      ) external onlyOwner checkCollateral(_collateral) {
```

- Saves **21 gas** on [`CollateralConfig.updateCollateralRatios()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L89)

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L133-L137)
```solidity
133     function pauseMinting() external {
134         require(
135             msg.sender == guardianAddress || msg.sender == governanceAddress,
136             "LUSD: Caller is not guardian or governance"
137         );
```

- Saves **21 gas** on [`LUSDToken.pauseMinting()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L134-L137)

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L141-L142)
```solidity
141     function unpauseMinting() external {
142         _requireCallerIsGovernance();
```

- Saves **21 gas** on [`LUSDToken.unpauseMinting()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L142) (see [`LUSDToken._requireCallerIsGovernance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L390))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L147)
```solidity
146     function updateGovernance(address _newGovernanceAddress) external {
147         _requireCallerIsGovernance();
```

- Saves **21 gas** on [`LUSDToken.updateGovernance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L147) (see [`LUSDToken._requireCallerIsGovernance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L390))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L153-L154)
```solidity
153     function updateGuardian(address _newGuardianAddress) external {
154         _requireCallerIsGovernance();
```

- Saves **21 gas** on [`LUSDToken.updateGuardian()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L154) (see [`LUSDToken._requireCallerIsGovernance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L390))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L160-L165)
```solidity
160     function upgradeProtocol(
161         address _newTroveManagerAddress,
162         address _newStabilityPoolAddress,
163         address _newBorrowerOperationsAddress
164     ) external {
165         _requireCallerIsGovernance();
```

- Saves **21 gas** on [`LUSDToken.upgradeProtocol()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L165) (see [`LUSDToken._requireCallerIsGovernance()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L390))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L185-L187)
```solidity
185     function mint(address _account, uint256 _amount) external override {
187         _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`LUSDToken.mint()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L187) (see [`LUSDToken._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L362))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L191-L192)
```solidity
191     function burn(address _account, uint256 _amount) external override {
192         _requireCallerIsBOorTroveMorSP();
```

- Saves **21 gas** on [`LUSDToken.burn()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L192) (see [`LUSDToken._requireCallerIsBOorTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L367-L372))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L196-L197)
```solidity
196     function sendToPool(address _sender,  address _poolAddress, uint256 _amount) external override {
197         _requireCallerIsStabilityPool();
```

- Saves **21 gas** on [`LUSDToken.sendToPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L197) (see [`LUSDToken._requireCallerIsStabilityPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L377))

[Ethos-Core\contracts\LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L201-L202)
```solidity
201     function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override {
202         _requireCallerIsTroveMorSP();
```

- Saves **21 gas** on [`LUSDToken.returnFromPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L202) (see [`LUSDToken._requireCallerIsTroveMorSP()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L384-L386))

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L261-L273)
```solidity
261     function setAddresses(
262         address _borrowerOperationsAddress,
263         address _collateralConfigAddress,
264         address _troveManagerAddress,
265         address _activePoolAddress,
266         address _lusdTokenAddress,
267         address _sortedTrovesAddress,
268         address _priceFeedAddress,
269         address _communityIssuanceAddress
270     )
271         external
272         override
273         onlyOwner
```

- Saves **21 gas** on [`StabilityPool.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L273)

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L466-L467)
```solidity
466     function offset(address _collateral, uint _debtToOffset, uint _collToAdd) external override {
467         _requireCallerIsTroveManager();
```

- Saves **21 gas** on [`StabilityPool.offset()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L467) (see [`StabilityPool._requireCallerIsTroveManager()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L853))

[Ethos-Core\contracts\StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L486-L487)
```solidity
486     function updateRewardSum(address _collateral, uint _collToAdd) external override {
487         _requireCallerIsActivePool();
```

- Saves **21 gas** on [`StabilityPool.updateRewardSum()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L487) (see [`StabilityPool._requireCallerIsActivePool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L849))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L232-L250)
```solidity
232     function setAddresses(
233         address _borrowerOperationsAddress,
234         address _collateralConfigAddress,
235         address _activePoolAddress,
236         address _defaultPoolAddress,
237         address _stabilityPoolAddress,
238         address _gasPoolAddress,
239         address _collSurplusPoolAddress,
240         address _priceFeedAddress,
241         address _lusdTokenAddress,
242         address _sortedTrovesAddress,
243         address _lqtyTokenAddress,
244         address _lqtyStakingAddress,
245         address _redemptionHelperAddress
246     )
247         external
248         override
249     {
250         require(msg.sender == owner);
```

- Saves **21 gas** on [`TroveManager.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L250)

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L939-L945)
```solidity
939     function redeemCloseTrove(
940         address _borrower,
941         address _collateral,
942         uint256 _LUSD,
943         uint256 _collAmount
944     ) external override {
945         _requireCallerIsRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.redeemCloseTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L945) (see [`TroveManager._requireCallerIsRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1527))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L957-L958)
```solidity
957     function reInsert(address _id, address _collateral, uint256 _newNICR, address _prevId, address _nextId) external override {
958         _requireCallerIsRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.reInsert()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L958) (see [`TroveManager._requireCallerIsRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1527))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L962-L968)
```solidity
962     function updateDebtAndCollAndStakesPostRedemption(
963         address _borrower,
964         address _collateral,
965         uint256 _newDebt,
966         uint256 _newColl
967     ) external override {
968         _requireCallerIsRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.updateDebtAndCollAndStakesPostRedemption()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L968) (see [`TroveManager._requireCallerIsRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1527))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L982-L990)
```solidity
982     function burnLUSDAndEmitRedemptionEvent(
983         address _redeemer,
984         address _collateral,
985         uint _attemptedLUSDAmount,
986         uint _actualLUSDAmount,
987         uint _collSent,
988         uint _collFee
989     ) external override {
990         _requireCallerIsRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.burnLUSDAndEmitRedemptionEvent()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L990) (see [`TroveManager._requireCallerIsRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1527))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1076-L1077)
```solidity
1076        function applyPendingRewards(address _borrower, address _collateral) external override {
1077            _requireCallerIsBorrowerOperationsOrRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.applyPendingRewards()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1077) (see [`TroveManager._requireCallerIsBorrowerOperationsOrRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1531))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1111-L1112)
```solidity
1111        function updateTroveRewardSnapshots(address _borrower, address _collateral) external override {
1112            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.updateTroveRewardSnapshots()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1112) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1183-L1184)
```solidity
1183        function removeStake(address _borrower, address _collateral) external override {
1184            _requireCallerIsBorrowerOperationsOrRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.removeStake()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1184) (see [`TroveManager._requireCallerIsBorrowerOperationsOrRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1531))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1195-L1196)
```solidity
1195        function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {
1196            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.updateStakeAndTotalStakes()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1196) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1273-L1274)
```solidity
1273        function closeTrove(address _borrower, address _collateral, uint256 _closedStatusNum) external override {
1274            _requireCallerIsBorrowerOperationsOrRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.closeTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1274) (see [`TroveManager._requireCallerIsBorrowerOperationsOrRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1531))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1316-L1317)
```solidity
1316        function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
1317            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.addTroveOwnerToArray()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1317) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1397-L1403)
```solidity
1397        function updateBaseRateFromRedemption(
1398            uint _collateralDrawn,
1399            uint _price,
1400            uint256 _collDecimals,
1401            uint _totalLUSDSupply
1402        ) external override returns (uint) {
1403            _requireCallerIsRedemptionHelper();
```

- Saves **21 gas** on [`TroveManager.updateBaseRateFromRedemption()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1403) (see [`TroveManager._requireCallerIsRedemptionHelper()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1527))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1485-L1486)
```solidity
1485        function decayBaseRateFromBorrowing() external override {
1486            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.decayBaseRateFromBorrowing()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1486) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1562-L1563)
```solidity
1562        function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
1563            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.setTroveStatus()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1563) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1567-L1568)
```solidity
1567        function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
1568            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.increaseTroveColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1568) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1574-L1575)
```solidity
1574        function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {
1575            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.decreaseTroveColl()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1575) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1581-L1582)
```solidity
1581        function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
1582            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.increaseTroveDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1582) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1588-L1589)
```solidity
1588        function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {
1589            _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`TroveManager.decreaseTroveDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1589) (see [`TroveManager._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1523))

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L67)
```solidity
61      function setAddresses
62      (
63          address _oathTokenAddress, 
64          address _stabilityPoolAddress
65      ) 
66          external 
67          onlyOwner 
```

- Saves **21 gas** on [`CommunityIssuance.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L67)

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L85)
```solidity
84      function issueOath() external override returns (uint issuance) {
85          _requireCallerIsStabilityPool();
```

- Saves **21 gas** on [`CommunityIssuance.issueOath()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L85) (see [`CommunityIssuance._requireCallerIsStabilityPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133))

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124-L125)
```solidity
124     function sendOath(address _account, uint _OathAmount) external override {
125         _requireCallerIsStabilityPool();
```

- Saves **21 gas** on [`CommunityIssuance.sendOath()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L125) (see [`CommunityIssuance._requireCallerIsStabilityPool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L133))

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L76)
```solidity
66      function setAddresses
67      (
68          address _lqtyTokenAddress,
69          address _lusdTokenAddress,
70          address _troveManagerAddress, 
71          address _borrowerOperationsAddress,
72          address _activePoolAddress,
73          address _collateralConfigAddress
74      ) 
75          external 
76          onlyOwner 
```

- Saves **21 gas** on [`LQTYStaking.setAddresses()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L76)

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L177-L178)
```solidity
177     function increaseF_Collateral(address _collateral, uint _collFee) external override {
178         _requireCallerIsTroveManagerOrActivePool();
```

- Saves **21 gas** on [`LQTYStaking.increaseF_Collateral()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L178) (see [`LQTYStaking._requireCallerIsTroveManagerOrActivePool()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L253-L258))

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L187-L188)
```solidity
187     function increaseF_LUSD(uint _LUSDFee) external override {
188         _requireCallerIsBorrowerOperations();
```

- Saves **21 gas** on [`LQTYStaking.increaseF_LUSD()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L188) (see [`LQTYStaking._requireCallerIsBorrowerOperations()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L262))

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L147-L152)
```solidity
147     function updateVeloSwapPath(
148         address _tokenIn,
149         address _tokenOut,
150         address[] calldata _path
151     ) external override {
152         _atLeastRole(STRATEGIST);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.updateVeloSwapPath()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L152)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L161)
```solidity
160     function setHarvestSteps(address[2][] calldata _newSteps) external {
161         _atLeastRole(ADMIN);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.setHarvestSteps()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L161)

[Ethos-Vault\contracts\ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L213-L214)
```solidity
213     function authorizedWithdrawUnderlying(uint256 _amount) external {
214         _atLeastRole(STRATEGIST);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.authorizedWithdrawUnderlying()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L214)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L149)
```solidity
144     function addStrategy(
145         address _strategy,
146         uint256 _feeBPS,
147         uint256 _allocBPS
148     ) external {
149         _atLeastRole(DEFAULT_ADMIN_ROLE);
```

- Saves **21 gas** on [`ReaperVaultV2.addStrategy()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L149)
- Saves **21 gas** on [`ReaperVaultERC4626.addStrategy()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L149)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L179)
```solidity
178     function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
179         _atLeastRole(ADMIN);
```

- Saves **21 gas** on [`ReaperVaultV2.updateStrategyFeeBPS()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L179)
- Saves **21 gas** on [`ReaperVaultERC4626.updateStrategyFeeBPS()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L179)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L192)
```solidity
191     function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {
192        _atLeastRole(STRATEGIST);
```

- Saves **21 gas** on [`ReaperVaultV2.updateStrategyAllocBPS()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L192)
- Saves **21 gas** on [`ReaperVaultERC4626.updateStrategyAllocBPS()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L192)

[Ethos-Vault\contracts\ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L259)
```solidity
258     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
259        _atLeastRole(ADMIN);
```

- Saves **21 gas** on [`ReaperVaultV2.setWithdrawalQueue()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L259)
- Saves **21 gas** on [`ReaperVaultERC4626.setWithdrawalQueue()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L259)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L96)
```solidity
95      function withdraw(uint256 _amount) external override returns (uint256 loss) {
96          require(msg.sender == vault, "Only vault can withdraw");
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.withdraw()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L96)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L109-L110)
```solidity
109     function harvest() external override returns (int256 roi) {
110        _atLeastRole(KEEPER);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.harvest()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L110)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L157)
```solidity
156     function setEmergencyExit() external {
157         _atLeastRole(GUARDIAN);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.setEmergencyExit()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L157)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L167-L168)
```solidity
167     function initiateUpgradeCooldown() external {
168         _atLeastRole(STRATEGIST);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.initiateUpgradeCooldown()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L168)

[Ethos-Vault\contracts\abstract\ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L179-L180)
```solidity
179     function clearUpgradeCooldown() public {
180         _atLeastRole(GUARDIAN);
```

- Saves **21 gas** on [`ReaperStrategyGranarySupplyOnly.clearUpgradeCooldown()`](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L180)
