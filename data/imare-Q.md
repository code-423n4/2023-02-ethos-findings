## **QA-01**: No min/max checking when setting yield claim trashold field
When setting the claim treshold field that is used inside rebalancing there is no verification for the field to trigger any profit distribution.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L138-L142

Other yield parameters are checked for their correct working.

## **QA-02**: require inside `_requireCallerIsBOorTroveMorSP` error is missing the redemptionHelper explanation

When the require error is triggered its explanation is missing to report that also `redemptionHelper` can be the caller of this function:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L329-L334

## **QA-03**: maxMint can revert

This method by definition MUST NOT REVERT but if `tvlCap` is lower then the current `balance()` the version of solidity used will revert the call.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125

A check before the calculation must be done to prevent reverting the call.

Note: `tvlCap` value can be arbitrary changed by the `admin` user by calling `updateTvlCap` method.

## **QA-04**: `ReaperVaultV2#updateTvlCap` doesn't properly validate the newTvlCap

There is no prevention of setting the total value locked cap to be lower then the current balance.

## **QA-05**: inside `ReaperBaseStrategyv4#__ReaperBaseStrategy_init` multisigRoles is not verified that role is indeed assigned

The intention of role assignments is not verified and will silently fail when called with an empty or smaller `_multisigRoles` array

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L83-L85

## **QA-06**: `ReaperBaseStrategyv4#initiateUpgradeCooldown()` should emit an event
There should be an event emitted when calling this function to signal users to prepare for imminent protocol upgrade

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L167-L170

## **QA-07**: collateral can not be removed
Once the collateral is added with `CollateralConfig#initialize` it cannot be removed. There is no functionality that addresses the removal of collateral.

## **QA-08**: TellorCaller seams to be deprecated in favor of TellorFlex contract

On the main page of [Tellor](https://docs.tellor.io/tellor/the-basics/contracts-overview) for the oracle there is only mention for the TellorFlex and is not the one that is currently in use for `Ethos` protocol.

On optimism network the currently used/deployed [TellorCaller](https://optimistic.etherscan.io/address/0x0358AA3dc3D7101a601d66974e8b84d449C1C4bD) seams to be used only by the Ethos protocol and is not supported by the original Tellor developer.

## **QA-09**: `yieldClaimThreshold` in `ActivePool#rebalance` is not taking in account token directly sent to the contract
When profit is finally detected because of the yield generator the token sent directly to the contract are finally taken into account.

A better solution would be to take this token in account earlier by checking the difference in `collAmount` and the balance of the contract for the same collateral.

``` diff
vars.profit = vars.sharesToAssets.sub(vars.currentAllocated);
+int256 diff = IERC20(_collateral).balanceOf(address(this)) - collAmount[_collateral];
+
+if (diff > 0) {
+    vars.profit = vars.profit.add(diff);
+}
+
if (vars.profit < yieldClaimThreshold[_collateral]) {
    vars.profit = 0;
}
```

## **QA-10**: Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

File : https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L17

Note: inhering contract of this abstract contract is also missing the __gap field variable.
