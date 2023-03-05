## GAS-2: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* ReaperVaultV2.sol (Line: 168, 194, 196, 214, 395, 396, 445, 452, 515, 521)

## GAS-3: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* ActivePool.sol (Line: 127, 133)
* TroveManager.sol (Line: 372)

## GAS-4: Bytes32 constants are more efficient than string constants

### Description

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

### Affected file

* ActivePool.sol (Line: 30)
* BorrowerOperations.sol (Line: 21)
* CommunityIssuance.sol (Line: 19)
* LQTYStaking.sol (Line: 23)
* LUSDToken.sol (Line: 32, 33, 34)
* StabilityPool.sol (Line: 150)

## GAS-5: Empty blocks should be removed or emit something

### Description

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

### Affected file

* ReaperVaultERC4626.sol (Line: 16)

## GAS-8: Internal functions only called once can be inlined to save gas

### Description

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Affected file

* ActivePool.sol (Line: 320, 337)
* BorrowerOperations.sol (Line: 438, 455, 476, 524, 533, 537, 546, 551, 555, 567, 571, 624, 636, 640, 661, 682)
* LQTYStaking.sol (Line: 251, 261, 265, 269)
* LUSDToken.sol (Line: 320, 328, 360, 365, 375, 380, 393)
* ReaperStrategyGranarySupplyOnly.sol (Line: 86, 176, 187, 206)
* ReaperVaultV2.sol (Line: 462)
* StabilityPool.sol (Line: 421, 439, 597, 637, 657, 693, 729, 776, 794, 848, 852, 856, 869, 873)
* TroveManager.sol (Line: 478, 582, 671, 785, 864, 1082, 1213, 1321, 1339, 1516, 1538)

## GAS-10: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* ReaperStrategyGranarySupplyOnly.sol (Line: 104)
* ReaperVaultERC4626.sol (Line: 29, 37, 51, 66, 79, 96, 122, 165, 220, 240)
* StabilityPool.sol (Line: 512, 626)
* TroveManager.sol (Line: 330, 370, 370, 899, 1316, 1321)

## GAS-11: Prefix increments cheaper than Postfix increments

### Description

++i costs less gas than i++, especially when it’s used in for-loops (—i/i— too). Saves 5 gas per loop.

### Affected file

* ActivePool.sol (Line: 108)
* CollateralConfig.sol (Line: 56)
* LQTYStaking.sol (Line: 206, 228, 240)
* ReaperVaultERC4626.sol (Line: 273)
* StabilityPool.sol (Line: 351, 397, 640, 810, 831, 859)
* TroveManager.sol (Line: 608, 690, 808, 882)

## GAS-12: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* ReaperStrategyGranarySupplyOnly.sol (Line: 62)
* ReaperVaultV2.sol (Line: 295)
* TroveManager.sol (Line: 1045, 1440)

## GAS-13: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* CollateralConfig.sol (Line: 128)

## GAS-15: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* ReaperVaultV2.sol (Line: 581)

## GAS-16: Splitting require() statements that use && saves gas

### Description

Saves 16 gas per instance. If you’re using the Optimizer at 200, instead of using the && operator in a single require statement to check multiple conditions, multiple require statements with 1 condition per require statement should be used to save gas.

### Affected file

* BorrowerOperations.sol (Line: 653)
* LUSDToken.sol (Line: 347, 352)
* TroveManager.sol (Line: 1539)

## GAS-18: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* StabilityPool.sol (Line: 558, 559, 699, 700, 738, 739, 745, 817, 818)

## GAS-21: Use elementary types or a user-defined type instead of a struct that has only one member

### Affected file

* StabilityPool.sol (Line: 174)

## GAS-22: Use require() instead of assert()

### Description

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

### Affected file

* BorrowerOperations.sol (Line: 128, 197, 301, 331)
* LUSDToken.sol (Line: 312, 313, 321, 329, 337, 338)
* StabilityPool.sol (Line: 526, 551, 591)
* TroveManager.sol (Line: 417, 1224, 1279, 1342, 1348, 1414, 1489)

## GAS-24: Using Solidity version 0.8.17 will provide an overall gas optimization

### Description

Using at least 0.8.10 will save gas due to skipped extcodesize check if there is a return value.

### Affected file

* ReaperBaseStrategyv4.sol (Line: 3)
* ReaperStrategyGranarySupplyOnly.sol (Line: 3)
* ReaperVaultERC4626.sol (Line: 3)
* ReaperVaultV2.sol (Line: 3)

## GAS-26: Using private rather than public for constants saves gas

### Description

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

### Affected file

* ActivePool.sol (Line: 30)
* BorrowerOperations.sol (Line: 21)
* CollateralConfig.sol (Line: 21, 25)
* CommunityIssuance.sol (Line: 19)
* LQTYStaking.sol (Line: 23)
* LUSDToken.sol (Line: 32, 33, 34, 35)
* ReaperStrategyGranarySupplyOnly.sol (Line: 24, 25, 27, 29)
* ReaperVaultV2.sol (Line: 40, 41, 73, 74, 75, 76)
* StabilityPool.sol (Line: 150, 197)
* TroveManager.sol (Line: 48, 53, 54, 55, 61)

## GAS-28: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* LUSDToken.sol (Line: 284, 305)