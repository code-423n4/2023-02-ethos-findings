## Summary

| Number | Issues Details                                          |
| ------ | ------------------------------------------------------- |
| [L-01] | Outdated compiler version                               |
| [N-01] | Lock `pragma` to specific compiler version              |
| [N-02] | Assert statements should not be used                    |
| [N-03] | Using vulnerable dependency of OpenZeppelin             |
| [N-04] | For modern and more readable code; update import usages |

## [L-01] Outdated compiler version

Very old compiler version is used inside the contracts of Ethos-Core. There are a lot of major improvements and bug fixes in the new compiler versions which the protocol will be missing on.

**Context**
5 results - 5 files

    Ethos-Core/contracts/ActivePool.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LUSDToken.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/StabilityPool.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
    3: pragma solidity 0.6.11;

    Ethos-Core/contracts/LQTY/LQTYStaking.sol:
    3: pragma solidity 0.6.11;

**Recommendation:**

Use a more recent version of the compiler. The latest compiler version available is 0.8.18.

## [N-01] Lock `pragma` to specific compiler version

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. All of the contracts inside Ethos-Vault which are in scope use floatable pragma.

**Recommendation:**
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.

[solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

## [N-02] Assert statements should not be used

Properly functioning code should never reach a failing assert statement. If it happened, it would indicate the presence of a bug in the contract. Assert statements are used mainly for testing.

**Context**

20 results - 4 files

    Ethos-Core/contracts/BorrowerOperations.sol:
    128:         assert(MIN_NET_DEBT > 0);
    197:         assert(vars.compositeDebt > 0);
    301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
    331:         assert(_collWithdrawal <= vars.coll);

    Ethos-Core/contracts/LUSDToken.sol:
    312:         assert(sender != address(0));
    313:         assert(recipient != address(0));
    321:         assert(account != address(0));
    329:         assert(account != address(0));
    337:         assert(owner != address(0));
    338:         assert(spender != address(0));

    Ethos-Core/contracts/StabilityPool.sol:
    526:         assert(_debtToOffset <= _totalLUSDDeposits);
    551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
    591:         assert(newP > 0);

    Ethos-Core/contracts/TroveManager.sol:
    417:             assert(_LUSDInStabPool != 0);
    1224:             assert(totalStakesSnapshot[_collateral] > 0);
    1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
    1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
    1348:         assert(index <= idxLast);
    1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
    1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

**Recommendation:**

Replace the assert statements with a require statement or a custom error.

## [N-03] Using vulnerable dependency of OpenZeppelin

An old version of the OpenZeppelin contracts is used. OpenZeppelin contracts are constantly upgraded to fix bugs and potential vulnerabilities.

    "dependencies": {
        "@openzeppelin/contracts": "^4.7.3",
        "@openzeppelin/contracts-upgradeable": "^4.7.3",
        "dotenv": "^16.0.1"
    }

**Recommendation**
Use patched versions
Latest non-vulnerable version 4.8.2.

## [N-04] For modern and more readable code; update import usages

    100 results - 11 files

    Ethos-Core/contracts/ActivePool.sol:
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

    Ethos-Core/contracts/BorrowerOperations.sol:
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

    Ethos-Core/contracts/CollateralConfig.sol:
    5: import "./Dependencies/CheckContract.sol";
    6: import "./Dependencies/Ownable.sol";
    7: import "./Dependencies/SafeERC20.sol";
    8: import "./Interfaces/ICollateralConfig.sol";

    Ethos-Core/contracts/LUSDToken.sol:
    5: import "./Interfaces/ILUSDToken.sol";
    6: import "./Interfaces/ITroveManager.sol";
    7: import "./Dependencies/SafeMath.sol";
    8: import "./Dependencies/CheckContract.sol";
    9: import "./Dependencies/console.sol";

    Ethos-Core/contracts/StabilityPool.sol:
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

    Ethos-Core/contracts/TroveManager.sol:
    5: import "./Interfaces/ICollateralConfig.sol";
    6: import "./Interfaces/ITroveManager.sol";
    7: import "./Interfaces/IStabilityPool.sol";
    8: import "./Interfaces/ICollSurplusPool.sol";
    9: import "./Interfaces/ILUSDToken.sol";
    10: import "./Interfaces/ISortedTroves.sol";
    11: import "./Interfaces/ILQTYStaking.sol";
    12: import "./Interfaces/IRedemptionHelper.sol";
    13: import "./Dependencies/LiquityBase.sol";
    14: // import "./Dependencies/Ownable.sol";
    15: import "./Dependencies/CheckContract.sol";
    16: import "./Dependencies/IERC20.sol";

    Ethos-Core/contracts/LQTY/CommunityIssuance.sol:
    5: import "../Dependencies/IERC20.sol";
    6: import "../Interfaces/ICommunityIssuance.sol";
    7: import "../Dependencies/BaseMath.sol";
    8: import "../Dependencies/LiquityMath.sol";
    9: import "../Dependencies/Ownable.sol";
    10: import "../Dependencies/CheckContract.sol";
    11: import "../Dependencies/SafeMath.sol";

    Ethos-Core/contracts/LQTY/LQTYStaking.sol:
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

    Ethos-Vault/contracts/ReaperVaultERC4626.sol:
    5: import "./ReaperVaultV2.sol";
    6: import "./interfaces/IERC4626Functions.sol";

    Ethos-Vault/contracts/ReaperVaultV2.sol:
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

    Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol:
    5: import "../interfaces/IStrategy.sol";
    6: import "../interfaces/IVault.sol";
    7: import "../libraries/ReaperMathUtils.sol";
    8: import "../mixins/ReaperAccessControl.sol";
    9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
    10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
    11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
    12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";

**Description**

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation**

`import {contract1 , contract2} from "filename.sol";`

A good example from the ArtGobblers project;

```solidity
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```
