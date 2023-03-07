# G1. `require` or `revert` should be used instead of `assert`

Sanity checks of functions arguments can be through `require` or `revert` instead of `assert` because later takes up all the gas while checking for the conditions, but the former will return the gas if the condition fails. 

## List
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L311
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L320
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L328
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L336

## Recommendation
Change all `assert` condition checks with `require` or `revert`.

# G2. No need to explicitly initialize variables with default values

Default values (`0` for `uint`, `false` for `bool`, `address(0)` for `address`) are assumed if a variable is not set or initialized. Explicitly initializing it with its default value is wastes of gas and not a good practice.

# List of instances

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L37
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L32
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L18
Also, in most of the `for` loops

## Recommendation

Don't remove initialize the variables with default values to save gas. 

