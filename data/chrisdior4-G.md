# Gas Optimization report

| G-O    |Issue|
|:------:|:----|
| [G&#x2011;01] | Using private rather than public for constants, saves gas | 9 |
| [G&#x2011;02] | There's no need to set default values for variables | 9 |
| [G&#x2011;03] | Consider using custom errors instead of require statements with string error | 9 |
| [G&#x2011;04] | ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too) | 9 |
| [G&#x2011;05] | Cache array length outside of loop | 9 |
| [G&#x2011;06] | Bytes constants are more efficient than string constants | 9 |
| [G&#x2011;07] | Use require instead of assert | 9 |
| [G&#x2011;08] | Use x != 0 instead of x > 0 for uint types | 9 |
| [G&#x2011;09] | Long revert strings | 9 |
| [G&#x2011;10] | x = x - y costs less gas than x -= y, same for addition | 9 |
| [G&#x2011;11] | Using storage instead of memory for structs/arrays saves gas | 9 |
| [G&#x2011;12] | `internal` functions only called once can be inlined to save gas | 9 |
| [G&#x2011;13] | Splitting require() statements that use && saves gas | 9 |

Total: 13 issues

## [G-01] Using private rather than public for constants, saves gas

File: `CollateralConfig.sol`

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

```solidity
uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
...
uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L25


## [G-02] There's no need to set default values for variables

File: `CollateralConfig.sol`

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

```diff
- bool public initialized = false;
+ bool public initialized;
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L18

File:  `ActivePool.sol`

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

```diff
- bool public addressesSet = false;
+ bool public addressesSet;
```

Lines of code:
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L32


## [G-03] Consider using custom errors instead of require statements with string error

Custom errors reduce the contract size and can provide easier integration with a protocol. Consider using those instead of require statements with string errors

Couple of examples:

```solidity
require(!initialized, "Can only initialize once");
require(_collaterals.length != 0, "At least one collateral required");
require(_MCRs.length == _collaterals.length, "Array lengths must match");
require(_CCRs.length == _collaterals.length, "Array lenghts must match");
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L51
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L52
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L53
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L54


## [G-04] ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)

File: `CollateralConfig.sol`

```solidity
for(uint256 i = 0; i < _collaterals.length; i++) {
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56


## [G-05] Cache Array Length Outside of Loop

File: `CollateralConfig.sol`

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```diff
- for(uint256 i = 0; i < _collaterals.length; i++) {

+ uint256 collateralsLength = _collaterals.length;
+ for(uint256 i = 0; i < collateralsLength; i++) {
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56


## [G-06] Bytes constants are more efficient than string constants

File: `BorrowerOperations.sol`

If data can fit into 32 bytes, then you should use bytes32 datatype rather than string as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size. That will save gas on the contract.

```diff
- string constant public NAME = "BorrowerOperations";
+ bytes32 constant public NAME = "BorrowerOperations";
```


## [G-07] Use require instead of assert

File: `BorrowerOperations.sol`

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

```solidity
assert(MIN_NET_DEBT > 0);
....
assert(vars.compositeDebt > 0);
....
assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
....
assert(_collWithdrawal <= vars.coll); 
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L128
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L197
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L301
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L301


## [G-08] Use x != 0 instead of x > 0 for uint types

File: `BorrowerOperations.sol`

The != operator costs less gas than > and for uint types you can use it to check for non-zero values to save gas

```diff
- assert(MIN_NET_DEBT > 0);
+ assert(MIN_NET_DEBT != 0);
- assert(vars.compositeDebt > 0);
+ assert(vars.compositeDebt != 0);
- assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
+ assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp != 0 && _LUSDChange == 0));
- if (!_isDebtIncrease && _LUSDChange > 0) {
+ if (!_isDebtIncrease && _LUSDChange != 0) {
- require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
+ require(_LUSDChange != 0, "BorrowerOps: Debt increase requires non-zero debtChange");
```


## [G-09] Long revert strings

File: `BorrowerOperations.sol`

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

```solidity
require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
 
require(status == 1, "BorrowerOps: Trove does not exist or is closed");
 
require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

require(!_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),"BorrowerOps: Operation not permitted during Recovery Mode");

require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");

require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

require(_maxFeePercentage <= DECIMAL_PRECISION,"Max fee percentage must less than or equal to 100%");

require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION, "Max fee percentage must be between 0.5% and 100%");
```

Lines of code: 

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L525
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L530
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L534
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L538
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L543
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L552
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L563
- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L568


### [G-10]  x = x - y costs less gas than x -= y, same for addition

File: `ReaperVaultV2.sol`

You can replace all -= and += occurrences to save gas.

Example on how to fix it:

```diff
- totalAllocBPS += _allocBPS;
+ totalAllocBPS = totalAllocBPS + _allocBPS;
```

```solidity
totalLoss += loss;
```

```solidity
stratParams.losses += loss;
```

```solidity
strategy.gains += vars.gain;
```

```solidity
strategy.allocated += vars.credit;
```

```solidity
totalAllocated += vars.credit;
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L391

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L450

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L505

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L520

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L521



### [G-11] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

File: `ReaperVaultV2.sol`

```solidity
LocalVariables_report memory vars;
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L494


### [G-12] `internal` functions only called once can be inlined to save gas
File: `BorrowerOperations.sol`

function `_getUSDValue` is only called in the same contract in function `getUSDValue`. Inline it to save gas.


### [G-13] Splitting require() statements that use && saves gas

This will have a large deployment gas cost but with enough runtime calls the split require version will be 3 gas cheaper

File: `ActivePool.sol`

```solidity
if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
...
} else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
```

Lines of code:

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L264

- https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L269
