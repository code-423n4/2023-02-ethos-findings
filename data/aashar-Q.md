### Low Severity

1. Use named imports instead of ``.
2. Unused state variables:
    1. [BorrowerOperations.sol#L145](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L145)
    2. [StabilityPool.sol#L284](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L284)
    3. [TroveManager.sol#L277](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L277)
    4. [TroveManager.sol#L276](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L276)
    5. [StabilityPool.sol#L160](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L160)
    6. [StabilityPool.sol#L339](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L339)
    7. [StabilityPool.sol#L384](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L384)
3. `[LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L282)` uses a deprecated function - `now.`Consider using `block.timestamp` instead
4. The Solidity version of `LUSDToken.sol` and `CommunityIssuance.sol` is 0.6.11. Consider using the latest version.
5. Interchangeable usage of `uint` and `uint256` in `CommunityIssuance.sol`.
6. Double imports of `IBorrowerOperations` in `StabilityPool.sol`.
7. Unused imports:
    1. [StabilityPool.sol#L5](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L5).
