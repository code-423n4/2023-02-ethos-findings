
### [L01] require() should be used instead of assert()


#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::128 => assert(MIN_NET_DEBT > 0);
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::197 => assert(vars.compositeDebt > 0);
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::301 => assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::331 => assert(_collWithdrawal <= vars.coll);
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::312 => assert(sender != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::313 => assert(recipient != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::321 => assert(account != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::329 => assert(account != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::337 => assert(owner != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::338 => assert(spender != address(0));
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::526 => assert(_debtToOffset <= _totalLUSDDeposits);
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::551 => assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::591 => assert(newP > 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::417 => assert(_LUSDInStabPool != 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1224 => assert(totalStakesSnapshot[_collateral] > 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1279 => assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1342 => assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1348 => assert(index <= idxLast);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1414 => assert(newBaseRate > 0); // Base rate is always non-zero after redemption
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1489 => assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```



### [L02] `initialize` functions can be front-run

#### Impact
See [this](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/40) finding from a prior badger-dao contest for details
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::46 => function initialize(
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::62 => function initialize(
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::67 => ) public initializer {
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [L03] `safeApprove()` is deprecated

#### Impact
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of safeIncreaseAllowance() and safeDecreaseAllowance()
#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::74 => IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);
```



### [L04] `address.call{value:x}()` should be used instead of `.transfer()`

#### Impact
The use of `.transfer()` is heavily frowned upon because it can lead to the locking of funds. The transfer() call requires that the recipient has a payable callback, only provides 2300 gas for its operation. 
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::127 => OathToken.transfer(_account, _OathAmount);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::135 => lusdToken.transfer(msg.sender, LUSDGain);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::171 => lusdToken.transfer(msg.sender, LUSDGain);
```



### [L05] `_safeMint()` should be used rather than `_mint()` wherever possible

#### Impact
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::188 => _mint(_account, _amount);
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::336 => _mint(_receiver, shares);
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::467 => _mint(treasury, shares);
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [L06] Return values of `transfer()`/`transferFrom()` not checked

#### Impact
Not all IERC20 implementations revert() when there’s a failure in transfer()/transferFrom(). The function signature has a boolean
 return value and they indicate errors that way instead. By not checking
 the return value, operations that should have been marked as failed may 
potentially go through without actually making a payment
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::103 => OathToken.transferFrom(msg.sender, address(this), amount);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::127 => OathToken.transfer(_account, _OathAmount);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::135 => lusdToken.transfer(msg.sender, LUSDGain);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::171 => lusdToken.transfer(msg.sender, LUSDGain);
```


### [L07] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::3 => pragma solidity ^0.8.0;
```


### Non-Critical Issues



### [N01] constants should be defined rather than using magic numbers


#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::133 => require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::145 => require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::433 => uint usdValue = _price.mul(_coll).div(10**_collDecimals);
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::125 => lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::296 => return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N02] Use a more recent version of solidity

#### Impact
Use a solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>)
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::3 => pragma solidity 0.6.11;
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N03] Variable names that consist of all capital letters should be reserved for `const`/`immutable `variables

#### Impact
If the variable needs to be different based on which class it comes from, a view/pure function should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::170 => uint256 internal totalLUSDDeposits;
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N04] Use a more recent version of solidity

#### Impact
Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::27 => using SafeMath for uint256;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::28 => using SafeERC20 for IERC20;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::19 => using SafeERC20 for IERC20;
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::16 => using SafeERC20 for IERC20;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::15 => using SafeMath for uint;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::19 => using SafeERC20 for IERC20;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::20 => using SafeMath for uint;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::29 => using SafeMath for uint256;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::147 => using LiquitySafeMath128 for uint128;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::148 => using SafeERC20 for IERC20;
```



### [N05] Open TODOs

#### Impact
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::335 => /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::380 => /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
```


### [N06] Numeric values having to do with time should use time units for readability

#### Impact
There are [units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units) for seconds, minutes, hours, days, and weeks
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::58 => distributionPeriod = 14 days;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::125 => lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::24 => uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::25 => uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N07] Constant redefined elsewhere

#### Impact
Consider defining in only one contract so that values cannot become out of sync when only one location is updated
#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::76 => bytes32 public constant ADMIN = keccak256("ADMIN");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::49 => bytes32 public constant KEEPER = keccak256("KEEPER");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::50 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::51 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::52 => bytes32 public constant ADMIN = keccak256("ADMIN");
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N08] NC-library/interface files should use fixed compiler versions, not floating ones


#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::3 => pragma solidity ^0.8.0;
```


### [N09] now is deprecated

#### Impact
Use `block.timestamp` instead
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::282 => require(deadline >= now, 'LUSD: expired deadline');
```


### [N10] initialize function lacks address(0) checks

#### Impact
Zero address check should be used in the initialize function to prevent setting smth as mistake to address(0).

#### Findings:
```
2023-02-ethos-Core/Ethos-Core/contracts/CollateralConfig.sol::47 => address[] calldata _collaterals,
```

### Consider adding zero address check like this:

  `require(_collaterals != address(0), "Address Zero");`





### [N11] Constructor lacks address(0) check

#### Impact
Zero-address check should be used in the constructors, to avoid the risk of setting a storage variable as address(0) at deploying time.

#### Findings:
```
2023-02-ethos-Vault/Ethos-Core/contracts/ReaperVaultERC4626.sol::16 => constructor(
2023-02-ethos-Vault/Ethos-Core/contracts/ReaperVaultV2.sol::111 => constructor(
```