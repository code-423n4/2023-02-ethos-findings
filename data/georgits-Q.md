## Use latest Solidity version with a stable pragma statement
[ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol), [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol), [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol), [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

## Remove unused imports
LUSDToken.sol - [6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L6)

CommunityIssuance.sol - [8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L8)

StabilityPool.sol - [14](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L14)

## Use two-step role transfer pattern
LUSDToken.sol - [146](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L146), [153](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L153)

## Wrong event emitted
LQTYStaking.sol - [94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L94)

 Should be `LUSDTokenAddressSet` instead of `LQTYTokenAddressSet`

## Missing arrays length check
LQTYStaking.sol - [238](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L238)

## Add `uint constant` equal to `10_000` instead of reusing `10_000`
ActivePool.sol - [127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127), [145](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L145), [258](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258), [262](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L262), [291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L291), [296](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L296)

##  `IBorrowerOperations` interface is imported twice
StabilityPool.sol - [5](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5), [8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8)