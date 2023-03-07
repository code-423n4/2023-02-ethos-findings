# [GAS Optimization]

## [G-1] Structs can be closely packed 
There are 5 instances of it total gas saving around ~10500
1st instance is at [CollateralConfig.sol#L28-L29](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L28-L29) here instead of ` uint256 decimals;` can be changed to ` uint1288 decimals;` it will be more than enough for the decimals and it can pack with `bool allowed;` can save 1 slot (2100 gas can be saved here)
2nd instance at [BorrowerOperations.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L50) here `uint256 collDecimals;` can be changed to `uint128 collDecimals;` because collDecimals lies between 6 to 18 as confirmed by team and can easily pack it with `bool isCollIncrease;` to save 1 storage slot (2100 gas can be saved here)
3rd instance is at [TroveManager.sol#L130](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L130) here also `uint256 collDecimals;` can be changed to `uint128 collDecimals;` because collDecimals lies between 6 to 18 and can easily pack it with `bool recoveryModeAtStart;` can save 1 storage slot (2100 gas can be saved here)
4th instance is at [TroveManager.sol#L155](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L155) here also `uint256 collDecimals;` can be changed to `uint128 collDecimals;` because collDecimals lies between 6 to 18 and can easily pack it with `bool backToNormalMode;` & `address user;` can save 1 storage slot (2100 gas can be saved here)
5th instance is [ReaperVaultV2.sol#L26](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L26) here `uint256 activation;` can be changed to `uint128 activation;` can be more than enough since it is a timestamp & [ReaperVaultV2.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L32) here `uint256 lastReport;` can be changed to `uint128 lastReport;` since it is using timestamp it will be suffient and both can packed thus result in saving 1 storage slot (2100 gas can be saved here) 

## [G-2] Fixed bytes instead of strings
There are 6 instances of it here we are focusing only on constants which cant change later 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L32-L34

## [G-3] State variables can be closely packed 
There is 1 instance of it 
[ActivePool.sol#L49-L54](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L49-L54) here all the state varaiables of uint256 can be changed to uint128 for saving 2 storage slots 
since all those state variables can fit under uint128

## [G-4] Splitting of `require()` statements that uses `&&` saves some gas
There are 2 instances of it
[LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L347-L351) 
& [LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L352-L357)
its good to use multiple require statements to save some gas


