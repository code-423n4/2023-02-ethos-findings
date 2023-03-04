# Multiple storage reads from `yieldGenerator[collateral]`

In the [`_rebalance`](https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L239) function, we make multiple storage reads from `yieldGenerator[collateral]`. Storage is read in the following lines:
1. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L246
2. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L279
3. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L280
4. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L282

The address has already been stored in memory [here](https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L243) and the `IERC4626` vault has also been stored in memory [here](https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L246). By reading from memory instead of storage, we can save users gas.

# No need to recalculate `vars.yieldingAmount`

We calculate 
`yieldingAmount = yieldingAmount - toWithdraw`

However, 
`yieldingAmount = currentAllocated`

We recalculate `yieldingAmount` [here](https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L267) as 
`yieldingAmount = yieldingAmount - toWithdraw`

where `toWithdraw = currentAllocated - finalYieldingAmount`

Substituting values for `toWithdraw`,

`yieldingAmount = currentAllocated - currentAllocated + finalYieldingAmount`
`yieldingAmount = finalYieldingAmount`

Thus, these extra computation steps are unnecessary and only waste gas. I suggest replacing this (line)[https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L267] with 
`vars.yieldingAmount = vars.finalYieldingAmount`

This same logic can be applied to the calculation (here)[https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/ActivePool.sol#L272]

# Adjust trove should use `>` instead of `!=`
In the conditional check (here)[https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/BorrowerOperations.sol#L269], we can save 3 gas by using `>` instead of `!=` since `_collTopUp` is a `uint` and thus must be `>= 0`

# Prevent passing `LUSD_GAS_COMPENSATION` constant around
We are passing around the constant `LUSD_GAS_COMPENSATION` instead of just using the value where it is needed directly. This defeats some of the benefit of constants where the compiler will directly replace the values in code. The value is used in these locations

1. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L342
2. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L379
3. https://github.com/code-423n4/2023-02-ethos/blob/52aba524ede2e9becff9b8f0025b863c1029adac/Ethos-Core/contracts/TroveManager.sol#L500

I suggest not saving this value into memory and instead using the constant directly where it is needed.

