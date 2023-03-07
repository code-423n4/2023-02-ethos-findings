## [G-1] Use elementary types or a user-defined type instead of a struct that has only one member

### Impact

Use elementary types or a user-defined type instead of a struct that has only one member.

### Findings

Total:1

[Ethos-Core/contracts/StabilityPool.sol#L174-L176](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/StabilityPool.sol#L174-L176)

```solidity
174:    struct Deposit {
175:        uint initialValue;
176:    }
```

## [G-2] Splitting require() statements that use `&&` saves gas

### Impact

When using `&&` in a `require()` statement, the entire statement will be evaluated before the transaction reverts. If the first condition is false, the second condition will not be evaluated. This means that if the first condition is false, the second condition will not be evaluated, which is a waste of gas. Splitting the `require()` statement into two separate statements will save gas.

### Findings

Total:3

[Ethos-Core/contracts/LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L352-L357)

```solidity
352:    require(
353:        !stabilityPools[_recipient] &&
354:        !troveManagers[_recipient] &&
355:        !borrowerOperations[_recipient],
356:            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:   );
```

[Ethos-Core/contracts/LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/LUSDToken.sol#L347-L351)

```solidity
347:    require(
348:        _recipient != address(0) &&
349:        _recipient != address(this),
350:            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:    );
```
[Ethos-Core/contracts/BorrowerOperations.sol#L653-L654](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)

```solidity
653:    require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:        "Max fee percentage must be between 0.5% and 100%");
```


## [G-3] Using `calldata` instead of memory for read-only arguments in external functions saves gas

### Impact

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a `for-loop` to copy each index of the `calldata` to the `memory` index. Each iteration of this `for-loop` costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead.<br><br> If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

### Findings

Total:1

[Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L65](https://github.com/code-423n4/2023-02-ethos/blob/main//Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L65)

```solidity
62:    function initialize(
63:        address _vault,
64:        address[] memory _strategists,
65:        address[] memory _multisigRoles,
```