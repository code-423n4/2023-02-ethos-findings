# [01] Floating Pragma

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

# [02] Use two-step procedure for critical address changes

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L146
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L153

# [03] Missing emit-keywords for events

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L194
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L201

# [04] Consider using named imports

Updating imoprts improves readability: E.g. `import "@openzeppelin/contracts/token/ERC20/IERC20.sol";` can be rewritten as `import {IERC20} from “import “@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol”;`

All Files

# [05] Omission in events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters such as old values in addition to the new ones.

Events with no old value:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L92-L102
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L186-L198
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58-L61
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L236-L244
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L50
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L52
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49-L54
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L78-L82

# [06] Remove unused imports

- BorrowerOperations.sol: console.sol https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L15
- StabilityPool.sol: console.sol https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18
- LUSDToken.sol: console.sol https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9
- ActivePool.sol: console.sol https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L15

# [07] Owner can renounce ownership

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L15
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L26
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14

Description: Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

onlyOwner functions:

CollateralConfig: 
updateCollateralRatios(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contract/CollateralConfig.sol#L85-L89

ActivePool:
setYieldingPercentage(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125
setYieldingPercentageDrift(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132
setYieldClaimThreshold(,,,) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138
setYieldDistributionParams(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144

CommunityIssuance:
setAddresses(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L69
fund(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101
updateDistributionPeriod(...) https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120

I recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

# [08] Use internal pure consistently

BorrowerOperations: function _getNewICRFromTroveChange(...) is marked pure internal whereas the rest of the internal and pure functions are marked internal pure

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L693-L694

# [09] Contract layout and function order

Context: All Contracts

According to the solidity style guide it is recommended to declare state variables and structs before all functions. Consider moving the variables below to the top of the contracts:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L220-L237

Another recommendation is to declare internal functions bellow external/public functions and to declare view and pure functions at the end of a grouping.

The instances bellow highlightes public below internal:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715

internal above external:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282

pure/view functions not at the end of a grouping:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432

# [10] Add a limit for characters on one line to prevent long lines

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters.

Consider adding a limit of 120 characters or less to prevent large lines.

Just a few examples from the codebase:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L348
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L407

# [11] Disable the optimizer in order to prevent bugs 

The Optimizer is being acitively developed and is not considered 100% safe. Here is an audit that advises against using it: https://github.com/trailofbits/publications/blob/master/reviews/SeaportProtocol.pdf

# [12] NatSpec comments should be increased in contracts

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts

# [14] open TODOs 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L335
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380

# [15] remove unnecessary comments in TroveManager 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L14
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L18
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L19

These comments are not necessary and someone forgot to remove them.

# [16] set visibility of initialize function to external

There is a couple of reasons as to why you may want to to that:
1. If someone wants to extend via inheritance, it might make more sense that the overridden initialize(…) function calls the internal {…}_init function, not the parent public initialize(…) function.
2. External instead of public would give more the sense of the initialize(…) functions to behave like a constructor (only called on deployment, so should only be called externally)
3. From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract. 
4. It might cost a bit less gas to use external over public.
5. It is possible to override a function from external to public 
but it is not possible to override a function from public to external

For above reasons you can change initialize(…) to external
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L67
