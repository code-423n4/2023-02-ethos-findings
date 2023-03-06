### 1. Incorrect tvlCap check in ReaperVaultV2._deposit()

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L324

`_amount` is updated after transferring tokens to the contract.
It would be more accurate to move this require right after the `_amount` reassignment (line 330).

### 2. Duplicated checking if the collateral is allowed in BorrowerOperations.openTrove()

`_requireValidCollateralAddress(_collateral)` could be removed from `openTrove()` function since it’s checked when getting collateralCCR or collateralMCR.