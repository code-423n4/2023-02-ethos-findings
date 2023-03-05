## Unneeded `assert()`
The following assertion of `vars.compositeDebt != 0` is unnecessary. Even if `_LUSDAmount` has accidentally been inputted as `0` making `vars.LUSDFee == 0` too, [`_getCompositeDebt()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L41-L43) is going to have the constant `LUSD_GAS_COMPENSATION` which is `10e18` added to `vars.compositeDebt`.

Consider removing it from `openTrove()` to save gas on function call and also contract deployment:

[File: BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197)

```diff
-        assert(vars.compositeDebt > 0);
```
Similarly, asserting `MIN_NET_DEBT` greater than zero in `setAddresses()` of BorrowerOperations.sol is unnecessary since the constant has been assigned [90e18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L25) in LiquityBase.sol:

[File: BorrowerOperations.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128)

```diff
-        assert(MIN_NET_DEBT > 0);
```
## Use smaller uint128 or smaller type and pack structs variables to use lesser amount of storage slots
As the solidity EVM works with 32 bytes, most variables work fine with less than 32 bytes and may be packed inside a struct when sitting next to each other so that they can be stored in the same slot, this saves gas when writing to storage ~2000 gas.

For instance, struct `LocalVariables_OuterLiquidationFunction` of TroveManager.sol may be refactored as follows:

[File: TroveManager.sol#L129-L138](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L129-L138)

```diff
    struct LocalVariables_OuterLiquidationFunction {
-        uint256 collDecimals;
+        uint128 collDecimals;
+        bool recoveryModeAtStart;
-        uint256 collCCR;
-        uint256 collMCR;
+        uint128 collCCR;
+        uint128 collMCR;
-        uint price;
-        uint LUSDInStabPool;
+        uint128 price;
+        uint128 LUSDInStabPool;
-        bool recoveryModeAtStart;
-        uint liquidatedDebt;
-        uint liquidatedColl;
+        uint128 liquidatedDebt;
+        uint128 liquidatedColl;
    }
```
This alone will save ~8000 gas with 8 slots of storage reduced to 4 slots. 

## Internal function with one line code excution
Internal functions entailing only one line of code may be embedded inline with the function logic invoking it.

For instance, the following code line may be refactored as follows to save gas both on function calls and contract deployment considering [`_requireNonZeroDebtChange`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551-L553) is only called once in BorrowerOperations.sol by `_adjustTrove()` and not anywhere else:

[File: BorrowerOperations.sol#L294](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L294)

```
-        _requireNonZeroDebtChange(_LUSDChange);
+        require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
```
## Unused imports
Consider removing unused imports to help save gas on contract deployment.

Here is an instance entailed:

[File: LUSDToken.sol#L9](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L9)

```diff
- import "./Dependencies/console.sol";
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: CollateralConfig.sol#L128-L131](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L128-L131)

```diff
+    function _checkCollateral() private view {
+        require(collateralConfig[_collateral].allowed, "Invalid collateral address");
+    }

    modifier checkCollateral() {
-        require(collateralConfig[_collateral].allowed, "Invalid collateral address");
+        _checkCollateral();
        _;
    }
```
## Use storage instead of memory for structs/arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

For instance, the specific example below may be refactored as follows:

[File: TroveManager.sol#L722-L723](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L722-L723)

```diff
-        LocalVariables_OuterLiquidationFunction memory vars;
+        LocalVariables_OuterLiquidationFunction storage vars;
-        LiquidationTotals memory totals;
+        LiquidationTotals storage totals;
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, the following `>=` inequality instance may be refactored as follows:

[File: TroveManager.sol#L1503](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1503)

```diff
-        if (timePassed >= SECONDS_IN_ONE_MINUTE) {
// Rationale for subtracting 1 on the right side of the inequality:
// x >= 10; [10, 11, 12, ...]
// x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        if (timePassed > SECONDS_IN_ONE_MINUTE - 1) {
```
Similarly, the `<=` instance below may be replaced with `<` as follows:

[File: TroveManager.sol#L383](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L383)

```diff
-        if (_ICR <= _100pct) {
// Rationale for adding 1 on the right side of the inequality:
// x <= 10; [10, 9, 8, ...]
// x < 10 + 1 is the same as x < 11; [10, 9, 8 ...]
+        if (_ICR < _100pct + 1) {
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the `&&` instance below may be refactored as follows:

Note: Comment on the changes made for better readability where deemed fit.

[File: TroveManager.sol#L817](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L817)

```diff
-                if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }
+                if (!(vars.ICR < vars.collMCR || vars.remainingLUSDInStabPool != 0)) { continue; }
```
## Split require statements using &&
Instead of using the `&&` operator in a single require statement to check multiple conditions, using multiple require statements with 1 condition per require statement will save 3 GAS per `&&`.

For instance, the following code line may be refactored as follows:

[File: TroveManager.sol#L1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)

```diff
-        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
+        require (TroveOwnersArrayLength > 1);
+        require (sortedTroves.getSize(_collateral) > 1);
```