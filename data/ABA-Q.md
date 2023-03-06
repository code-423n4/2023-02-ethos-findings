# QA Report for Ethos Reserve contest

## Overview
During the audit, 7 low and 14 non-critical issues were found.

### Low Risk Issues

Total: 11 instances over 7 issues

|#|Issue|Instances|
|-|:-|:-:|
| [L-01] | `hasPendingRewards` only checks for pending collateral rewards and not for pending LUSD Debt Rewards | 1 | 
| [L-02] | Fallback price oracle Tellor can be easily manipulated | 1 |
| [L-03] | Local variables or function arguments declared as `storage` without actual need | 5 | 
| [L-04] | `setHarvestSteps` accepts an empty array which simply deletes the steps | 1 | 
| [L-05] | `updateStrategyFeeBPS` should have differentiating behavior when `emergencyShutdown` is set | 1 | 
| [L-06] | Ambiguous implementation of `setWithdrawalQueue` may lead to unforeseen issues | 1 | 
| [L-07] | There is no way to completely remove a strategy from the ReaperVault | 1 | 

### Non-critical Issues

Total: 29 instances over 14 issues

|#|Issue|Instances|
|-|:-|:-:|
|[NC-01]| `updateCollateralRatios` accepts redundant updating with the same values | 1 |
|[NC-02]| Not enough tests for calculating collateral surplus in a sub-case of recovery mode | 6 |
|[NC-03]| Use "type(uint256).max" instead of "uint256(-1)" | 1 |
|[NC-04]| Inconsistent event emission syntax used | 2 |
|[NC-05]| Incomplete require error messages | 2 |
|[NC-06]| Unused code | 1 |
|[NC-07]| Missing events for critical parameter changes | 3 |
|[NC-08]| `_requireValidMaxFeePercentage` can be rewritten to optimize calculations | 1 |
|[NC-09]| `_cascadingAccessRoles` can be pure instead of view | 2 |
|[NC-10]| Dead Comments | 1 |
|[NC-11]| Inconsistent shares calculation across code base | 2 |
|[NC-12]| Open TODOs | 3 |
|[NC-13]| Already declared variable not reused | 2 |
|[NC-14]| Functions do not respect camelCase naming convention | 2 |


## Low Risk Issues (7)
#
### [L-01] `hasPendingRewards` only checks for pending collateral rewards and not for pending LUSD Debt Rewards
##### Description

`applyPendingRewards` from [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol), commits what pending rewards a borrower has to internal accounting. It uses the function `hasPendingRewards` to determine if there are pending rewards to work with. If not, will exist without doing anything.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1076-L1083
```Solidity
    function applyPendingRewards(address _borrower, address _collateral) external override {
        _requireCallerIsBorrowerOperationsOrRedemptionHelper();
        return _applyPendingRewards(activePool, defaultPool, _borrower, _collateral);
    }


    // Add the borrowers's coll and debt rewards earned from redistributions, to their Trove
    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
        if (hasPendingRewards(_borrower, _collateral)) {
```


The issue is with `hasPendingRewards` that only checks if pending collateral rewards are present

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1160
```Solidity
        return (rewardSnapshots[_borrower][_collateral].collAmount < L_Collateral[_collateral]);
```

even though both pending collateral and LUSD rewards are then processed and distributed

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1087-L1088
```Solidity
            uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);
            uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);
```

This issue may lead to pending LUSD rewards for users to not be distributed in a timely manner when `redeemCollateral` is called by a redeemer from `TroveManager`.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1083

```Solidity
        if (hasPendingRewards(_borrower, _collateral)) {
```

##### Recommendation
Modify `hasPendingRewards` to also check for pending LUSD Debt rewards.

#

### [L-02] Fallback price oracle Tellor can be easily manipulated
##### Description

Tellor is a Decentralized Oracle Protocol, which incentivizes permissionless data reporting and data validation. Anyone can report offchain data to Tellor oracles and any on-chain contract can consume that data.
The only requirement to be a reporter is to stake 10 TRB tokens (Tellor protocol token). If the reporter reports incorrect price their staked amount (10 TRB) is slashed.
At the time of this writing, TRB (Tellor) token price is $15.28 so an oracle manipulation can be done with less then $160.

An exploit using a manipulated Tellor price has actually happened quite recently with *Bonq protocol*. A more detailed explanation can be found by warden [`Akshay Srivastav`](https://twitter.com/akshaysrivastv/status/1621024868304293890).

This issue affects `PriceFeed.sol` directly, which we are aware is out of scope, and indirectly the entire protocol.
Although the implementation in `PriceFeed.sol` does have several safety features that will not allow a simple oracle manipulation attack (as well as Tellor being a backup solution) having an easily manipulated price oracle can lead to an actual exploit in the right conditions in the future and cannot be considered a good practice.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L558
```Solidity
        try tellorCaller.getTellorCurrentValue(tellorQueryId[_collateral]) returns
```

##### Recommendation

Use a different secondary/backup price oracle.

#

### [L-03] Local variables or function arguments declared as `storage` without actual need

##### Description

There are place where local variables are declared as `storage`. This is usually done when expecting to modify the storage content, which in these cases is not actually done.
In some of those cases, functions that use them also declare them as `storage` but should be set to `memory`.

##### Instances (5)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L630
```Solidity
            Snapshots storage snapshots = depositSnapshots[_depositor];
```
Also modify `_getCollateralGainFromSnapshots` and `_getSingularCollateralGain` to change the `Snapshots` type parameter from `storage` to memory, as it's never committed to.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637
```Solidity
    function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L657
```Solidity
    function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L118
```Solidity
            address[2] storage step = steps[i];
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L266
```Solidity
            StrategyParams storage params = strategies[strategy];
```

##### Recommendation

Modify all instances to `memory` instead of `storage`.

#

### [L-04] `setHarvestSteps` accepts an empty array which simply deletes the steps

##### Description

The function `setHarvestSteps` from `ReaperStrategyGranarySupplyOnly.sol`:
- accepts as input `address[2][] calldata _newSteps`. It only checks if
- deletes the [original steps](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L162)
- [if there are any steps given, will iterate through them](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L164-L165)
- if the [provided steps are valid](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167-L168) will add them to the new steps array

If an empty array is given, will simply delete the steps. This will not cause an error/ revert in code as the list is again checked for size in `_harvestCore`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L116-L118

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L165
```Solidity
    function setHarvestSteps(address[2][] calldata _newSteps) external {
        _atLeastRole(ADMIN);
        delete steps;


        uint256 numSteps = _newSteps.length;
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
```

##### Recommendation

If intended document it or make a separate function for deleting the array for clarity.
If not intended add a check that `_newSteps` array length to not be 0

#

### [L-05] `updateStrategyFeeBPS` should have differentiating behavior when `emergencyShutdown` is set
##### Description

`updateStrategyFeeBPS` from `ReaperVaultV2.sol` allows the fee BPS of a strategy to change even when `emergencyShutdown`. 
Lowering the fee may serve to help the protocol when in emergency shutdown but rising it will not. 

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184
```Solidity
    function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
        _atLeastRole(ADMIN);
        require(strategies[_strategy].activation != 0, "Invalid strategy address");
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        strategies[_strategy].feeBPS = _feeBPS;
        emit StrategyFeeBPSUpdated(_strategy, _feeBPS);
    }
```

##### Recommendation

Implement a subcase that, if the protocol is in emergency shutdown that the new fee BPS should not be higher then the previous one. 
Example implementation:

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..43cd745 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -178,6 +178,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
         _atLeastRole(ADMIN);
         require(strategies[_strategy].activation != 0, "Invalid strategy address");
+        if(emergencyShutdown) require(_feeBPS < strategies[_strategy].feeBPS, "Fee cannot be higher previous fee BPS during emergency shutdown");
         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
         strategies[_strategy].feeBPS = _feeBPS;
         emit StrategyFeeBPSUpdated(_strategy, _feeBPS);

```
#

### [L-06] Ambiguous implementation of `setWithdrawalQueue` may lead to unforeseen issues

##### Description 

In `ReaperVaultV2` the function `setWithdrawalQueue` is used to modify the `withdrawalQueue` for the utilized strategies.
Function deletes original queue and proceeds to validate/update with the new provided one.

There are too many subcases that this function does not explicitly cover:
- no deduplication is done. If by mistake 2 or more identical strategies are set, will not cause any issue
- protocol may use the function to remove a strategy from withdrawal. This must be documented
- although the strategy is removed from queue, it still exists and is operated with within the Vault
- if only a reordering is intended, a check/parameter to ensure same-size queue can facilitate a more safe transition
- a revoked strategy can be added to the queue

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L258-L271

```Solidity
    function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {
        _atLeastRole(ADMIN);
        uint256 queueLength = _withdrawalQueue.length;
        require(queueLength != 0, "Queue must not be empty");


        delete withdrawalQueue;
        for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
            address strategy = _withdrawalQueue[i];
            StrategyParams storage params = strategies[strategy];
            require(params.activation != 0, "Invalid strategy address");
            withdrawalQueue.push(strategy);
        }
        emit UpdateWithdrawalQueue(_withdrawalQueue);
    }
```

##### Recommendation

Modify the documentation or function to clearly define it's purpose and expected usecases.

#

### [L-07] There is no way to completely remove a strategy from the ReaperVault
##### Description

`ReaperVaultV2` supports adding strategies via `addStrategy` but does not support removing them.
Through a combination of calls:
- updateStrategyFeeBPS
- updateStrategyAllocBPS
- revokeStrategy
- setWithdrawalQueue

A strategy will be reduced as not to impacti the protocol in a negative way while also allowing for it to pay off it's debt (via `report`) even if it was set with lowest possible permissions.
After a strategy has paid off it's debt and there is no more use for it, if there will ever be such a case, there is currently no complete removal mechanism implementation.

Actions that are possible to be done on a revoked, debtless, stripped strategy:
- add to withdrawal queue via `setWithdrawalQueue`
- change it's configurations via:
    - `updateStrategyFeeBPS`
    - `updateStrategyAllocBPS`
- report profit/loss as well as any repayment of debt via `report`

The above check for an inexistent strategy only via `strategy.activation` variable (which is set with the block timestamp of when the strategy was added) and is never reset.
`require(strategy.activation != 0, "Unauthorized strategy");`

This issue does not lead to any loss of funds as the operations above, although will not directly reject a strategy, have code paths that use the parameters of a revoked strategy in a correct manner.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

##### Recommendation

Add a removeStrategy function for when the complete removal of a strategy can be done.
Ideally it should set strategy activation to 0 // delete it from the `strategies` array only after debt has been paid.

#

## Non-critical Issues (14)

### [NC-01] `updateCollateralRatios` accepts redundant updating with the same values

##### Description

The function `updateCollateralRatios` from [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol) changes the MCR and CCR of a particular collateral. 
As a filter, they require that MCR/CCR be less _or equal_ to the existing configured values. The equality has no sense as it would only serve to lose gas setting it without actually modifying anything.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L90-L92sol#L85-L92
```Solidity
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");
```

##### Recommendation
Modify the 2 require statements to accept only "<" versus "<=" how it is now

#

### [NC-02] Not enough tests for calculating collateral surplus in a sub-case of recovery mode
##### Description

In [`TroveManager.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol) when a specific subcase of recovery mode is activated, a calculated value, collateral surplus, is determined in `_getCappedOffsetVals` which users can then withdrawal.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L478-L507

```Solidity
    function _getCappedOffsetVals
    (
        uint _entireTroveDebt,
        uint _entireTroveColl,
        uint _price,
        uint256 _MCR,
        uint _collDecimals
    )
        internal
        pure
        returns (LiquidationValues memory singleLiquidation)
    {
        // ... non-relevant code ...
    
        uint cappedCollPortion = _entireTroveDebt.mul(_MCR).div(_price);
        if (_collDecimals < LiquityMath.CR_CALCULATION_DECIMALS) {
            cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));
        } else if (_collDecimals > LiquityMath.CR_CALCULATION_DECIMALS) {
            cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));
        }

        // ... non-relevant code ...
    
        singleLiquidation.collSurplus = _entireTroveColl.sub(cappedCollPortion);
    
        // ... non-relevant code ...    
    }
```

Since this is one of only 2 places in `TroveManager.sol` where *price* is used directly (not passed to and used in `LiquityMath`), this area should see more tests, especially that it treats different decimals tokens.

Also must take into consideration that price has always 18 decimals with the current protocol implementation.

##### Instances (6)

Existing test only verify that the resulted value is calculated as expected, not that the actual value is calculated correctly.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/test/TroveManager_RecoveryModeTest.js#L927-L936
```Solidity
    // check collateral surplus
    const dennis_scaledCollateral = D_totalDebt.mul(coll0MCR).div(price)
    const bob_scaledCollateral = B_totalDebt.mul(coll0MCR).div(price)
    const carol_scaledCollateral = C_totalDebt.mul(coll0MCR).div(price)
    const dennis_remainingCollateral = D_coll.sub(th.unwrapCollNativeDec(dennis_scaledCollateral, coll0Decimals))
    const bob_remainingCollateral = B_coll.sub(th.unwrapCollNativeDec(bob_scaledCollateral, coll0Decimals))
    const carol_remainingCollateral = C_coll.sub(th.unwrapCollNativeDec(carol_scaledCollateral, coll0Decimals))
    th.assertIsApproximatelyEqual(await collSurplusPool.getUserCollateral(dennis, collaterals[0].address), dennis_remainingCollateral)
    th.assertIsApproximatelyEqual(await collSurplusPool.getUserCollateral(bob, collaterals[0].address), bob_remainingCollateral)
    th.assertIsApproximatelyEqual(await collSurplusPool.getUserCollateral(carol, collaterals[0].address), carol_remainingCollateral)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/test/TroveManager_RecoveryModeTest.js#L1681-L1683
```Solidity
    const bob_scaledCollateral = B_totalDebt.mul(coll0MCR).div(price)
    const bob_remainingCollateral = B_coll.sub(th.unwrapCollNativeDec(bob_scaledCollateral, coll0Decimals))
    th.assertIsApproximatelyEqual(await collSurplusPool.getUserCollateral(bob, collaterals[0].address), bob_remainingCollateral)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/test/TroveManager_RecoveryModeTest.js#L1759-L1762
```Solidity
    // check Bob’s collateral surplus
    const bob_scaledCollateral = B_totalDebt_2.mul(coll0MCR).div(price)
    const bob_remainingCollateral = B_coll_2.sub(th.unwrapCollNativeDec(bob_scaledCollateral, coll0Decimals))
    th.assertIsApproximatelyEqual((await collSurplusPool.getUserCollateral(bob, collaterals[0].address)).toString(), bob_remainingCollateral.toString())
```

##### Recommendation
Extend tests:

- add to `Ethos-Core/utils/deploymentHelpers.js` the following token

```diff
diff --git a/Ethos-Core/utils/deploymentHelpers.js b/Ethos-Core/utils/deploymentHelpers.js
index 8f80307..1e868f7 100644
--- a/Ethos-Core/utils/deploymentHelpers.js
+++ b/Ethos-Core/utils/deploymentHelpers.js
@@ -334,12 +334,14 @@ class DeploymentHelper {
     const multisig = "0x3b410908e71Ee04e7dE2a87f8F9003AFe6c1c7cE"  // Arbitrary address for the multisig, which is not tested in this file
     const collateral1 = await ERC20.new("Wrapped Ether", "wETH", 12, multisig, 0); // 12 decimal places
     const collateral2 = await ERC20.new("Wrapped Bitcoin", "wBTC", 8, multisig, 0); // 8 decimal places
+    const collateral3 = await ERC20.new("YAM Finance v2 ", "YAM-V2", 24, multisig, 0); // 24 decimal places
     const vault1 = await ERC4626.new(collateral1.address, "wETH Crypt", "rfwETH");
     const vault2 = await ERC4626.new(collateral2.address, "wBTC Crypt", "rfwBTC");
+    const vault3 = await ERC4626.new(collateral3.address, "YAM-V2 Crypt", "rfYAM-V2");
 
     contracts.treasury = multisig;
-    contracts.collaterals = [collateral1, collateral2];
-    contracts.erc4626vaults = [vault1, vault2];
+    contracts.collaterals = [collateral1, collateral2, collateral3];
+    contracts.erc4626vaults = [vault1, vault2, vault3];
     return contracts
   }
 
@@ -379,8 +381,8 @@ class DeploymentHelper {
   static async connectCoreContracts(contracts, LQTYContracts) {
     await contracts.collateralConfig.initialize(
       contracts.collaterals.map(c => c.address),
-      [toBN(dec(12, 17)), toBN(dec(13, 17))], // MCR for WETH at 120%, and for WBTC at 130%
-      [toBN(dec(165, 16)), toBN(dec(18, 17))] // CCR for WETH at 165%, and for WBTC at 180%
+      [toBN(dec(12, 17)), toBN(dec(13, 17)), toBN(dec(12, 17))], // MCR for WETH at 120%, for WBTC at 130%, and for YAM-V2 at 120%
+      [toBN(dec(165, 16)), toBN(dec(18, 17)), toBN(dec(165, 16))] // CCR for WETH at 165%, for WBTC at 180% and for YAM-V2 at 165%
     )
 
     // set TroveManager addr in SortedTroves
```

Add the following 2 tests to `Ethos-Core/test/TroveManager_RecoveryModeTest.js`:

```Solidity
  it("liquidate(), with MCR < ICR < TCR, digits(collateral) > 18, collateral surplus corect calculation", async () => {
    // --- SETUP ---
    // Alice withdraws up to 1500 LUSD of debt, resulting in ICRs of 266%.
    // Bob withdraws up to 480 LUSD of debt, resulting in ICR of 250%. Bob has lowest ICR.
    const { collateral: B_coll, totalDebt: B_totalDebt } = await openTrove({ collateral: collaterals[2], ICR: toBN(dec(250, 16)), extraLUSDAmount: dec(480, 18), extraParams: { from: bob } })
    const { collateral: A_coll } = await openTrove({ collateral: collaterals[2], ICR: toBN(dec(266, 16)), extraLUSDAmount: B_totalDebt, extraParams: { from: alice } })

    const collAddress = collaterals[2].address;
    const collDecimals = await contracts.collateralConfig.getCollateralDecimals(collAddress)
    const collMCR = await contracts.collateralConfig.getCollateralMCR(collAddress)

    // Alice deposits LUSD in the Stability Pool
    await stabilityPool.provideToSP(B_totalDebt, { from: alice })

    // --- TEST ---
    // price drops to 1YAMv2:100LUSD, reducing TCR below 165%
    await priceFeed.setPrice(collAddress, '100000000000000000000')
    let price = await priceFeed.getPrice(collAddress)
    const TCR = await th.getTCR(contracts, collAddress)

    const recoveryMode = await th.checkRecoveryMode(contracts, collAddress)
    assert.isTrue(recoveryMode)

    // Check Bob's ICR is between MCR and TCR
    const bob_ICR = await troveManager.getCurrentICR(bob, collAddress, price)
    assert.isTrue(bob_ICR.gt(coll0MCR) && bob_ICR.lt(TCR))

    // Liquidate Bob
    await troveManager.liquidate(bob, collAddress, { from: owner })

    /* check Bob's collateral surplus

    entireTroveDebt =     582400000000000000000 (18 decimals) = 582,4 LUSD
    entireTroveColl = 7280000000000000000000000 (24 decimals) = 7,28 YAMv2
    price           =     100000000000000000000 (18 decimals) = 100 LUSD/YAMv2
    collMCR         =       1200000000000000000 (18 decimals) = 1.2

    Without taking into account decimals:
    cappedCollPortion = entireTroveDebt * MCR / price
    collateralSurplus = entireTroveColl - cappedCollPortion

    => collateralSurplus = entireTroveColl - entireTroveDebt * MCR / price
                         = 7,28 - 582,4 * 1.2 / 100
                         = 7,28 - 698,88 / 100
                         = 7,28 - 6,9888
                         = 0,2912
                         => apply 24 decimals =>
                         = 291200000000000000000000
    */

    const normalisedPrice = th.unwrapCollNativeDec(price, collDecimals)
    const bob_scaledCollateral = B_totalDebt.mul(collMCR).div(price)

    const bob_remainingCollateral = B_coll.sub(th.unwrapCollNativeDec(bob_scaledCollateral, collDecimals))

    const userSurplusCollateral = await collSurplusPool.getUserCollateral(bob, collAddress);
    th.assertIsApproximatelyEqual(userSurplusCollateral, bob_remainingCollateral)

    assert.equal("291200000000000000000000", userSurplusCollateral.toString())
  })

  it("liquidate(), with MCR < ICR < TCR, digits(collateral) < 18, collateral surplus corect calculation", async () => {
    // --- SETUP ---
    // Alice withdraws up to 1500 LUSD of debt, resulting in ICRs of 266%.
    // Bob withdraws up to 480 LUSD of debt, resulting in ICR of 250%. Bob has lowest ICR.
    const { collateral: B_coll, totalDebt: B_totalDebt } = await openTrove({ collateral: collaterals[0], ICR: toBN(dec(250, 16)), extraLUSDAmount: dec(480, 18), extraParams: { from: bob } })
    const { collateral: A_coll } = await openTrove({ collateral: collaterals[0], ICR: toBN(dec(266, 16)), extraLUSDAmount: B_totalDebt, extraParams: { from: alice } })

    const collAddress = collaterals[0].address;
    const collDecimals = await contracts.collateralConfig.getCollateralDecimals(collAddress)
    const collMCR = await contracts.collateralConfig.getCollateralMCR(collAddress)

    // Alice deposits LUSD in the Stability Pool
    await stabilityPool.provideToSP(B_totalDebt, { from: alice })

    // --- TEST ---
    // price drops to 1YAMv2:100LUSD, reducing TCR below 165%
    await priceFeed.setPrice(collAddress, '100000000000000000000')
    let price = await priceFeed.getPrice(collAddress)
    const TCR = await th.getTCR(contracts, collAddress)

    const recoveryMode = await th.checkRecoveryMode(contracts, collAddress)
    assert.isTrue(recoveryMode)

    // Check Bob's ICR is between MCR and TCR
    const bob_ICR = await troveManager.getCurrentICR(bob, collAddress, price)
    assert.isTrue(bob_ICR.gt(coll0MCR) && bob_ICR.lt(TCR))

    // Liquidate Bob
    await troveManager.liquidate(bob, collAddress, { from: owner })

    /* check Bob's collateral surplus

    entireTroveDebt =     582400000000000000000 (18 decimals) = 582,4 LUSD
    entireTroveColl =               72800000000 (12 decimals) = 7,28 wETH
    price           =     100000000000000000000 (18 decimals) = 100 LUSD/YAMv2
    collMCR         =       1200000000000000000 (18 decimals) = 1.2

    Without taking into account decimals:
    cappedCollPortion = entireTroveDebt * MCR / price
    collateralSurplus = entireTroveColl - cappedCollPortion

    => collateralSurplus = entireTroveColl - entireTroveDebt * MCR / price
    => collateralSurplus = entireTroveColl - entireTroveDebt * MCR / price
                         = 7,28 - 582,4 * 1.2 / 100
                         = 7,28 - 698,88 / 100
                         = 7,28 - 6,9888
                         = 0,2912
                         => apply 14 decimals =>
                         = 291200000000

    */

    const normalisedPrice = th.unwrapCollNativeDec(price, collDecimals)
    const bob_scaledCollateral = B_totalDebt.mul(collMCR).div(price)

    const bob_remainingCollateral = B_coll.sub(th.unwrapCollNativeDec(bob_scaledCollateral, collDecimals))

    const userSurplusCollateral = await collSurplusPool.getUserCollateral(bob, collAddress);
    th.assertIsApproximatelyEqual(userSurplusCollateral, bob_remainingCollateral)

    assert.equal("291200000000", userSurplusCollateral.toString())
  })
```

#

### [NC-03] Use "type(uint256).max" instead of "uint256(-1)"

##### Description

In code, there is a usage of `uint256(-1)` instead of a more readable version "type(uint256).max"
It is also supported for the file `ActivePool.sol` which uses the solidity version 0.6.11 (added in 0.6.8)

Reference https://docs.soliditylang.org/en/v0.6.8/types.html#integers

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258
```Solidity
        vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
```

##### Recommendation

Change uint256(-1)" to "type(uint256).max" 

#

### [NC-04] Inconsistent event emission syntax used

##### Description

In [`ActivePool.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol) there are both old style and new style event emission syntax:

Be consistent and use the newer style format, that uses the `emit` keyword.

##### Instances (2)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L194
```Solidity
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L201
```Solidity
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
```

##### Recommendation

Use the newer style format, that uses the `emit` keyword.

#

### [NC-05] Incomplete require error messages

##### Description

`_requireCallerIsBOorTroveMorSP` from `ActivePool.sol` checks that the sender is one of 4 allowed addresses but if the sender is not, will use a revert message indicating only 3 of them.
It is missing the RedemptionHelper: `"ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"`

`_requireCallerIsTroveManagerOrActivePool` from `LQTYStaking.sol` checks that the sender is one of 3 allowed addresses but if the sender is not, will use a revert message indicating only 2 of them.
It is missing the RedemptionHelper: `"LQTYStaking: caller is not TroveM or ActivePool"`

##### Instances (2)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L327-L335
```Solidity
    function _requireCallerIsBOorTroveMorSP() internal view {
        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
        require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251-L259
```Solidity
    function _requireCallerIsTroveManagerOrActivePool() internal view {
        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
        require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool"
        );
    }
```

##### Recommendation

Add `" nor RedemptionHelper"` to the end of the require error messages.

#

### [NC-06] Unused code

##### Description

There are instances of code that is simply not used. Either this is an indication of unfinished logic or forgoten dead code that servers no purpose

##### Instances (1)

In the function `_getSingularCollateralGain` from `StabilityPool.sol` the `vars.collDecimals` variable is not used
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L664
```Solidity
        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

##### Recommendation

Remove the code or implement the expected logic.

#

### [NC-07] Missing events for critical parameter changes

##### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes

##### Instances (3)

In [CommunityIssuance.sol](`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol`) `distributionPeriod`, which is used in the calculation of the issued OATH tokens has no event when it is set.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L58
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L121

In [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol) `setHarvestSteps`, which sets the steps touple list, the steps added/changed should be broadcasted

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L170

In [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol) `updateTreasury`, which sets up the new treasury, the change should be broadcasted

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L630


##### Recommendation
Add events in the indicated locations

#

### [NC-08] `_requireValidMaxFeePercentage` can be rewritten to optimize calculations

##### Description

Function `_requireValidMaxFeePercentage` from `BorrowerOperations.sol` does the operation `_maxFeePercentage <= DECIMAL_PRECISION` 2 times instead of once. 
Can be optimized.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648-L656
```Solidity
    function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {
        if (_isRecoveryMode) {
            require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must less than or equal to 100%");
        } else {
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
        }
    }
```

##### Recommendation

Remove the duplicated calculation. Example implementation:

```Solidity
 function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure { 
        require(_maxFeePercentage <= DECIMAL_PRECISION,
            "Max fee percentage must less than or equal to 100%");
        
        if (!_isRecoveryMode) {
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR,
                "Max fee percentage must be between 0.5% and 100%");
        }
 }
```
#

### [NC-09] `_cascadingAccessRoles` can be pure instead of view

##### Description

Function `_cascadingAccessRoles` state mutability can be restricted to pure. 

##### Instances (2)
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203-L211
```Solidity
    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
        cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;
        cascadingAccessRoles[1] = ADMIN;
        cascadingAccessRoles[2] = GUARDIAN;
        cascadingAccessRoles[3] = STRATEGIST;
        cascadingAccessRoles[4] = KEEPER;
        return cascadingAccessRoles;
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L659-L667
```Solidity
    function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
        cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;
        cascadingAccessRoles[1] = ADMIN;
        cascadingAccessRoles[2] = GUARDIAN;
        cascadingAccessRoles[3] = STRATEGIST;
        cascadingAccessRoles[4] = DEPOSITOR;
        return cascadingAccessRoles;
    }
```


##### Recommendation

Change `view` to `pure`.

#

### [NC-10] Dead Comments

##### Description

In ReaperVaultV2.sol in `_deposit` the comment `// use "freeFunds" instead of "pool"` seems to be forgotten there, as it is already implemented in the same line.

##### Instances (1)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334
```Solidity
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
```

##### Recommendation

Remove dead comments.

#

### [NC-11] Inconsistent shares calculation across code base

##### Description

In `ReaperVaultV2.sol` the amount of shares is sometimes required to be calculated relative to a specific amount.
There are 2 different implementations that do the same thing. Consider combining them, or alt least have the same implementation.

##### Instances (2)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L331-L335
```Solidity
        if (totalSupply() == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
        }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L465-L466
```Solidity
            uint256 supply = totalSupply();
            uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();
```

##### Recommendation

Consider combining them into a separate function or alt least have the same consistent implementation version.

#

### [NC-12] Open TODOs

##### Description

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment
2 TODOs are already mentioned in the C4 report but the third is not.

##### Instances (3)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380
```Solidity
        /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
```

```Solidity
        /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
         * Doesn't make a lot of sense to include in multiple CollateralGainWithdrawn logs.
         * If needed could create a separate event just to report this.
         */
        uint LUSDLoss = initialDeposit.sub(compoundedLUSDDeposit); // Needed only for event log
```
This second case also includes an unused variable `LUSDLoss`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L759-L767
```Solidity
        /*
        * If compounded deposit is less than a billionth of the initial deposit, return 0.
        *
        * NOTE: originally, this line was in place to stop rounding errors making the deposit too large. However, the error
        * corrections should ensure the error in P "favors the Pool", i.e. any given compounded deposit should slightly less
        * than it's theoretical value.
        *
        * Thus it's unclear whether this line is still really needed.
        */
```

Third case is a TODO by it's content.

##### Recommendation

Resolve them and eliminate the comments.

#

### [NC-13] Already declared variable not reused
##### Description

In [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L246) the local variable `vars.yieldGenerator` is set in order to be used as `vars.yieldGenerator = IERC4626(yieldGenerator[_collateral]);`.
The next 2 lines it is used but further down, the casting/setting is again, executed `IERC4626(yieldGenerator[_collateral])` when `vars.yieldGenerator` could simply be reused.

##### Instances (2)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L280
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L282

##### Recommendation

Modify the code to reuse the already initialized `vars.yieldGenerator` variable.

#

### [NC-14] Functions do not respect camelCase naming convention
##### Description

In [BorrowerOperations.sol](`https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol`) there are 2 functions that do not respect the camelCase naming convention.

##### Instances (2)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L541
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L546

##### Recommendation

Rename the functions to respect proper standards.

#