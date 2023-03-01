# Low

* Strategy can return `0` from `IStrategy.withdraw()` and avoid de-allocation during `ReaperVaultV2._withdraw()`
* ERC4626 vault wrapper contract lacks first-depositor attack mitigation
* No `StrategyReported` event emission during `ReaperVaultV2._withdraw()` in the event of a reported loss
* `ReaperVaultV2` and `ReaperBaseStrategyv4` initialization does not check for strategist addresses in the administrative users array
* `ReaperVaultV2` lacks administrative users array length check present in `ReaperBaseStrategyv4`
* Strategist can time allocation update to avoid the `StrategyRevoked` event emission
* Tests lack coverage of important functions and don't exercise code paths
* Consider stronger bounds on `setLockedProfitDegredation`
* `CollateralConfig.updateCollateralRatios()` implementation drifts from comments
* `CommunityIssuance.updateDistributionPeriod()` setter lacks bounds checks


## Strategy can return `0` from `IStrategy.withdraw()` and avoid de-allocation during `ReaperVaultV2._withdraw()`


Non-0 `loss` values returned from `IStrategy.withdraw()` cause the execution of `ReaperVaultV2._withdraw()` to take a branch whereby the strategy will lose allocation points via `_reportLoss()`.

A strategy can self-report `0` loss but still send all tokens from any internal actions, like self-liquidation. This could potentially upset the accounting between the two contracts.

As likely as it is that all strategy contracts are vetted before being included in the `ReaperVault`, it would be preferrable for the `ReaperVault` to directly read a designated state value from the contract, one that must (by design) reflect the strategy's internal accounting of its debt. 

This significantly lessens the chance that an unintentional return value of 0 causes later issues because of some bug in the strategy.

## ERC4626 vault wrapper contract lacks first-depositor attack mitigation

`ReaperVaultERC4626` wraps the underlying `ReaperVaultV2` to make it compatible with consumers of the ERC4626 standard. ERC4626 has a known issue with the first vault depositor, whereby a very small initial deposit allows an attacker to significantly disrupt the efficacy of subsequent deposits. More information can be found [here](https://github.com/transmissions11/solmate/issues/178).

The wrapper currently does not implement a mitigitation for this. Normally, this would be a substantial issue, but the vault itself isn't open to the general public, only to certain actors. As such, the impact is significantly lessened but still potentially disruptive.

## No `StrategyReported` event emission during `ReaperVaultV2._withdraw()` in the event of a reported loss

Normally during a self-reported loss via `ReaperVaultV2.report()`, the function will emit a `StrategyReported` event, with information indicating a loss or gain in value in terms of the loaned token.

This allows observers to track the performance of a strategy themselves. However, no event is emitted during `ReaperVaultV2._withdraw()` even if `_reportLoss()` is conditionally called. This makes it harder to see if a strategy was operating at a loss when `_withdraw()` was called.

Consider adding a value to the `Withdraw` event that reports the sum of incurred losses, as well as a loss Event to `_reportLoss()` for easier tracking on-chain.

## `ReaperVaultV2` and `ReaperBaseStrategyv4` initialization does not check for strategist addresses in the administrative users array

Both of these contracts have initialization functions that set access control roles to various addresses. It is very important that the array of addresses used to populate administrative users does not also contain any address from the `_strategists` array. 

A check should be added to explicitly account for this important security boundary.

## `ReaperVaultV2` lacks administrative users array length check present in `ReaperBaseStrategyv4`

`ReaperBaseStrategyv4`'s initialization function contains a safety check for administrative user array length, but this check is absent from the mirrored function in `ReaperVaultV2`. This check prevents error during initialization and should be included.

## Strategist can time allocation update to avoid the `StrategyRevoked` event emission

Administrative users down to the `guardian` level have access to `reaperVaultV2.revokeStrategy()`, which revokes the distribution allocation for strategies. 

A strategist monitoring the mempool can look for such a call, and attempt to sandwich it with a pair of calls to `ReaperVaultV2.updateStrategyAllocBPS()`. The first call would set the allocation to 0, and the second call would return it to the original allocation.

The effect is that, when `revokeStrategy()` is entered, it will return early via

```
if (strategies[_strategy].allocBPS == 0) {return;}
```

which avoids the emission of the `StrategyRevoked` event. Monitors of the chain would see the allocation changes but not the actual revocation event. Users only monitoring for `StrategyRevoked` wouldn't see anything amiss at all.

Consider emitting the `StrategyRevoked` event even if the allocation is already 0. Better would be to implement a stateful enum as suggested in another finding, and set a revoked state via the enum in addition to the event.

## Consider stronger bounds on `setLockedProfitDegredation`

While the setter function `ReaperVault.setLockedProfitDegredation()` has a check to ensure the value isn't higher than the maximum desired value, the bound check is arguably not very effective.

The business case for allowing maximal and minimal values should be added to the documentation. Otherwise, a strong set of boundary values should be checked for, based on the analysis of their effect on the vault. Allowing any value inside the full range without documentation on appropriate situations for each could lead to operator error or unpredictable effects on vault status.

## `CollateralConfig.updateCollateralRatios()` implementation drifts from comments

Code comments and require strings in `CollateralConfig.updateCollateralRatios()` indicate that the new values passed to this function will be rejected if they attempt to increase the current state values.

However, they actually allow the same value to be re-used:

```
function updateCollateralRatios(address _collateral, uint256 _MCR, uint256 _CCR) external onlyOwner checkCollateral(_collateral) {
    Config storage config = collateralConfig[_collateral];
    require(_MCR <= config.MCR, "Can only walk down the MCR"); // <= instead of <
    require(_CCR <= config.CCR, "Can only walk down the CCR"); // <= instead of <

    require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
    config.MCR = _MCR; // 🟡

    require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
    config.CCR = _CCR; // 🟡
    emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
}
```

Update the require strings and comments to reflect the code, or update the code to reflect the intention of the comments and require strings.

## `CommunityIssuance.updateDistributionPeriod()` setter lacks bounds checks

This administrative function lacks any sort of bounds checking. There is no guidance in the comments on appropriate values, nor their effect on the system as a whole. 

Guidance should be included, and a sensible range of allowed values should be required by the setter. Otherwise, a poor setting could have an effect on token distribution and ecosystem valuation of the token as a result.

# NC

* License typo in ReaperStrategyGranarySupplyOnly
* Some contracts lack events for important state changes
* Variable with confusing name in `_liquidatePosition`
* Consider moving `require` to top of function to avoid wasting gas
* Reason string in require doesn't reflect actual content of require
* Omitted `emit` keyword
* Add "nor RedemptionHelper" to revert string
* Cast `lqtyStaking` to address when needed rather than storing address
* `CollSurplusPool.pullCollateralFromActivePool()` lacks event emission present in corresponding `DefaultPool.pullCollateralFromActivePool()` function
* `RedemptionHelper` revert strings missing
* Define enum rather than pass magic values in `RedemptionHelper._redeemCollateralFromTrove`
* Consider swapping order of arguments to match other similar function signatures
* Comment refers to ETH collateral

## License typo in ReaperStrategyGranarySupplyOnly

License line typo, should be `// SPDX-License-Identifier: BUSL-1.1`

## Some contracts lack events for important state changes

Contracts should generally include event emissions for important state changes like admin user actions, fees, bounded parameter changes, etc.

Here are functions where event should be considered:

`ReaperVaultV2.updateTreasury()`
`ReaperStrategyBasev4.harvest()`
`ReaperStrategyBasev4.initiateUpgradeCooldown()`
`ReaperStrategyBasev4.clearUpgradeCooldown()`
`ReaperStrategyGranarySupplyOnly.updateVeloSwapPath()`
`ReaperStrategyGranarySupplyOnly.setHarvestSteps()`
`ReaperStrategyGranarySupplyOnly.authorizedWithdrawUnderlying()`
`ActivePool._rebalance()`
`CommunityIssuance.updateDistributionPeriod()`

## Variable with confusing name in `ReaperStrategyGranarySupplyOnly._liquidatePosition()`

The named function uses the variable `liquidatedAmount` as a return value. Its name implies that the quantity is the result of a liquidation process, but in reality its just the current held quantity of tokens after liquidation, which might be in part tokens already held. If no liquidation was needed, then the value is just set to the requested quantity of tokens, making the name more inapplicable.

Consider changing the name to better reflect its derivation or purpose, such as `quantityFreed` or `amountFreed` as used in the context of this function's callers.

## Consider moving `require` to top of function to avoid wasting gas

`ReaperBaseStrategyv4.__ReaperBaseStrategy_init()` contains an important safety check on the exact length of the `_multisigRoles` argument. It would be best to move this require to the top of the function rather than down at the state assignment, as there's no purpose in performing all of the steps in between if the require will fail. 

Also consider adding a check on the `_multisigRoles` argument for duplicate addresses; the security model of the contract is weakened if a single address is added to more than one tier of access privilege.

## Reason string in require doesn't reflect actual content of require

`ReaperBaseStrategyv4.withdraw()` contains a `require` that ensures that the `_amount` argument is less than or equal to the current balance of the contract. However, the reason string in the event of a revert says that the quantity must be less than the balance. 

`require(_amount <= balanceOf(), "Ammount must be less than balance");`

This is incorrect. It can be equal or less than the balance. Update the string to reflect the code, and fix the typo in 'amount.'

## Omitted `emit` keyword

`ActivePool` contains the `ActivePoolLUSDDebtUpdated` event. It's use in `increaseLUSDDebt` and `decreaseLUSDDebt` omits the use of the `emit` keyword. 

The compiler version used in these contracts doesn't require the use of the keyword, but to maintain consistency with other events in the codebase, consider adding the keyword anyway.

## Add "nor RedemptionHelper" to revert string

`ActivePool._requireCallerIsBOorTroveMorSP()` is an internal helper function to block access to functions based on the caller. Its revert string indicates one of three conditions, but there are actually four checks.

Consider changing the string to better represent the checks, like "ActivePool: Caller is not BorrowerOperations, TroveManager, RedemptionHelper, or StabilityPool"

## Cast `lqtyStaking` to address when needed, rather than storing address

`BorrowerOperations.setAddresses()` is used to set a storage value for the LQTYStaking contract(as an interface), as well as its address. 

However, the storage address is only used once in the entire contract, `_triggerBorrowingFee()`.

Rather than store this address separately, consider casting the Interface to an address: 

`_lusdToken.mint(address(lqtyStaking), LUSDFee);`

This reuses available information rather than storing essentially a duplicate.

## `CollSurplusPool.pullCollateralFromActivePool()` lacks event emission present in corresponding `DefaultPool.pullCollateralFromActivePool()` function

As stated, the `DefaultPoolCollateralBalanceUpdated` event lacks an matching event in `CollSurplusPool` and the corresponding event emission.

## `RedemptionHelper` revert strings missing

`RedemptionHelper` contract exists to skirt around the contract maximum size requirement. However, now that the contract exists as a separate entity, there's no reason to strip revert strings out of it to save space.

Consider re-adding the relevant revert strings back into the `require` checks.

## Define enum rather than pass magic values in `RedemptionHelper._redeemCollateralFromTrove()`

RedemptionHelper lacks a local definition of the `Status` enum from `TroveManager`, and as such passes the underlying `uint8` value during `_redeemCollateralFromTrove()`. 

For consistentcy and clarity, consider re-defining the enum locally so that the intent of the code is clear and recognizable.

## Consider swapping order of arguments to match other similar function signatures

`TroveManager.liquidate()` is a function modified from the forked source in order to incorporate the `_collateral` argument. 

Notably though, the order of these arguments is inverted from other similar functions, like `batchLiquidateTroves()` and `liquidateTroves`, where the `_collateral` argument appears first. As these are both address values, it is easier to accidentally swap them and waste gas.

## Comment refers to ETH collateral

`TroveManager.redeemCloseTrove()` still refers to the collateral as 'ETH' as in the fork source. Given the extensive effort to refer generically to 'collateral' throughout the codebase and comments, consider removing this dangling reference to the previous collateral setup.