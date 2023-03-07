# 1. Use mixedCase

It is recommended to use the mixedCase naming convention for improving the readability and consistency of source code.

This convention involves combining words with no spaces, with the first letter of each word capitalized except for the first word.

For more information, see the Solidity style guide at the following link:

- [Naming Convention For Local and State Variable Names](https://docs.soliditylang.org/en/v0.5.3/style-guide.html#local-and-state-variable-names)

Affected lines of code:

- [TroveManager.sol#L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L86)

- [TroveManager.sol#L116](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L116)

- [TroveManager.sol#L119-L120](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L119-L120)

# 2. Unspecific Compiler Version Pragma

It is generally not recommended to use floating pragmas (i.e. pragmas that do not specify a specific compiler version) in contracts that are not intended to be used as libraries.

This is because using floating pragmas in application contracts can pose a security risk.

For example, a known vulnerable compiler version may be selected by mistake, or security tools might revert to an older compiler version that produces a different EVM compilation than the one intended to be deployed on the blockchain.

To avoid these potential issues, consider specifying a specific compiler version in your pragmas.

So instead of using a floating pragma like `pragma solidity ^0.8.0;`, it is better to use a concrete compiler version like `pragma solidity 0.8.17;`.

More information can be found in the following links:

- [Consensys Audit of 1inch](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)
- [Solidity docs](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#version-pragma)
- [Solidity Specific](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

Affected line of code:

- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
- [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
- [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
- [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

# 3. Use newer versions of solidity

Consider using the latest version of solidity rather than version `0.6.11` as newer versions have bug fixes, as well as new features.

The latest versions provide things like SafeMaths (built into version `0.8.0` and above.), `using for`(`0.8.13` and above.), `string.concat()` instead of `abi.encodePacked()`(`0.8.12` and above.), and `bytes.concat()` instead of `abi.encodePacked()`(`0.8.4` and above.)

Affected line of code:

- [CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)
- [LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)
- [CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3)
- [BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3)
- [StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3)
- [LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3)
- [ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3)
- [TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)

# 4. Lines too long

In general, it is a good practice to keep lines of source code within 80 characters in length.
Although, some flexibility is allowed and it is reasonable to let lines be up to 120 characters in some instances.

On modern screens, it is even possible to go beyond this limit.
However, it is recommended to split lines when they reach a length of 164 characters or more, as this is the point at which GitHub will introduce a scroll bar to view the code.

This can help to make the code more readable and easier to work with.

Affected line of code:

- [BorrowerOperations.sol#L172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172)
- [BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)
- [BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282)
- [BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)
- [BorrowerOperations.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511)

# 5. Improve the Readability by following modularity principles

To make your Solidity code more readable, it is recommended to update your import statements to only include the specific contracts or objects that you need.

This practice, known as modular programming, helps to keep the code cleaner, maintainable and more organized.

To avoid unnecessarily polluting the source code with unused contracts or objects, it is recommended to use specific imports with curly braces. An example of this syntax is shown below:

```solidity
import {contract1, contract2} from "filename.sol";
```

Affected line of code:

- [CollateralConfig.sol#L5-L8](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L5-L8)

# 6. Fix typos to improve readability

```solidity
File: Ethos-Core/contracts/TroveManager.sol
Line  936:    * The debt recorded on the trove's struct is zero'd elswhere, in _closeTrove.

...

Line 1155:    * this indicates that rewards have occured since the snapshot was made, and the user therefore has
```

Consider making the following changes to [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L936):

- `elswhere` To `elsewhere` ([Line 936](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L936))
- `occured` To `occurred` ([Line 1155](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1155))
