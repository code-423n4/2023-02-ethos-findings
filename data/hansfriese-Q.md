## Low severity findings

### [L-01] Core contracts are susceptible to initializer re-entrancy due to OpenZeppelin version advisory

The version of OpenZeppelin in the core package falls within [this advisory](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-9c22-pwxw-p6hx), so advised to upgrade to avoid accidentally adding a vulnerability if external calls are made.

### [L-02] Re-entrancy risk in openTrove

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L526
  Although unlikely bad collateral will be included, it is recommended to keep CEI pattern.
  Otherwise, malicious strategy can reenter `report()` function and cause problems.
  (It is understood strategy is added by admin and this is very unlikely)

### [L-03] No rewards swapped in \_harvestCore

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L114-L125

When `steps` are not initialized it seems admin has to `setHarvestSteps` first, but this may not happen so rewards aren't swapped.

### [L-04] Call `_disableInitializers()`

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61
  It is recommended to call `_disableInitializers()` in the initializer function.

### [L-05] Wrong event string

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155
  `require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");`
  I believe the code is correct and the event string is misleading here, should be changed into 2000 BPS or 20%.

### [L-06] Wrong comments

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L660
  "collateral ratio" → "nominal .."
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L49
  "ETH" → "collateral"

### [L-07] Double entrypoint token can break vault if `inCaseTokensGetStuck` called with legacy address

`ReaperVaultV2::inCaseTokensGetStuck` can be called by an admin to withdraw tokens which are mistakenly sent to the vault.
A double entrypoint token messes this up to cause complete withdrawal, breaking the vault.

Although it is understood collateral tokens are added by admin; it certainly wouldn't be ideal as collaterals are immutable and USDT being upgradeable/high quality/large market cap collateral makes it a good candidate for this vulnerability.

### [L-08] Inconsistency between `previewDeposit` and `deposit` of `ReapervaultV2`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L52
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334
If `_freeFunds()==0`, the `ReapervaultV2::previewDeposit` returns the same amount to the input `assets` value.
But in actual deposit, it reverts when `freeFunds==0` at ReaperVaultV2.sol#L334

### [L-09] Sanity check in `ReaperVaultV2::setLockedProfitDegradation`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617
`lockedProfitDegradation` should be greater than zero or else funds are not released.
It is understood that it is reversible by calling the same function with a correct value, but recommend to add this check.

## Refactoring findings

### [R-01] Use emit keyword for events

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L194;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L201;
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L198

### [R-02] Inconsistent use of uint/uint256

In many places, the protocol is using `uint` and `uint256`.
It is recommended to use consistent type names.

### [R-03] Misleading variable name

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715
  Based on the context it is recommended to rename `_troveArray` into `_troveOwnerArray`.

## Noncritical findings

### [N-01] Use more explicit modifier names

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L128

The modifier `checkCollateral` can be renamed to be more explicit with what it is "checking", for example:

```
isValidCollateral
isAllowedCollateral
requireValidCollateral
```

### [N-02] `withdrawFromSp` doesn't `_requireNonZeroAmount()` unlike `provideToSP`

Although this does not seems to cause problems, it is recommended to keep consistency for these functions.

### [N-03] Add access control to the `ReaperStrategygranarySupplyOnly.initialize()`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62

### [N-04] Remove console.log imports

All instances should be removed.

### [N-05] Missing info in the emitted event string

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L327-L335
It is good to include the actual `redemptionHelper` address value.
