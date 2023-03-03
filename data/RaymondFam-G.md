## Unneeded `assert()`
The following assertion of `vars.compositeDebt != 0` is unnecessary. Even if `_LUSDAmount` has accidentally been inputted as `0` making `vars.LUSDFee == 0` too, [`_getCompositeDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L41-L43) is going to have the constant `LUSD_GAS_COMPENSATION` which is `10e18` added to `vars.compositeDebt`.

Consider removing it from `openTrove()` to save gas on function call and also contract deployment:

[File: BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197)

```diff
-        assert(vars.compositeDebt > 0);
```