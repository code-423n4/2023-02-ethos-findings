
## Summary

### Low-Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Use ```require()``` instead of ```assert()```. | 15 |
| [L&#x2011;02] | Trove status check depends on the enumeration order in BorrowerOperations and StabilityPool. | 2 |
| [L&#x2011;03] | `_harvestCore()` has no slippage protection when swapping. | 1 |
| [L&#x2011;04] | Upgradeable contracts miss storage gaps. | 2 |

Total: 20 instances over 4 issues

Note: The table above was created considering the **automatic findings**; thus, those are not included.


### [L&#x2011;01]  Use ```require()``` instead of ```assert()```
Prior to solc 0.8.0, `assert()` used the invalid opcode which used up all the remaining gas while `require()` used the revert opcode which refunded the gas and therefore the importance of using require() instead of assert() was greater. However, after 0.8.0, `assert()` uses revert opcode just like `require()` but creates a Panic(uint256) error instead of Error(string) created by require(). 

It's mentioned in the Solidity’s documentation:
"Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.”

whereas

“The require function either creates an error without any data or an error of type Error(string). It should be used to ensure valid conditions that cannot be detected until execution time. This includes conditions on inputs or return values from calls to external contracts.”

*There are 15 instances of this issue:*

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L313
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338

### [L&#x2011;02]  Trove status check depends on the enumeration order in BorrowerOperations and StabilityPool.
BorrowerOperations and StabilityPool check the active status of a trove by comparing TroveManager's getTroveStatus with 1:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L542
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1544-L1546

With further system development, it will be harder to track fixes needed for enumeration change.
Consider implementing TroveManager.isTroveActive(borrower) where trove.status is checked against Status.active and the corresponding boolean is returned.

### [L&#x2011;03]  `_harvestCore()` has no slippage protection when swapping.
Single swaps of `_harvestCore()` contains no slippage or deadline, which makes it vulnerable to sandwich attacks, and MEV exploits, and may lead to a significant loss of yield.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L124

### [L&#x2011;04]  Upgradeable contracts miss storage gaps.
A lack of storage gaps may lead to a typical "storage slot collision" problem. Gaps are some fixed arrays that are added to the code to give the developers sufficient freedom to add some storage variables for a possible future upgrade. Otherwise, it may be challenging to write new implementation code.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol