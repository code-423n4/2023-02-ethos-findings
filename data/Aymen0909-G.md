# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | Variables inside struct should be packed to save gas  | 2  |
| 2  | `storage` variable should be cached into `memory` variables instead of re-reading them  |  6 |
| 3  | Use `unchecked` blocks to save gas  |  5 |
| 4  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  9 |


## Findings

### 1- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There are 2 instance of this issue:

File: interfaces/vault/IVaultRegistry.sol [Line 7-22](https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L7-L22)

```solidity
struct Config {
    bool allowed;
    uint256 decimals;
    uint256 MCR;
    uint256 CCR;
}
```

As the `MCR` and `CCR` variables values can't really overflow **2^96** and the `decimals` value (usually less than 18) can't overflow **2^8**, so their values should be packed in order to save gas, the struct should be modified as follow :

```solidity
struct Config {
    bool allowed;
    uint8 decimals;
    uint96 MCR;
    uint96 CCR;
}
```


File: ReaperVaultV2.sol [Line 25-33](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L25-L33)

```solidity
struct StrategyParams {
    uint256 activation; // Activation block.timestamp
    uint256 feeBPS; // Performance fee taken from profit, in BPS
    uint256 allocBPS; // Allocation in BPS of vault's total assets
    uint256 allocated; // Amount of capital allocated to this strategy
    uint256 gains; // Total returns that Strategy has realized for Vault
    uint256 losses; // Total losses that Strategy has realized for Vault
    uint256 lastReport; // block.timestamp of the last time a report occured
}
```

Because the `feeBPS` and `allocBPS` variables values are always less than `10000` and as the variables `activation` and `lastReport` are representing time values they can't really overflow **2^64**, so their values should be packed together in order to save gas, the struct should be modified as follow :

```solidity
struct StrategyParams {
    uint256 allocated; // Amount of capital allocated to this strategy
    uint256 gains; // Total returns that Strategy has realized for Vault
    uint256 losses; // Total losses that Strategy has realized for Vault
    uint64 activation; // Activation block.timestamp
    uint64 lastReport; // block.timestamp of the last time a report occured
    uint64 feeBPS; // Performance fee taken from profit, in BPS
    uint64 allocBPS; // Allocation in BPS of vault's total assets
}
```

    
### 2- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 6 instances of this issue :

File: CommunityIssuance.sol [Line 86-91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86-L91)

In the code linked above the variable `lastDistributionTime` is read multiple times from storage and because its value won't be changed in those lines of code, `lastDistributionTime` should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

File: ActivePool.sol [Line 278-283](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L278-L283)

In the code linked above the values of `yieldGenerator[_collateral]` is read multiple times (more than 3) from storage and its respective values does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 3- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 5 instances of this issue:

File: ReaperVaultV2.sol [Line 244](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L244)
```solidity
uint256 available = stratMaxAllocation - stratCurrentAllocation;
```

In code linked above both operation cannot underflow due to the checks [Line 236](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L236) so it should be marked as `unchecked`. 


File: ReaperVaultV2.sol [Line 384](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L384)
```solidity
uint256 remaining = value - vaultBalance;
```

In code linked above the operation cannot underflow because the of the checks [Line 374](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L374) so it should be marked as `unchecked`.


File: ReaperStrategyGranarySupplyOnly.sol [Line 81](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81)
```solidity
uint256 toReinvest = wantBalance - _debt;
```

In code linked above the operation cannot underflow because the of the checks that preceeds it [Line 80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L80) so it should be marked as `unchecked`.


File: ReaperStrategyGranarySupplyOnly.sol [Line 100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100)
```solidity
loss = _amountNeeded - liquidatedAmount;
```

In code linked above the operation cannot underflow because the of the checks that preceeds it [Line 99](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L99) so it should be marked as `unchecked`.


File: ReaperStrategyGranarySupplyOnly.sol [Line 132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L132)
```solidity
uint256 profit = totalAssets - allocated;
```

In code linked above the operation cannot underflow because the of the checks that preceeds it [Line 131](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L131) so it should be marked as `unchecked`.


### 4- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 9 instances of this issue:

```solidity
File: ReaperVaultV2.sol

168     totalAllocBPS += _allocBPS;
194     totalAllocBPS -= strategies[_strategy].allocBPS;
196     totalAllocBPS += _allocBPS;
214     totalAllocBPS -= strategies[_strategy].allocBPS;
396     totalAllocated -= actualWithdrawn;
445     totalAllocBPS -= bpsChange;
452     totalAllocated -= loss;
515     totalAllocated -= vars.debtPayment;
521     totalAllocated += vars.credit;
```