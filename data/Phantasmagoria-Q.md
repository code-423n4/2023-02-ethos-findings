# Low Risk Issues
|       | Issue | Instances     |
| :---              |    :----             |         :----:  |
| 1     | Deprecated safeApprove() function     | 1      |
| 2     |   _safeMint() should be used rather than _mint() wherever possible   | 2    |

Total: 3 instances

# Non-Critical Issues
|       | Issue | Instances     |
| :---              |    :----             |         :----:  |
| 1     | Imports can be grouped together     | 3      |
| 2     |  Constant redefined elsewhere  |  2   |
| 3     | Named imports can be used    | 3      |
| 4     |   Remove console.log import   | 5  |
| 5     | Interchangeable usage of uint and uint256   | 3      |
| 6     |   Add a limit for the maximum number of characters per line   | 1    |
| 7     | Can be rewritten in many lines    | 2      |
| 8     |  NatSpec comments should be increased in contracts  | 2    |
| 9     | Commented code    | 1      |
| 10     |  Unused events  | 2    |
| 11    | Proper use of _ as a function name prefix   | 1      |
| 12     |   Use named returns for local variables where it is possible   | 22    |
| 13     | No need to use return    | 4      |
| 14     |   Code is not properly formatted  | 5    |
| 15     | Use _ with big numbers     | 1      |

Total: 57 instances

### [L-01] Deprecated safeApprove() function
Deprecated in favor of safeIncreaseAllowance() and safeDecreaseAllowance(). If only setting the initial allowance to the value that means infinite, safeIncreaseAllowance() can be used instead
```
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

74: IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74

### [L-02] _safeMint() should be used rather than _mint() wherever possible
_mint() is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function
```
Ethos-Vault/contracts/ReaperVaultV2.sol

336: _mint(_receiver, shares);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L336
```
Ethos-Vault/contracts/ReaperVaultV2.sol

467: _mint(treasury, shares);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L467

### [NC-1] Imports can be grouped together
Consider importing OZ first, then all interfaces, then all utils
```
Ethos-Vault/contracts/ReaperVaultV2.sol

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

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L5-L14
```
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

5: import "../interfaces/IStrategy.sol";
6: import "../interfaces/IVault.sol";
7: import "../libraries/ReaperMathUtils.sol";
8: import "../mixins/ReaperAccessControl.sol";
9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L5-L12
```
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

5: import "./abstract/ReaperBaseStrategyv4.sol";
6: import "./interfaces/IAToken.sol";
7: import "./interfaces/IAaveProtocolDataProvider.sol";
8: import "./interfaces/ILendingPool.sol";
9: import "./interfaces/ILendingPoolAddressesProvider.sol";
10: import "./interfaces/IRewardsController.sol";
11: import "./libraries/ReaperMathUtils.sol";
12: import "./mixins/VeloSolidMixin.sol";
13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L5-L14

### [NC-2] Constant redefined elsewhere
Consider defining in only one contract so that values cannot become out of sync when only one location is updated.
A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.
```
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

50: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52: bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50-L52
```
Ethos-Vault/contracts/ReaperVaultV2.sol

74: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76: bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74-L76

### [NC-3] Named imports can be used
It’s possible to name the imports to improve code readability. 
```
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

13: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
14: import "@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L13-L14
```
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

9: import "@openzeppelin/contracts-upgradeable/access/AccessControlEnumerableUpgradeable.sol";
10: import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
11: import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
12: import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L9-L12
```
Ethos-Vault/contracts/ReaperVaultV2.sol

9: import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
10: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
13: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
14: import "@openzeppelin/contracts/utils/math/Math.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L9-L14

### [NC-4] Remove console.log import
```
Ethos-Core/contracts/BorrowerOperations.sol

15: import "./Dependencies/console.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L15
```
Ethos-Core/contracts/ActivePool.sol

15: import "./Dependencies/console.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L15
```
Ethos-Core/contracts/StabilityPool.sol

18: import "./Dependencies/console.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L18
```
Ethos-Core/contracts/LQTY/LQTYStaking.sol

9: import "./Dependencies/console.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L9
```
Ethos-Core/contracts/LUSDToken.sol

9: import "./Dependencies/console.sol";
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9

### [NC-5] Interchangeable usage of uint and uint256
Consider using only one approach throughout the codebase, e.g. only uint or only uint256.
```
Ethos-Core/contracts/BorrowerOperations.sol

557: uint _price,
558: uint256 _CCR,
559: uint256 _collateralDecimals
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L557-L559
```
Ethos-Core/contracts/BorrowerOperations.sol

663: uint _coll,
664: uint _debt,
665: uint _collChange,

667: uint _debtChange,

669: uint256 _collateralDecimals
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L663-L669
```
Ethos-Core/contracts/TroveManager.sol

362: uint _ICR,
363: uint _LUSDInStabPool,
364: uint _TCR,
365: uint _price,
366: uint256 _MCR
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L362-L366

### [NC-6] Add a limit for the maximum number of characters per line
The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters. Consider adding a limit of 120 characters or less to prevent large lines.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L934

### [NC-7] Can be rewritten in many lines
To increase readability consider to rewrite following functions like this
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268
before:
```
Ethos-Core/contracts/BorrowerOperations.sol

268: function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {}
```
after:
```
function adjustTrove(
    address _collateral,
    uint _maxFeePercentage,
    uint _collTopUp,
    uint _collWithdrawal,
    uint _LUSDChange,
    bool _isDebtIncrease,
    address _upperHint,
    address _lowerHint
    ) 
	external 
    {}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282
before:
```
Ethos-Core/contracts/BorrowerOperations.sol

282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {}
```
after:
```
function _adjustTrove(
    address _borrower, 
    address _collateral, 
    uint _collTopUp, 
    uint _collWithdrawal, 
    uint _LUSDChange, 
    bool _isDebtIncrease, 
    address _upperHint, 
    address _lowerHint, 
    uint _maxFeePercentage
    ) 
      internal 
    {}
```

### [NC-8] NatSpec comments should be increased in contracts
Consider adding NATSPEC on all public/external functions to improve documentation
```
Ethos-Core/contracts/ActivePool.sol

144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144
```
Ethos-Core/contracts/ActivePool.sol

138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138

### [NC-9] Commented code
Remove this comment because it's already not needed
```
Ethos-Core/contracts/TroveManager.sol

1413: //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1413

### [NC-10] Unused events
These events are not used anywhere else
```
Ethos-Core/contracts/StabilityPool.sol

240: event DefaultPoolAddressChanged(address _newDefaultPoolAddress);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L240
```
Ethos-Core/contracts/LQTY/LQTYStaking.sol

50: event LUSDTokenAddressSet(address _lusdTokenAddress);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L50

### [NC-11] Proper use of _ as a function name prefix
It is a common pattern is to prefix internal and private function names with _ 
```
Ethos-Vault/contracts/ReaperVaultERC4626.sol

269: function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256)
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L269

### [NC-12] Use named returns for local variables where it is possible
Consider using local variables, like in the examples below
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432
```
Ethos-Core/contracts/BorrowerOperations.sol

432: function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint){
433:    uint usdValue = _price.mul(_coll).div(10**_collDecimals);
434:
435:    return usdValue;
436: }
```
fix:
```
function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint usdValue) {
    uint usdValue = _price.mul(_coll).div(10**_collDecimals);
}
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455
```
Ethos-Core/contracts/BorrowerOperations.sol

455: function _updateTroveFromAdjustment
456:    (
457:         ITroveManager _troveManager,
458:         address _borrower,
459:         address _collateral,
460:         uint _collChange,
461:         bool _isCollIncrease,
462:         uint _debtChange,
463:         bool _isDebtIncrease
464:     )
465:         internal
466:         returns (uint, uint)
467:     {
468:         uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)
469:                                         : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);
470:         uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)
471:                                         : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);
472: 
473:        return (newColl, newDebt);
474:    }
```
fix:
```
function _updateTroveFromAdjustment
    (
        ITroveManager _troveManager,
        address _borrower,
        address _collateral,
        uint _collChange,
        bool _isCollIncrease,
        uint _debtChange,
        bool _isDebtIncrease
    )
        internal
        returns (uint newColl, uint newDebt)
{
    uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)
                                        : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);
    uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)
                                        : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);
}
```
Other instances:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L678
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L700
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L721
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L745
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1050
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1063
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1073
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1134
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1149
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1209
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1451
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1571
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1578
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1585
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1592
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L456
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L690
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L709
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L725
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L217

### [NC-13] No need to use return 
Do not use return at the end of the function
```
Ethos-Core/contracts/TroveManager.sol

353: return singleLiquidation;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L353
```
Ethos-Core/contracts/TroveManager.sol

436: return singleLiquidation;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L436
```
Ethos-Core/contracts/TroveManager.sol

1332: return index;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1332
```
Ethos-Core/contracts/StabilityPool.sol

543: return (collGainPerUnitStaked, LUSDLossPerUnitStaked);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L543

### [NC-14] Code is not properly formatted
Use only " or ' throughout codebase. Some instances:
```
Ethos-Core/contracts/StabilityPool.sol

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
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5-L14
```
Ethos-Core/contracts/StabilityPool.sol

870: require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L870
```
Ethos-Core/contracts/StabilityPool.sol

874: require(_amount > 0, 'StabilityPool: Amount must be non-zero');
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L874
Unneeded space in require statements
```
Ethos-Core/contracts/BorrowerOperations.sol

633: require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L633
```
Ethos-Core/contracts/TroveManager.sol

1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539

### [NC-15] Use _ with big numbers
Consider using underscores for number values to improve readability. 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41
before:
```
Ethos-Vault/contracts/ReaperVaultV2.sol

41: uint256 public constant PERCENT_DIVISOR = 10000;
```
after:
```
uint256 public constant PERCENT_DIVISOR = 10_000;
```