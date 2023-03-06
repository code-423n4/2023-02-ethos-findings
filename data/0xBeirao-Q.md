## Low

### [L-1] Unsafe transfer when transfering OATH

Consider `using SafeERC20 for IERC20;` In the `CommunityIssuance` contract. User may provide or withdraw from the stability pool and never receive their OATH reward. 

[CommunityIssuance.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127)

## Non-Critical

[N-1] [moveCollateralGainToTrove()](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L248-L251) is never used please remove

[N-2] `_scaleTellorPriceByDigits` in PriceFeed is useless since TARGET_DIGITS = TELLOR_DIGITS = 18 [PriceFeed L521](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/PriceFeed.sol#L521-L529) just return the input price.

[N-3] `emit` keyword is needed when emitting events [ActivePool.sol#L194,](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L194) [ActivePool.sol#L201](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L201)

[N-4] **[yieldSplitSP](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L53)** is not used in `StabilityPool` please remove

[N-5] The correct error description for ReaperVaultV2 [L528](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155) and [L155](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155) is : ”Fee cannot be higher than 20 %”

[N-6] Cache 10_000 BPS in a constant variable in all Ethos-Core contracts

[N-7] TODOs and commented code should be removed for readability

[N-8] A `_LUSDAmount != 0` check can be added in **[openTrove()](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L172)** to improve error handling