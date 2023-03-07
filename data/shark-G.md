## 1. Using `storage` instead of `memory` for structs/arrays saves gas

When retrieving data from a `storage` location, assigning the data to a `memory` variable will cause all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incurring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct

For example, the following `memory` variable could be changed to `storage`:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L494

```solidity
        LocalVariables_report memory vars;
```

can be changed to:

```solidity
        LocalVariables_report storage vars;
```

## 2. Pack structs by putting data types that can be packed together next to each other.

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage (around 2000 gas per SLOT)

For example:

File: `CollateralConfig.sol` [Line 27-32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L32)

```diff
    struct Config {
        bool allowed;
-       uint256 decimals;
+       uint8 decimals;
        uint256 MCR;
        uint256 CCR;
    }
```

`decimals` can be changed to `uint8` as it is unlikely to need `uint256`. With that change, 4 SLOTs will be tightly packed into 3 SLOTs. This will save 1 SLOT.

## 3. Splitting `require()` statements that use && saves gas

Instead of using the && operator in a single require statement to check multiple conditions, using multiple require statements with 1 condition per require statement will save 3 GAS per &&.

Here are 2 instances of this issue:

- File: `BorrowerOperations.sol` [Line 653](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)
- File: `TroveManager.sol` [Line 1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)
