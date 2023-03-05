# Failed to get pending rewards if the collateral's decimal is 0

In TroveManager.sol, on lines 1132 and 1147, `div(10**collDecimals)` is used for calculation. However, if the collateral token has 0 decimal places (which is allowed in ERC20), the transaction will be reverted and the reward will not be received.

```
function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {
        ...
        uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

        return pendingCollateralReward;
    }
```


Even if the collateral token is on the whitelist, due to the contracts being non-upgradable contracts, if we want to support tokens with decimal 0, we will need to redeploy the contract, which will cause a bad user experience.