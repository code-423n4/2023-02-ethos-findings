Splitting `require()` statements that uses `&&` saves gas
====================================
Description:
-------------
Instead of using the `&&` operator on a single require statement, use two `require` can save more gas

i.e for `require(version==1 && balance >= 0, "Error")` use
```
require(version==1);
require(balance>=0);
```

Proof Of Concept
-------------------
* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653
```653:  require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,"Max fee percentage must between 0.5% and 100%");```

* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L348
```348: require(_recipient != address(0) && _recipient != address(this),"LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address");```

* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L353
```353: require(!stabilityPools[_recipient] && !troveManagers[_recipient] && !borrowerOperations[_recipient],"LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps");```

* https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539
``` 1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);```