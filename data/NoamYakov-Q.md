## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Wrong emission of events | 3 |
| [L&#x2011;02] | Privileged addresses can accidentally set system parameters with illegal values, leading to denial-of-service (and potential gas drainage) | 2 |


## Low Risk Issues

### [L&#x2011;01]  Wrong emission of events
Events are emitted when they aren't supposed to.

*There are 3 instances of this issue:*

[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L94)
```solidity
94          emit TotalOATHIssuedUpdated(totalOATHIssued);
```

The function `CommunityIssuance.issueOath()` doesn't update the total OATH issued unless `lastIssuanceTimestamp < lastDistributionTime`. This event should only be emitted if `lastIssuanceTimestamp < lastDistributionTime`.

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L94)
```solidity
94          emit LQTYTokenAddressSet(_lusdTokenAddress);
```

The function `LQTYStaking.setAddresses()` doesn't set the LQTY token address `_lusdTokenAddress`. It sets it to `_lqtyTokenAddress` - and the function indeed emits the `LQTYTokenAddressSet(_lqtyTokenAddress)` event.

This function also sets the LUSD token address to `_lusdTokenAddress`, but doesn't emit the corresponding `LUSDTokenAddressSet(_lusdTokenAddress)` event.

[Ethos-Core\contracts\LQTY\LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L131)
```solidity
131         emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);
```

The function `LQTYStaking.stake()` doesn't withdraw staking gains unless `currentStake != 0`. This event should only be emitted if `currentStake != 0`.

### [L&#x2011;02]  Privileged addresses can accidentally set system parameters with illegal values, leading to denial-of-service (and potential gas drainage)
Privileged addresses can accidentally set system parameters with illegal values. This may lead cause several external functions to constantly revert on legitimate operations (denial-of-service). If the revert is made by the INVALID opcode (see [Panic exception](https://docs.soliditylang.org/en/latest/control-structures.html#panic-via-assert-and-error-via-require)), all the remaining gas is consumed - and the users end up paying large amounts of gas for nothing. Although the impact is severe, the likelihood is small - so I consider it low-risk.

*There are 2 instances of this issue:*

[Ethos-Core\contracts\ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L144-L150)
```solidity
144     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
145         require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
146         yieldSplitTreasury = _treasurySplit;
147         yieldSplitSP = _SPSplit;
148         yieldSplitStaking = _stakingSplit;
149         emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
150     }
```

The function `ActivePool.setYieldDistributionParams()` doesn't correctly make sure the that the splits add up to 10,000 BPS. The requirement can still pass if there's an overflow. In this case, if `yieldSplitTreasury` plus `yieldSplitStaking` is greater than 10,000, then the function `ActivePool._rebalance()` will revert on [line 302](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L302) because of an underflow.


[Ethos-Core\contracts\LQTY\CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122)
```solidity
120     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121         distributionPeriod = _newDistributionPeriod;
122     }
```

The function `CommunityIssuance.updateDistributionPeriod()` doesn't make sure the that the distribution period isn't zero. If `distributionPeriod` is zero, then the function `CommunityIssuance.fund()` will revert on [line 112](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112) because of a division by zero.
