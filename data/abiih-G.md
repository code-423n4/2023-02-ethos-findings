# Report

---

## Gas Optimizations

---

|     | Findings                                                                                                                                     |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [ Use Internal View Functions in Modifiers To Save Bytecode](#use-internal-view-functions-in-modifiers-to-save-bytecode)                  |
| 2   | [Increments/Decrements Can Be Unchecked In For-Loops](#incrementsdecrements-can-be-unchecked-in-for-loops)                                |
| 3   | [Use Named Returns For Local Variables Where It Is Possible](#use-named-returns-for-local-variables-where-it-is-possible)                 |
| 4   | [Multiple Mapping Can be Combined In a Single Struct](#multiple-mapping-can-be-combined-in-a-single-struct)                               |
| 5   | [Using Storage Instead Of Memory For Structs/Arrays Saves Gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)              |
| 6   | [Avoid Emitting a Storage Variable When a Memory Value Is Available](#avoid-emitting-a-storage-variable-when-a-memory-value-is-available) |

---

1. ### Use Internal View Functions in Modifiers To Save Bytecode

   When you add a function modifier, the code of that function is picked up and put in the function modifier in place of the "\_" symbol. This can also be understood as ‘The function modifiers are inlined”. In normal programming languages, inlining small code is more efficient without any real drawback but Solidity is no ordinary language. In Solidity, the maximum size of a contract is restricted to 24 KB by EIP 170. If the same code is inlined multiple times, it adds up in size and that size limit can be hit easily.

   Internal functions, on the other hand, are not inlined but called as separate functions. This means they are very slightly more expensive in run time but save a lot of redundant bytecode in deployment. Internal functions can also help avoid the dreaded “Stack too deep Error” as variables created in an internal function don’t share the same restricted stack with the original function, but the variables created in modifiers share the same stack limit.

   - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L128

   ```diff
   - modifier checkCollateral(address _collateral) {
   -       require(collateralConfig[_collateral].allowed, "Invalid collateral address");
   -    _;
   }
   + modifier checkCollateral(address _collateral) {
   + _isAllowed(_collateral);
   +   _;
   }
   +function _isAllowed(address _collateral) internal view {
       require(collateralConfig[_collateral].allowed, "Invalid collateral address");
   }

   ```

2. ### Increments/Decrements Can Be Unchecked In For-Loops
   Consider wrapping with an unchecked block here (around 25 gas saved per instance): The change would be:

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

```diff
-for(uint256 i = 0; i < _collaterals.length; i++) {}
+   for (uint256 i = 0; i < _collaterals.length;) {
  // ...
+ unchecked { ++i; }

```

3. ### Use Named Returns For Local Variables Where It Is Possible

---

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419

```diff
-     function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {
+    function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint LUSDFee) {
```

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L432
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L724
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1123

---

4. ### Multiple Mapping Can be Combined In a Single Struct

- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L88
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L41
- https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L54

---

```diff

- mapping (address => uint) public totalStakesSnapshot;
- mapping (address => uint) public totalCollateralSnapshot;
+ struct Snapshot{
+  uint totalStakesSnapshot;
+  uint totalCollateralSnapshot;
+}
mapping (address => Snapshot) public snapshots;

```

---

5.  ### Using Storage Instead Of Memory For Structs/Arrays Saves Gas

    When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

        - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L284
        - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L518
        - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L105
        - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L687

6.  ### Avoid Emitting a Storage Variable When a Memory Value Is Available

    When they are the same, consider emitting the memory value instead of the storage value:

    ```diff
    guardianAddress = _newGuardianAddress;
    - emit GuardianAddressChanged(_newGuardianAddress);
    + emit GuardianAddressChanged(guardianAddress);

    ```
