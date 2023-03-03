## Unneeded `assert()`
The following assertion of `vars.compositeDebt != 0` is unnecessary. Even if `_LUSDAmount` has accidentally been inputted as `0` making `vars.LUSDFee == 0` too, [`_getCompositeDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L41-L43) is going to have the constant `LUSD_GAS_COMPENSATION` which is `10e18` added to `vars.compositeDebt`.

Consider removing it from `openTrove()` to save gas on function call and also contract deployment:

[File: BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197)

```diff
-        assert(vars.compositeDebt > 0);
```
## Internal function with one line code excution
Internal functions entailing only one line of code may be embedded inline with the function logic invoking it.

For instance, the following code line may be refactored as follows to save gas both on function calls and contract deployment considering [`_requireNonZeroDebtChange`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551-L553) is only called once in BorrowerOperations.sol by `_adjustTrove()` and not anywhere else:

[File: BorrowerOperations.sol#L294](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L294)

```
-        _requireNonZeroDebtChange(_LUSDChange);
+        require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
```