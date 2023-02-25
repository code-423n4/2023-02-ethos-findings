# CHECK TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L335
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380

# `decimals()` NOT PART OF `ERC20` STANDARD
`decimals()` is not part of the official `ERC20` standard and could potentially fail for tokens that do not implement it. 
Unlikely in practice, but something to keep in mind as it could be potentially an issue.
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L651

# IMMUTABLE ADDRESSES LACK ZERO-ADDRESS CHECK
Checking addresses against zero-address during initialization or during setting is a security best-practice. 
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L120

# NatSpec COMMENTS SHOULD BE INCREASED IN CONTRACTS
It's good practice to make sure Solidity contracts are fully annotated using NatSpec. Especially, in complex DeFi projects, clear comments about all functions, their arguments and returns is important for readability and auditability.

# STAKING CONTRACT `LQTYStaking.sol` SHOULD HAVE PAUSE/UNPAUSE FUNCTIONALITY
In case a hack occurs or an exploit is discovered, the team should be able to pause functionality until the necessary changes are made to the system. The deposits should be paused with Pause modifier.

# UNUSED VARIABLES
## Instances

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L240
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50

# UNUSED IMPORTS
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L8

# DUPLICATED IMPORTS
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8

# UNUSED `console.log` IMPORT
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L15
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L15
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTYStaking.sol#L9

# NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
## Instances

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3

# GROUP RELATED DATA INTO SEPARATE STRUCTS
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L44-L47

Having three separate fields that map from `_collateral` is error-prone.
Replace: 
```
    mapping (address => uint256) public yieldingPercentage;
    mapping (address => uint256) public yieldingAmount;
    mapping (address => address) public yieldGenerator;
    mapping (address => uint256) public yieldClaimThreshold;
```
With: 

```
struct Yield {
	uint256 percentage;
	uint256 amount;
	address generator;
	uint256 claimThreshold;
}
```
# POSSIBLE DIVIDE-BY-ZERO ERROR
Consider adding a check on `freeFunds` before division.
## Instances
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334
