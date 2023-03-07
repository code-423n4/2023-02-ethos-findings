Hello,

Thanks in advance for your work.

Here is my QA Report:

1. Interfaces are not used in all necessary places instead of the address argument.
 
For type safety, interfaces should be used. When a function takes a contract address as an argument, it is better to pass an interface or contract type rather than a raw address. If the function is called elsewhere within the source code, the compiler will provide additional type safety guarantees.

List of places where the interface should be used:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L783
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71 (IVault for _erc4626vaults)
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L171
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111 (IERC20Metadata for _token)
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L637
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L58 (IVault should be used or it should be name vaultAddress for consistency with the rest of code, I would opt for interface). Then here interface should be passed https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63

Recommended solution: 
Use interfaces instead of addresses as function parameters.

2. Improvement of import usage

The Point struct, which was previously imported globally in Solidity code, may have cluttered the source code with an unused object. This violates the principles of modularity and modular programming, which suggest that we should only import what we need. To address this issue, we can use specific imports with curly braces to better adhere to this principle.

All the files in the scope are affected by this.

Recommended solution: 
To implement specific imports, we can use the following syntax:

`import {IActivePool} from './Interfaces/IActivePool.sol';`

This allows us to import only the necessary contracts and avoid cluttering the code with unnecessary objects like the Point struct.

3. Inconsistent formatting
In some places, additional space is added. Examples:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L849
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1127
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1142

Recommended solution: 
Format everything with a linter.

4. No events for ReaperBaseStrategyv4.sol, and ReaperVaultV2.sol

Recommended solution: 
Add events for crucial actions like withdrawal, and deposit, so interested parties can subscribe to them

5. Use underscore for all functions arguments for consistency.

In general, across the code underscore is used for argument, but there are places where this pattern is not followed. Examples:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L432
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462

Recommended solution: 
Use underscores for all functions arguments for consistency

6. Use a consistent pattern for using `setAddresses`, across different smart contracts.

In different `setAddresses` functions implementation inside ActivePool.sol, StabilityPool.sol, TroveManager.sol, BorrowerOperations.sol, CollateralConfig.sol etc. different ways of blocking this function to be called twice are used. It is done through: Ownable contract + `_renounceOwnership()`, setting `owner` directly to `address(0)`, setting flag `initialized`, setting flag `addressesSet`.

Recommended solution: 
Choose a consistent pattern for blocking calling twice `setAddresses`.

7. No emitted events on Trove manager close trove event

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1278 (There is no event for closing trove)
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562 (set trove status)
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1567
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1574
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1581
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1588

Recommended solution: 
Emit state-changing events in the Trove manager, so parties can subscribe to those.

8. Enhance readability, it is suggested to use underscores for number literals.

Example: https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41

Recommended solution: 
Use `10_000`.

9. Use a message with require
In some places require doesn't contain an error message

Examples:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716
These are just a few examples, there are more occurrences across the codebase.

Recommended solution: 
Use error messages with require to improve error handling. 

10. Use require instead of the assert

Examples:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
These are just a few examples, there are more occurrences across the codebase.

Using the assert() function in Solidity code, except for testing purposes, can lead to consuming all available gas instead of returning it. Prior to Solidity 0.8.0, this behavior was different from request()/revert().

Assert() and require():
There is a significant difference between the two functions. When false, assert() uses up all remaining gas and reverts all changes made, whereas require() also reverts changes but refunds remaining gas fees offered to pay. The require() function is the most commonly used for debugging and error handling.

Even after Solidity version 0.8.0, it is recommended to avoid using assert() due to its documentation stating that "The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. There's a mistake.

Recommendation:

To avoid consuming all available gas and handle errors more efficiently, use require() instead of assert() except for testing purposes.

11. Use a timelock to critical functions

When implementing critical changes in a project, it is advisable to allow users sufficient time to respond and adapt. One way to accomplish this is by using a timelock mechanism, which enhances security and reduces the level of trust needed from users. Therefore, it is recommended to consider adding a timelock to:
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L160
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L146
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L153

12. Use Solidity Style Guide for function writing 

Functions don't follow
https://docs.soliditylang.org/en/v0.8.17/style-guide.html

They should be ordered as follow: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private (within a grouping, place the view and pure functions last)


That is all. Thanks again for checking my work.

Kind regards,
Paweł






