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

### Recommended Mitigation Steps

We recommend defining constants for the numbers used throughout the code.

# 4: FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGE

Vulnerability details

## Context: 

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces to allow us to apply this rule better.


Multiple results - 12 files

src/CollateralConfig.sol

5: import "./Dependencies/CheckContract.sol";
6: import "./Dependencies/Ownable.sol";
7: import "./Dependencies/SafeERC20.sol";
8: import "./Interfaces/ICollateralConfig.sol";

src/BorrowerOperations.sol

5: import "./Interfaces/IBorrowerOperations.sol";
6: import "./Interfaces/ICollateralConfig.sol";
7: import "./Interfaces/ITroveManager.sol";
8: import "./Interfaces/ILUSDToken.sol";
9: import "./Interfaces/ICollSurplusPool.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Dependencies/LiquityBase.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";

src/TroveManager.sol

5: import "./Interfaces/ICollateralConfig.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Interfaces/IStabilityPool.sol";
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILUSDToken.sol";
10: import "./Interfaces/ISortedTroves.sol";
11: import "./Interfaces/ILQTYStaking.sol";
12: import "./Interfaces/IRedemptionHelper.sol";
13: import "./Dependencies/LiquityBase.sol";
14: 
15: import "./Dependencies/CheckContract.sol";
16: import "./Dependencies/IERC20.sol";


src/ActivePool.sol

5: import './Interfaces/IActivePool.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IDefaultPool.sol';
8: import "./Interfaces/ICollSurplusPool.sol";
9: import "./Interfaces/ILQTYStaking.sol";
10: import "./Interfaces/IStabilityPool.sol";
11: import "./Interfaces/ITroveManager.sol";
12: import "./Dependencies/SafeMath.sol";
13: import "./Dependencies/Ownable.sol";
14: import "./Dependencies/CheckContract.sol";
15: import "./Dependencies/console.sol";
16: import "./Dependencies/SafeERC20.sol";
17: import "./Dependencies/IERC4626.sol";


src/StabilityPool.sol

5: import './Interfaces/IBorrowerOperations.sol';
6: import "./Interfaces/ICollateralConfig.sol";
7: import './Interfaces/IStabilityPool.sol';
8: import './Interfaces/IBorrowerOperations.sol';
9: import './Interfaces/ITroveManager.sol';
10: import './Interfaces/ILUSDToken.sol';
11: import './Interfaces/ISortedTroves.sol';
12: import "./Interfaces/ICommunityIssuance.sol";
13: import "./Dependencies/LiquityBase.sol";
14: import "./Dependencies/SafeMath.sol";
15: import "./Dependencies/LiquitySafeMath128.sol";
16: import "./Dependencies/Ownable.sol";
17: import "./Dependencies/CheckContract.sol";
18: import "./Dependencies/console.sol";
19: import "./Dependencies/SafeERC20.sol";


src/CommunityIssuance.sol 

5: import "../Dependencies/IERC20.sol";
6: import "../Interfaces/ICommunityIssuance.sol";
7: import "../Dependencies/BaseMath.sol";
8: import "../Dependencies/LiquityMath.sol";
9: import "../Dependencies/Ownable.sol";
10: import "../Dependencies/CheckContract.sol";
11: import "../Dependencies/SafeMath.sol";


src/LQTYStaking.sol 

5: import "../Dependencies/BaseMath.sol";
6: import "../Dependencies/SafeMath.sol";
7: import "../Dependencies/Ownable.sol";
8: import "../Dependencies/CheckContract.sol";
9: import "../Dependencies/console.sol";
10: import "../Dependencies/IERC20.sol";
11: import "../Interfaces/ICollateralConfig.sol";
12: import "../Interfaces/ILQTYStaking.sol";
13: import "../Interfaces/ITroveManager.sol";
14: import "../Dependencies/LiquityMath.sol";
15: import "../Interfaces/ILUSDToken.sol";
16: import "../Dependencies/SafeERC20.sol";


src/LUSDToken.sol

5: import "./Interfaces/ILUSDToken.sol";
6: import "./Interfaces/ITroveManager.sol";
7: import "./Dependencies/SafeMath.sol";
8: import "./Dependencies/CheckContract.sol";
9: import "./Dependencies/console.sol";


src/ReaperVaultV2.sol

5: import "./interfaces/IERC4626Events.sol";
6: import "./interfaces/IStrategy.sol";
7: import "./libraries/ReaperMathUtils.sol";
8: import "./mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
14: import "@openzeppelin/contracts/utils/math/Math.sol";


src/ReaperVaultERC4626.sol

5: import "./ReaperVaultV2.sol";
6: import "./interfaces/IERC4626Functions.sol";


src/ReaperBaseStrategyv4.sol

5: import "../interfaces/IStrategy.sol";
6: import "../interfaces/IVault.sol";
7: import "../libraries/ReaperMathUtils.sol";
8: import "../mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";


src/ReaperStrategyGranarySupplyOnly.sol 

5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
10: import "./interfaces/ILendingPoolAddressesProvider.sol";
11: import "./interfaces/IRewardsController.sol";
12: import "./libraries/ReaperMathUtils.sol";
13: import "./mixins/VeloSolidMixin.sol";
14: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
15: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

import {contract1 , contract2} from "filename.sol";

A good example from the ArtGobblers project;

import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";


# 5: SHOWING THE ACTUAL VALUES OF NUMBERS IN NATSPEC COMMENTS MAKES CHECKING AND READING CODE EASIER

Vulnerability details

## Context:

File: contracts/abstract/ReaperBaseStrategyv4.sol

\- 25: int256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

\+ 25: int256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100; // 31_536_000 ( 365 * 24 * 60 * 60) * 100

File: contracts/LQTY/CommunityIssuance.sol

\- 58: distributionPeriod = 14 days;

\+ 58: distributionPeriod = 14 days; // 1_209_600 ( 14 * 24 * 60 * 60)

File: contracts/StabilityPool.sol

 \- 197: uint public constant SCALE_FACTOR = 1e9; 

 \+ 197: uint public constant SCALE_FACTOR = 1e9; // 1,000,000,000

## Proof of Concept

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L25 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L58 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L197 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use NATSPEC comments where necessary

# 6: USE UNDERSCORES FOR NUMBER LITERALS

Vulnerability details

## Context:

There are occasions where certain numbers have been hardcoded, either in variables or in the code itself. Large numbers can become hard to read.

1 result - 1 file 

src/ReaperVaultV2.sol

41: uint256 public constant PERCENT_DIVISOR = 10000;

## Proof of Concept

 https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider using underscores for number literals to improve their readability.

# 7: MISSING NATSPEC

Vulnerability details

## Context:

Consider adding NATSPEC comments on all public/external functions to improve documentation.

## Proof of Concept


> ***File: LUSDToken.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L133 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L141 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L160 


> ***File: LQTYStaking.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66 


> ***File: CommunityIssuance.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61 

> ***File: StabilityPool.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L261 

> ***File: ActivePool.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L71 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L125 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L132 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L138 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L144 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L197 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L204 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L214 

> ***File: TroveManager.sol***

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L232 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L962 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L982 


### Tools Used

Manual Analysis


# 8: USE .CALL INSTEAD OF .TRANSFER TO SEND ETHER

Vulnerability details

## Impact:

.transfer will relay 2300 gas and .call will relay all the gas.

## Proof of Concept

> ***File: contracts/LQTY/LQTYStaking.sol***  

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135 

> ***File: contracts/LQTY/CommunityIssuance.sol*** 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Replace .transfer with .call. Note that the result of .call needs to be checked.

# 9: LACK OF CHECKS-EFFECTS-INTERACTIONS

Vulnerability details

## Context:

It’s recommended to execute external calls after state changes, to prevent reentrancy bugs.

Consider moving the external calls after the state changes in the following instances:


## Proof of Concept


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L328-L335

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L210-L22 


### Tools Used

Manual Analysis


