Unused variables `troveManagerAddress` in LUSDToken should be removed.

The global variable is defined at https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L67, then it is assigned values in the `constructor()` and `upgradeProtocol`, but its value is never read.

Hence, it is better to be removed to save gas.