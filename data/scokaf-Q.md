# 1: LOCK PRAGMAS TO SPECIFIC COMPILER VERSION

Vulnerability details

## Context:

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case of contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

For reference, see https://swcregistry.io/docs/SWC-103

## Proof of Concept

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L26 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3 

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler versions.
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

# 2: GENERATE PERFECT CODE HEADERS EVERY TIME


## Description:

We recommend using headers for Solidity code layout and readability

For reference, see https://github.com/transmissions11/headers 

## Proof of Concept

/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


# 3: USE CONSTANTS FOR NUMBERS

Vulnerability details

## Impact:

In several locations in the code, numbers like 1e9 are used. The same goes for values like type(uint256).max. It is quite easy to make a mistake somewhere, also when comparing values.

## Proof of Concept

File: contracts/abstract/ReaperBaseStrategyv4.sol

73: IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74 

File: contracts/ReaperVaultERC4626.sol

81: if (tvlCap == type(uint256).max) return type(uint256).max;

124: if (tvlCap == type(uint256).max) return type(uint256).max;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L81 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L124 


File: contracts/ReaperVaultV2.sol

588: updateTvlCap(type(uint256).max);

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L588 

File: contracts/StabilityPool.sol

129: uint public constant SCALE_FACTOR = 1e9;

768: if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;}

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L197 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L768 

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

We recommend defining constants for the numbers used throughout the code.

