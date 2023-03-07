`Dear Ethos Team, as we have gone through each contract within the scope, we have noticed good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing. We understand that every team has a different level of good practices, but we believe that at least 90% of the recommendations in the following report should be applied for better gas efficiency, readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future projects.`

### Issues 
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 34 |
|:--:|:--:|


### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | USER FACING BORROWEROPERATIONS AND TROVEMANAGER MISS EMERGENCY LEVER | 2 |
| [L-02] | MULTIPLE ADDRESS MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS TO A STRUCT, WHERE APPROPRIATE | 19 |
| [L-03] | PRAGMA EXPERIMENTAL ABIECODERV2 IS DEPRECATED | 2 |
| [L-04] | ERC20UPGRADEABLE SAFEAPPROVE() IS DEPRECATED | 1 |
| [L-05] | Open TODOs | 2 |
| [L-06] | DIVISION BEFORE MULTIPLICATION CAN LEAD TO PRECISION ERRORS | 2 |
| [L-07] | IMMUTABLE ADDRESSES LACK ZERO-ADDRESS CHECK | 1 |
| [L-08] | USE OF FLOATING PRAGMA | 17 |
| [L-09] | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE | 3 |

| Total Low Risk Issues | 9 | Total Instances | 49 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | TOO MANY BITS TO DESCRIBE SMALL QUANTITIES | MULTIPLE INSTANCES |
| [N-02] | MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS | 1 |
| [N-03] | RACE CONDITION ON ERC20 APPROVAL | 1 |
| [N-04] | CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES | 12 |
| [N-05] | LONG REVERT STRINGS   | 8 |
| [N-06] | USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18) | 6 |
| [N-07] | EVENTS IS MISSING INDEXED FIELDS | 75 |
| [N-08] | MISSING NATSPEC | 7 |
| [N-09] | MISSING ERROR MESSAGES IN REQUIRE STATEMENTS | 5 |
| [N-10] | USE CUSTOM ERRORS INSTEAD OF REVERT STRINGS TO SAVE GAS | 92 |
| [N-11] | USE A MORE RECENT VERSION OF SOLIDITY | 12 |
| [N-12] | LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR | 3 |
| [N-13] | LINES ARE TOO LONG | MULTIPLE INSTANCES |
| [N-14] | DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES | 32 |

| Total Non-Critical Issues | 14 | Total Instances | 253 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | NUMERIC VALUES HAVING TO DO WITH TIME SHOULD USE TIME UNITS FOR READABILITY | 2 |
| [R-02] | USE REQUIRE INSTEAD OF ASSERT | 20 |
| [R-03] | SOME NUMBER VALUES CAN BE REFACTORED WITH _ | 1 |
| [R-04] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 102 |
| [R-05] | REVERT SHOULD BE USED ON SOME FUNCTIONS INSTEAD OF RETURN | 9 |
| [R-06] | MISSING IMMUTABLE KEYWORD | 1 |
| [R-07] | INSTEAD OF USING "&&" IN REQUIRE, JUST USE REQUIRE MULTIPLE TIMES | 5 |

| Total Refactor Issues | 7 | Total Instances | 140 |
|:--:|:--:|:--:|--:|

### Ordinary Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | FUNCTION NAMING SUGGESTIONS | 1 |
| [O-02] | PROPER USE OF "GET" AS A FUNCTION NAME PREFIX | 23 |
| [O-03] | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS | 11 |
| [O-04] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO) | 20 |

| Total Ordinary Issues | 5 | Total Instances | 56 |
|:--:|:--:|:--:|--:|

# Detailed Findings

## [L-01] USER FACING BORROWEROPERATIONS AND TROVEMANAGER MISS EMERGENCY LEVER

If there be any emergency with system contracts or outside infrastructure, there is no way to temporary stop the operations.

### PROOF OF CONCEPT
- [Borrower Operations Contract](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
- [Trove Manager Contract](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

### MITIGATION
Make user facing contracts, first of all BorrowerOperations and TroveManager, pausable.

For example, by using OpenZeppelin's approach:
[Pausable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol)

It will be prudent to perform a deeper check to draw an extensive list of endpoints that require to be paused if something unexpected happens.


## [L-02] MULTIPLE ADDRESS MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/catellaTech/725eb34b6b12ebb39d44f1b80578f9e5) (Gkeccak256 - **30 gas**) and that calculation’s associated stack operations.

### PROOF OF CONCEPT

```solidity
Ethos-Core/contracts/ActivePool.sol

41: mapping (address => uint256) internal collAmount; 
42: mapping (address => uint256) internal LUSDDebt;  
44: mapping (address => uint256) public yieldingPercentage; 
45: mapping (address => uint256) public yieldingAmount; 
46: mapping (address => address) public yieldGenerator;
47: mapping (address => uint256) public yieldClaimThreshold; 
```

`Ethos-Core/contracts/TroveManager.sol`
- [Lines of code in TroveManager](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L88-L105)
- [Lines of code in TroveManager](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L119-L120)


`Ethos-Core/contracts/StabilityPool.sol`
- [Lines of code in StabityPool](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L167)
- [Lines of code in StabilityPool](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L228)


`Ethos-Core/contracts/LQTY/LQTYStaking.sol`
- [Lines of code in LQTYStaking](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L28)


`Ethos-Core/contracts/LUSDToken.sol`
- [Lines of code in LUSDToken](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L54-L57)
- [Lines of code in LUSDToken](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62-L64)


### MITIGATION

We recommend following your own patterns, as in this case: 
- [Solution code in CollateralConfig](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L35)


## [L-03] | PRAGMA EXPERIMENTAL ABIECODERV2 IS DEPRECATED 
Use `pragma abicoder v2` [instead](https://github.com/ethereum/solidity/blob/69411436139acf5dbcfc5828446f18b9fcfee32c/docs/080-breaking-changes.rst#silent-changes-of-the-semantics)


```solidity
Ethos-Vault/contracts/libraries/DistributionTypes.sol

3: pragma experimental ABIEncoderV2;

Ethos-Vault/contracts/interfaces/IRewardsController.sol

3: pragma experimental ABIEncoderV2;

Ethos-Vault/contracts/interfaces/IRewardsDistributor.sol

3: pragma experimental ABIEncoderV2;
```


## [L-04] ERC20UPGRADEABLE SAFEAPPROVE() IS DEPRECATED
[SafeApprove is deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol#L31-L35) use `safeIncreaseAllowance()` and `safeDecreaseAllowance()` instead.

`Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol`
- [Line of code in ReaperBaseStrategyV4 ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74)

We recommend following your own patterns, as in this case: 
- [Solution code in ReaperStrategyGranarySupplyOnly](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L179)



## [L-05] Open TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment.

`Ethos-Core/contracts/StabilityPool.sol`
- [Lines of code in StabityPool](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L335)
- [Lines of code in StabityPool](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380)



## [L-06] DIVISION BEFORE MULTIPLICATION CAN LEAD TO PRECISION ERRORS
Because Solidity integer division may truncate, it is often preferable to do multiplication before division to prevent precision loss.

### PROOF OF CONCEPT

```solidity
Ethos-Core/contracts/TroveManager.sol

54: uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
55: uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; 
```

### MITIGATION
We recommend the protocol avoid divison before multiplication and always perform division operation at last.


##  [L-07] IMMUTABLE ADDRESSES LACK ZERO-ADDRESS CHECK
Constructors should check the address written in an immutable address variable is not the zero address.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

52: IERC20Metadata public immutable token;
```


## [L-08] USE OF FLOATING PRAGMA
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

### MITIGATION

We recommend following your own best practice patterns in `ETHOS-VAULT` as they do in `ETHOS-CORE` for interfaces and contracts.

Otherwise in this case we recommend use the floating and follow your own best practice patterns in the libraries folder in `ETHOS-VAULT` but missing in `DistributionTypes.sol` in the same libraries folder.


## [L-09] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE
The critical procedures should be two step process.

### PROOF OF CONCEPT
- [lines of code in BorrowerOperations](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L167)
- [lines of code in TroveManager](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L294)
- [lines of code in StabilityPool](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L302)

### MITIGATION 
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.


## [N-01] TOO MANY BITS TO DESCRIBE SMALL QUANTITIES
Gas is expensive on Ethereum so it's best to optimize for it and not to use variables that are larger than what we need. If all we are going to store in a variable are numbers from 1 to max 100, it doesn't make sense to use uint256. It's a waste of gas and money.

Below, (almost) all the max values for each uint. We've added the number of digits for each value, so for example, if you know that your max value will have no more than 12 digits, you can safely choose uint40, which has 13 digits.

Don't include anything between uint128 and uint256. If you're going to use such large numbers, you'll probably use uint256 by default.

We've created a [cheatsheet](https://gist.github.com/catellaTech/1f96a90da9d09034ee3165996bcd8b64). Now you don't have to worry if you are chosen uint is going to be enough or too small.

Now your uints will always be within range and will never take up more space than needed. Happy savings!!

For example in this case instead of using uint256, you can use uint8 [Line of code in CollateralConfig](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63)

`There are multuples instances across the scope.`


## [N-02] MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS
In the folowing functions below, there are some checks that can be made in order to achieve more safe and efficient code.

Address zero check can be added in the functions __ReaperBaseStrategy_init, ReaperBaseStrategyV4 contract to ensure the new addresses aren't address(0).


```solidity
Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

72: vault = _vault;
73: want = _want;

```

## [N-03] RACE CONDITION ON ERC20 APPROVAL
Using approve() to manage allowances opens yourself and users of the token up to frontrunning.
Best practice, but doesn't usually matter.

[Explanation](https://docs.google.com/document/d/1UJhFpBXTj79nATfCoCnSCjlFTDwmpnkiR6yhd3E1TgA/edit#heading=h.m9fhqynw2xvt) of this possible attack vector.

### PROOF OF CONCEPT
[Ref Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L336-L342)

### MITIGATION
A potential fix includes preventing a call to approve if all the previous tokens are not spent through adding a check that the allowed balance is 0:

`require(allowed[msg.sender][_spender] == 0)`.


## [N-04] CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES 
For better readability, you should name the imports instead of using the regular ones.

Example:
```solidity
5: import {DistributionTypes} from "../libraries/DistributionTypes.sol";
```
`Instances - All of the contracts.`


## [N-05] LONG REVERT STRINGS
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

### PROOF OF CONCEPT

```solidity
Ethos-Core/contracts/LUSDToken.sol

346:function _requireValidRecipient(address _recipient) internal view {
        require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
        );
        require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
        );
    }
```

### MITIGATION

If the contract(s) in scope allow using Solidity ^0.8.0, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec. 



## [N-06] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)

Use it for better code readability.

`Ethos-Vault/contracts/ReaperVaultV2.sol`
- [Line of code in ReaperVaultV2 ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)

### MITIGATION

We recommend following your own best practice patterns like:

```solidity
Ethos-Core/contracts/Dependencies/LiquityBase.sol

22: uint constant public LUSD_GAS_COMPENSATION = 10e18;

```

## [N-07]  EVENTS IS MISSING INDEXED FIELDS

Index event fields make the field more quickly accessible to off-chain. Each event should use three [indexed](https://docs.soliditylang.org/en/v0.8.0/contracts.html#events) fields if there are three or more fields.

### PROOF OF CONCEPT

```solidity
Ethos-Core/contracts/LUSDToken.sol

// --- Events ---
78: event TroveManagerAddressChanged(address _troveManagerAddress);
79: event StabilityPoolAddressChanged(address _newStabilityPoolAddress);
80: event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
81: event GovernanceAddressChanged(address _governanceAddress);
82: event GuardianAddressChanged(address _guardianAddress);
```
There are too many instances with this inconsistency.


## [N-08] MISSING NATSPEC 
Some functions don't have description of what they are supposed to do.

There are too many instances with this inconsistency.


## [N-09] MISSING ERROR MESSAGES IN REQUIRE STATEMENTS
Require statements should have descriptive strings to describe why the revert occurs.

```solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

167: require(step[0] != address(0));
168: require(step[1] != address(0));


Ethos-Core/contracts/TroveManager.sol

250: require(msg.sender == owner);
551: require(totals.totalDebtInSequence > 0);
716: require(_troveArray.length != 0);
```

## [N-10] USE CUSTOM ERRORS INSTEAD OF REVERT STRINGS TO SAVE GAS
We found multiple instances with long revert strings. We recommend upgrading to version 0.8.4 at least for the vault contracts that have version ^0.8.0, in order to use and optimize your code in terms of gas and readability with the new improvements that Solidity offers, such as custom errors


## [N-11] USE A MORE RECENT VERSION OF SOLIDITY 
Old version of solidity is used, consider using the new one `0.8.19`. You can see what new versions offer regarding bug fixed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

Instances - All of the contracts.


## [N-12] LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR 

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

111: constructor(
        address _token,

Ethos-Core/contracts/TroveManager.sol

225: constructor() public {
        // makeshift ownable implementation to circumvent contract size limit
        owner = msg.sender;
    }
```
### MITIGATION

Consider adding zero-address checks in the discussed constructors:  

```solidity
require(newAddr != address(0));
```

## [N-13] LINES ARE TOO LONG
Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

### PROOF OF CONCEPT
```solidity
Ethos-Core/contracts/BorrowerOperations.sol

172: function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

248: function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {

268: function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {    

282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

510: function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {
``` 

### MITIGATION
We recommend following your own best practice patterns like:

```solidity
function _getNewTroveAmounts(
        uint _coll,
        uint _debt,
        uint _collChange,
        bool _isCollIncrease,
        uint _debtChange,
        bool _isDebtIncrease
    )
        internal
        pure
        returns (uint, uint){...}
``` 

`There are multuples instances accross the scope.`

## [N-14] DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES 

If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

example: 

Not recommended ❌:

```solidity
for (uint256 i = 0 ; i ...; i++) {// ...}
```
Recommended ✔: 

```solidity
for (uint256 i; i ...; i++) {// ...}
```

```solidity
Ethos-Core/contracts/ActivePool.sol
32: bool public addressesSet = false;
```

`There are multuples instances accross the scope.`
```solidity
Ethos-Core/contracts/CollateralConfig.sol
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
Ethos-Core/contracts/LUSDToken.sol
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
```


## [R-01] NUMERIC VALUES HAVING TO DO WITH TIME SHOULD USE TIME UNITS FOR READABILITY
Suffixes like seconds, minutes, hours, days and weeks after literal numbers can be used to specify units of time where seconds are the base unit and units are considered naively in the following way:

`1 == 1 seconds` `1 minutes == 60 seconds` `1 hours == 60 minutes` `1 days == 24 hours` `1 weeks == 7 days`

```solidity
Ethos-Core/contracts/TroveManager.sol

48: uint constant public SECONDS_IN_ONE_MINUTE = 60;
53: uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

We recommend following your own patterns, as in this case: 
- [Solution code in ReaperBaseStrategyv4](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L24-L25)



## [R-02] USE REQUIRE INSTEAD OF ASSERT
Properly functioning code should never reach a failing assert statement. If it happened, it would indicate the presence of a bug in the contract. A failing assert uses all the remaining gas, which can be financially painful for a user.

```solidity
Ethos-Core/contracts/TroveManager.sol

417:  assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);
```
There are more instances with this issues:

```
Ethos-Core/contracts/BorrowerOperations.sol
Ethos-Core/contracts/StabilityPool.sol
Ethos-Core/contracts/LUSDToken.sol
```

## [R-03] SOME NUMBER VALUES CAN BE REFACTORED WITH _
Consider using underscores for number values to improve readability.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol
41: uint256 public constant PERCENT_DIVISOR = 10000;
```
We recommend following your own patterns, as in this case: 
- [Solution code in ReaperBaseStrategyv4](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23)

## [R-04] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The regular if-else statement can be refactored in an abbreviated way to write it. This increases readability and reduces the overall SLOC (lines of code).

```solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

92:   if (wantBal < _amountNeeded) {
            _withdraw(_amountNeeded - wantBal);
            liquidatedAmount = balanceOfWant();
        } else {
            liquidatedAmount = _amountNeeded;
        }
```
There are more instances with this:

```solidity
Ethos-Core/contracts/BorrowerOperations.sol
Ethos-Core/contracts/TroveManager.sol
Ethos-Core/contracts/ActivePool.sol
Ethos-Core/contracts/StabilityPool.sol
Ethos-Core/contracts/LQTY/LQTYStaking.sol
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
Ethos-Core/contracts/LUSDToken.sol
Ethos-Vault/contracts/ReaperVaultV2.sol
Ethos-Vault/contracts/ReaperVaultERC4626.sol
Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
```

## [R-05] REVERT SHOULD BE USED ON SOME FUNCTIONS INSTEAD OF RETURN 
Some instances just return without doing anything, consider applying revert statement instead with a discriptive string why it does that.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

210:    if (strategies[_strategy].allocBPS == 0) {
            return;
        }
```
There are more instances with this issues:
```
Ethos-Core/contracts/TroveManager.sol
Ethos-Core/contracts/StabilityPool.sol
Ethos-Vault/contracts/ReaperVaultV2.sol
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
```

## [R-06] MISSING IMMUTABLE KEYWORD
In the `TroveManager contract`, the variable in the line `21` can be set as immutable since it's initialized in the constructor. It is a good practice and saves gas.

`Friendly reminder that the Solidity compiler does not reserve a storage slot for constants and immutables but instead inlines their values in the bytecode in the places where they are read.`


## [R-07] INSTEAD OF USING "&&" IN REQUIRE, JUST USE REQUIRE MULTIPLE TIMES
use require multiple times instead of &&.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

653: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

Ethos-Core/contracts/TroveManager.sol
1539:  require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```
- [Line of code in LUSDToken ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L357)

## [O-01]  FUNCTION NAMING SUGGESTIONS
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just this contract.

```solidity
Ethos-Vault/contracts/ReaperVaultERC4626.sol

269: function roundUpDiv
``` 

## [O-02] PROPER USE OF "GET" AS A FUNCTION NAME PREFIX
Clear function names can increase readability. Follow a standard convertion function names such as using `get` for getter `(view/pure) functions`.

```solidity
Ethos-Core/contracts/TroveManager.sol

1152: function hasPendingRewards(address _borrower, address _collateral) public view 

Ethos-Core/contracts/StabilityPool.sol

410:  function depositSnapshots_S(address _depositor, address _collateral) external override view 

Ethos-Core/contracts/LUSDToken.sol

212: function totalSupply() external view
216: function balanceOf(address account) external view
226: function allowance(address owner, address spender) external view 
254: function domainSeparator() public view
292: function nonces(address owner) external view 
399: function name() external view
403: function symbol() external view
407: function decimals() external view
411: function version() external view
415: function permitTypeHash() external view

Ethos-Vault/contracts/ReaperVaultV2.sol

225: function availableCapital() public view
278: function balance() public view

Ethos-Vault/contracts/ReaperVaultERC4626.sol

29: function asset() external view
37: function totalAssets() external view
51: function convertToShares(uint256 assets) public view
66: function convertToAssets(uint256 shares) public view 
79: function maxDeposit(address) external view
96: function previewDeposit(uint256 assets) external view 
122: function maxMint(address) external view
139: function previewMint(uint256 shares) public view
165: function maxWithdraw(address owner) external view
183: function previewWithdraw(uint256 assets) public view
220: function maxRedeem(address owner) external view
240: function previewRedeem(uint256 shares) external view

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

222: function balanceOf() public view 
226: function balanceOfWant() public view
230: function balanceOfPool() public view
``` 

## [O-03] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

```solidity
Ethos-Core/contracts/StabilityPool.sol

857: address[] memory collaterals = collateralConfig.getAllowedCollaterals();
``` 

- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L806-L807)
- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L722)
- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687)
- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L663)
- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L376)
- [Line of code in StabilityPool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L332)

- [Line of code in ActivePool ](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L240)

- [Line of code in BorrowerOperations](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L175-L176)


## [O-04] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2). Subtractions act the same way.


```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol 

168: totalAllocBPS += _allocBPS;
194: totalAllocBPS -= strategies[_strategy].allocBPS;
196: totalAllocBPS += _allocBPS;
214: totalAllocBPS -= strategies[_strategy].allocBPS;
390: value -= loss;
391: totalLoss += loss;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
450: stratParams.losses += loss;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;
505: strategy.gains += vars.gain;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment;
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;

Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
133: toFree += profit;
```