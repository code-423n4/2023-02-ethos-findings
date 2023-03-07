Hello,

Thanks in advance for your work.

Here is my Gas Report:

1. Increase Solidity's optimizer runs
Increasing Solidity's optimizer runs can lead to reduced gas costs in a smart contract. To optimize gas costs for contract deployment, it is recommended to set the Solidity optimizer at a lower number. On the other hand, if you want to optimize for run-time gas costs when functions are called on a contract, you should set the optimizer to a higher number.

To optimize gas costs and improve contract performance, it is suggested to set the optimization value higher than 800.

2. Emit a single event with a set config.

Instead of emitting every value separately in `setAddresses` single event can be emitted

Code places for optimization:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L155
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L115
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L293
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L79

3. Not needed usage of `SafeERC20` in `CollateralConfig.sol`.

`SafeERC20` here https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L16 is not needed, because in the file we use `IERC20` interface only to get decimals(), so we can skip the usage of `SafeERC20`, and save bytecode space.

4. Change state after the checks

State here https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L95 should be changed after `require()`, so the user won't pay for setting state if the next require condition will fail.

5. Check could be inlined

The following checks could be inlined so there will be no need to store them in the local variable:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L543
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L548
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L324

6. Flag mintedPause should be moved after guardianAddress.

If we move mintedPause https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37 to line 72, it will be stored with the guardianAddress taking only a single storage slot.

7. Use uintMax256 constant instead of type(uint256).max.

Using `type(uint120).max` results in higher gas consumption compared to using constants.

Places for optimization:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L588
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L81

Thanks again for checking my work.

Kind regards,
Paweł

