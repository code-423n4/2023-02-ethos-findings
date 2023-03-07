# [L-01] Add checking CCR > MCR

The invariant CCR > MCR should always hold. If the CCR is set smaller than MCR, the logic of the protocol will be broken. It also helps avoid mistake placing CCR and MCR in the wrong order when setting.

https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/CollateralConfig.sol#L85-L100

## Recommended Mitigation Steps: 

Add checking: 

        require(_CCR > _MCR, "CCR below MCR")

# [L-02] Avoid checking if an address is a contract by using size := extcodesize(_account)

The code size(extcodesize) of contract is currently 0 when contract is under creation and hasn't been deployed. The malicious could bypass extcodesize check put the function in the constructor.

## Recommended Mitigation Steps: 

As best practice, when checking if an address is a contract using: `require(tx.origin != msg.sender)`

https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/Dependencies/CheckContract.sol#L11-L18

For more please check here: 
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/extcodesize-checks/


# [L-03] The function revokeStrategy requires the GUARDIAN role level, but it can be bypassed by the STRATEGIST role.

Only an account with the GUARDIAN role or higher can revoke the strategy and set `strategy.allocBPS` = 0
https://github.com/code-423n4/2023-02-ethos/blob/1dc72b8185baba7dfd697aec41190028b41b686d/Ethos-Vault/contracts/ReaperVaultV2.sol#L205-L217

However, an account with the STRATEGIST role can change it and set `strategy.allocBPS` to a different value.
https://github.com/code-423n4/2023-02-ethos/blob/1dc72b8185baba7dfd697aec41190028b41b686d/Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L199

This means that the function `revokeStrategy` can be bypassed.

## Recommended Mitigation Steps: 

An additional field `active` can be added to `StrategyParams`. Only an account with the GUARDIAN role or higher can toggle the active field to true or false.

# [L-04] Unspecific compiler version pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an different/outdated compiler version that might introduce bugs that affect the contract system negatively.

## Example Instance:
https://github.com/code-423n4/2023-02-ethos/blob/1dc72b8185baba7dfd697aec41190028b41b686d/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/1dc72b8185baba7dfd697aec41190028b41b686d/Ethos-Vault/contracts/ReaperVaultV2.sol#L3



# [NC-01] Missing Events on State Changing Functions and critical functions:
Events are used by off-chain participants to track on-chain state changes. There are several functions that don't emit events:

Instance: 

When `treasuryAddress` and `lqtyStakingAddress` is updated.
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L102-L103


# [NC-02] Missing the emit keywords in event emitting
When emit events, the emit keywords are missing:

Instance:
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L194
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L201


# [NC-03] Using type(uint256).max instead of uint256(-1) for improving code readability.

Instance:
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L258

# [NC-04] Remove dead comments

Instance:
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L14
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L18
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L19
https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L26