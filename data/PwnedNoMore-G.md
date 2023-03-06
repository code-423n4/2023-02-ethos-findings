### TroveManager._computeNewStake()
Read totalStakesSnapshot[_collateral] twice.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1225

### TroveManager._computeNewStake()
Reading totalCollateralSnapshot[_collateral] twice.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1215
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1225

### ReaperVaultV2._withdraw()
Could assign with 'strategyBal - actualWithdrawn' instead of -=.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395

### ReaperVaultV2.updateTvlCap()
Emitting a storage value instead of memory one.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581

### ReaperVaultV2.addStrategy()
Prefer not to read/write the state variable `totalAllocBPS` for more than once.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156

