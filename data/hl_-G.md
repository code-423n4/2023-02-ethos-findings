# Table of contents

- [G-01] Use a more recent version of solidity
- [G-02] x += y costs more gas than x = x + y for state variables
- [G-03] Duplicated require() / revert() checks should be refactored to a modifier or function
- [G-04] Structs can be packed into fewer storage slots
- [G-05] Use Shift Right/Left instead of Division/Multiplication
- [G-06] Use constants instead of type(uintx).max
- [G-07] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
- [G-08] Use double require instead of using &&
- [G-09] Use nested if and avoid multiple check combinations
- [G-10] Sort Solidity operations using short-circuit mode
- [G-11] Optimize names to save gas

## [G-01] Use a more recent version of solidity

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining. 
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads. 
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings and get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>. 
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value. 
Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked (<str>,<str>)

```
LUSDToken.sol
3:    pragma solidity 0.6.11;
```

```
ReaperVaultERC4626.sol
3: pragma solidity ^0.8.0;
```

## [G-02] x += y costs more gas than x = x + y for state variables

Using the addition operator instead of plus-equals saves 113 gas.

For example: 
```
File:   2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol 
168:    totalAllocBPS += _allocBPS;
196:    totalAllocBPS += _allocBPS;
```

## [G-03] Duplicated require() / revert() checks should be refactored to a modifier or function

Saves deployment costs

```
File:    2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol 
180:    require(strategies[_strategy].activation != 0, "Invalid strategy address");
193:    require(strategies[_strategy].activation != 0, "Invalid strategy address");
```

## [G-04] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```
File:    2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol 
58:    struct ChainlinkResponse {
59:    uint80 roundId;
60:    int256 answer;
61:    uint256 timestamp;
62:    bool success;
63:    uint8 decimals;
64:    }
```

```
File:    2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol 
66:    struct TellorResponse {
67:    bool ifRetrieve;
68:    uint256 value;
69:    uint256 timestamp;
70:    bool success;
71:    }
```

## [G-05] Use Shift Right/Left instead of Division/Multiplication

A division/multiplication by any number x being a power of 2 can be calculated by shifting to the right/left. While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas.

Furthermore, Solidity’s division operation also includes a division-by-0 prevention which is bypassed using shifting.

For example: 
```
File:    2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol 
L125:    lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

## [G-06] Use constants instead of type(uintx).max

Using a constant value does not require the Solidity compiler to calculate the maximum value at compile time, which can reduce the amount of bytecode generated and therefore, reduce the gas cost of deploying and executing the contract.

For example: 
```
File:     2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol 
81:    if (tvlCap == type(uint256).max) return type(uint256).max;
```

## [G-07] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, the contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

For example:

```
File:     2023-02-ethos/Ethos-Core/contracts/TroveManager.sol 
1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
```

## [G-08] Use double require instead of using &&
It saves more gas.
When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.

For example: 
```
File:     2023-02-ethos/Ethos-Vault/contracts/mixins/ReaperAccessControl.sol 
39:    require(specifiedRoleFound && senderHighestRoleFound, "Unauthorized access");
```

## [G-09] Use nested if and avoid multiple check combinations

Using nested is cheaper than &&. It is also easier to read the code. 

For example: 
```
File:     2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol 
311:    if (_isDebtIncrease && !isRecoveryMode) { 
337:    if (!_isDebtIncrease && _LUSDChange > 0) {
```

## [G-10] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

For example: 


```
File:     2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol 
311:    if (_isDebtIncrease && !isRecoveryMode) { 
337:    if (!_isDebtIncrease && _LUSDChange > 0) {
538:    require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
```

## [G-11] Optimize names to save gas

Contracts most called functions could save gas by function ordering via `Method ID`. 
Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower `Method ID`) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 