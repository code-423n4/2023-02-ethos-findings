# Use require instead of assert in _adjustTrove

In Ethos-Core/contracts/BorrowerOperations.sol, `assert` is used in L331 to check if `_collWithdrawal <= vars.coll`. However, `_collWithdrawal` can be directly provided by the user, for example, when calling `withdrawColl`. Since assert consumes all gas, if the user maintains their collateral amount independently but miscalculates the amount due to other reasons, calling `withdrawColl` will fail and consume all gas, resulting in a poor user experience.

# Change setTroveStatus to activeTrove.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562

`setTroveStatus` is only used once in BorrowerOperations, and the status is fixed at 1, so there is no need to pass an additional parameter of 1.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L219

