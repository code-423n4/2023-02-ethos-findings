# QA report

## Low Risk
| L-N    |Issue|
|:------:|:----|
| [L&#x2011;01] | Single-step ownership transfer pattern is dangerous | 9 |
| [L&#x2011;02] | Decimals() not part of ERC20 standard | 1 |
| [L&#x2011;03] | Open TODO in the code | 2 |
| [L&#x2011;04] | Using vulnerable dependency of OpenZeppelin | 2 |
| [L&#x2011;05] | Usage of `ecrecover` should be replaced with usage of OpenZeppelin's ECDSA library 

Total: 5 issues

## Non-critical

| N-N    |Issue|
|:------:|:----|
| [N&#x2011;01] | Solidity safe pragma best practices are not used | 2 |
| [N&#x2011;02] | NatSpecs are missing | 6 |
| [N&#x2011;03] | Events, structs and custom errors declaration | 11 |
| [N&#x2011;04] | Event is missing `indexed` fields | 19 |
| [N&#x2011;05] | Typos in the comments | 31 |
| [N&#x2011;06] | Redundant code | 2 |
| [N&#x2011;07] | Use of magic number | 1 |
| [N&#x2011;08] | Interchangeable usage of uint and uint256 in in `BorrowerOperations.sol` | 1 |
| [N&#x2011;09] | Lines are too long | 1 |

Total: 9 issues

## Low Risk

### [L-01] Single-step ownership transfer pattern is dangerous

File: `CollateralConfig.sol`

Inheriting from OpenZeppelin's `Ownable` contract means you are using a single-step ownership transfer pattern. If an admin provides an incorrect address for the new owner this will result in none of the `onlyOwner` marked methods being callable again. The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim its new rights before they are transferred.

```solidity
import "./Dependencies/Ownable.sol";
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L6

#### Recommendation

Use OpenZeppelin's `Ownable2Step` instead of `Ownable`


### [L-02] Decimals() not part of ERC20 standard

File: `CollateralConfig.sol`


The `initialize` method is looping through `_collaterals` array and fetching tokens decimals. However decimals() is not part of the official ERC20 standard and might fall for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

```solidity
uint256 decimals = IERC20(collateral).decimals();
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63

#### Recommendation

Consider using a helper function to safely retrieve token decimals.


### [L-03] Open TODO in the code

There is an open TODO in `StabilityPool.sol` this implies changes that might not be audited. Resolve it or remove it.

```solidity
/* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
```

```solidity
/* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L335

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L380


### [L-04] Using vulnerable dependency of OpenZeppelin

The package.json configuration file says that the project is using 4.7.3 of OZ which has a not last update version:


#### Recommendation

Use patched versions.

Latest non vulnerable version 4.8.0.


### [L-05] Usage of ecrecover should be replaced with usage of OpenZeppelin's ECDSA library

File:  `LUSDToken.sol`

Signature malleability is one of the potential issues with ecrecover. Even though it is not a threat to the current implementation using the highest security standards is always good. 

```solidity
address recoveredAddress = ecrecover(digest, v, r, s);
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L288


#### Recommendation

Import ECDSA library. Replace the usage of ecrecover with the ECDSA.recover functionality.

## Non-critical

### [NC-01] Solidity safe pragma best practices are not used

Always use a stable pragma to be certain that you deterministically compile the Solidity code to the same bytecode every time. Also IStargateReceiver and IStargateRouter interfaces are using an old compiler version - upgrade it to a newer version, use the same pragma statement throughout the whole codebase.


### [NC-02] NatSpecs are missing

File: `CollateralConfig.sol`

@notice, @param and @return fields are missing throughout the codebase in a lot of external functions. NatSpec documentation is essential for better understanding of the code by developers and auditors and is strongly recommended, add it to each  struct, enum, function, interface and contract. Please refer to the [NatSpec format](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) and follow the guidelines outlined there.


### [NC-03] Events, structs and custom errors declaration

Most of the events and structs are declared in the implementation contracts when It is a best practice to be declared in the interface not in the implementation contract.


### [NC-04] Event is missing indexed fields

File: `CollateralConfig.sol`

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

```solidity
event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L37
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L38


### [NC-05] Typos in the comments
`lenghts` -> `lengths`

`elswhere` -> `elsewhere`

`calculcate` -> `calculate`

`occured` -> `occurred`


### [NC-06] Redundant code

File: `BorrowerOperations.sol`

`./Dependencies/console.sol";` import is not used and should be removed.


### [NC-07] Use of magic number

File: `BorrowerOperations.sol`

Constant variables instead of magic numbers can help keep the code easier to read and maintain.

```solidity
troveManagerCached.closeTrove(msg.sender, _collateral, 2);
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L397


File:  `ActivePool.sol`

Although it is understandable that these numbers are regarding the BPS value, constant variables instead of magic numbers can help keep the code easier to read and maintain.

```solidity
require(_bps <= 10_000, "Invalid BPS value");
```

```solidity
require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
```

```solidity
require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L127
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L133
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L145


### [NC-08] Interchangeable usage of uint and uint256 in in `BorrowerOperations.sol` 

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

### [NC-09] Lines are too long

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

In `ActivePool.sol`, there are multiple instances of this issue. Here are couple of examples:

```solidity
vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
...
 if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L262
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L264
