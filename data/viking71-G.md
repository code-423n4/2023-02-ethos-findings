# 1. Don't use floating pragma

Locking the version of solidity will avoid previous version solidity bugs. Also it is best to test the smart contracts by setting all to same compiler version. Deploying the thoroughly tested solidity version to mainnet is suggested. 

https://swcregistry.io/docs/SWC-103

## List
All contracts in `Ethos-Vault` (use ^0.8.0)

## Recommendation 

Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

# 2. Outdated solidity compiler version

All contracts in `Ethos-Core` use 0.6.11 version can be problematic especially if there are publicly disclosed bugs and issues that affect the current compiler version. Using recent versions will keep bugs away. 

https://swcregistry.io/docs/SWC-102

## Recommendation

It is recommended to use a recent version of the Solidity compiler.

# 3. Remove unused imports

Unused imports/variables are no security issues but removing them will be best practice and will increase the readability. 

https://swcregistry.io/docs/SWC-131

## List

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L15
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L9
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L18
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L15
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9

## Recommendation

Remove all unused imports from the code base.

