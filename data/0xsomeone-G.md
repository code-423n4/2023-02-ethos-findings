# Ethos Reserve Gas Optimization Report

This submission serves as a gas optimization report of the Ethos Reserve codebase. The ways the codebase can be optimized are as follows:

## `CommunityIssuance.sol`

### CIE-01G: Multiple Redundant Repetitive Storage Reads (Affected Lines: [L86](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86), [L87](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87), [L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88), [L106](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L106), [L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107), [L108](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L108), [L112-L113](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112-L113), [L116](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L116))

The referenced statements read the same value from the contract's storage multiple times in the same function context.

We advise them to instead cache the variable's value to a local in-memory variable instead, reducing the total warm `SLOAD` operations performed in the codebase.

### CIE-02G: Redundant Usage of `SafeMath` (Affected Lines: [L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L88), [L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L107))

The referenced operations are guaranteed to be performed safely by the logical constraints (i.e. `if` clauses) that precede their execution.

We advise the usage of `SafeMath` in these instances to be omitted, optimizing their gas cost.

### CIE-03G: Unconditional Emission of Event (Affected Lines: [L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L94))

The referenced `TotalOATHIssuedUpdated` event is emitted regardless of whether the `totalOATHIssued` was actually adjusted.

We advise it to be relocated within the `if` clause, removing the potentially redundant `SLOAD` operation and ensuring the event is emitted optimally.

## `LQTYStaking.sol`

### LSG-01G: Multiple Redundant Repetitive Storage Reads (Affected Lines: [L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L125), [L160](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L160))

The referenced statements read the same value from the contract's storage multiple times in the same function context.

We advise them to instead cache the variable's value to a local in-memory variable instead, reducing the total warm `SLOAD` operations performed in the codebase.

### LSG-02G: Redundant Usage of `SafeMath` (Affected Lines: [L155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L155), [L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L181))

The referenced operations are guaranteed to be performed safely by the logical constraints (i.e. `if` clauses) that precede their execution.

We advise the usage of `SafeMath` in these instances to be omitted, optimizing their gas cost.

### LSG-03G: Optimal Execution of Functions (Affected Lines: [L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L181), [L191](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L191))

The referenced `if` conditionals will cause their function to result in a no-op if they are not satisfied.

As such, we advise an `else` clause to be introduced to them that immediately stops the function's execution (i.e. `return`), optimizing their gas cost significantly.

### LSG-04G: Inefficient Mapping Entry Usages (Affected Lines: [L208-L209](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L208-L209), [L230](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L230), [L234](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L234))

The referenced `mapping` variable usages do so using direct accessors (i.e. `foo[bar][zoo]`) instead of storing the `mapping` lookups to interim `memory` / `storage` variables.

We advise the lookups to be cached to local variables as described above, significantly optimizing their usage cost as each `mapping` lookup performs a `keccak256` operation under-the-hood to assess the storage offset of the entry each time.

## `ReaperBaseStrategyv4.sol`

### RBS-01G: Inefficient Unchecked Increments (Affected Lines: [L77](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L77))

The referenced lines perform unchecked increments of loop iterators, however, this is achieved via library calls which are `delegatecall` instructions at the EVM level.

We advise them to be performed directly, avoiding the unnecessary gas overhead of `delegatecall` instructions the current paradigm incurs.

### RBS-02G: Inefficient Checked Arithmetic (Affected Lines: [L121](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L121), [L123](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L123))

The referenced statements perform Solidity's built-in safe arithmetics redundantly as their safety is guaranteed by the logical constraints (i.e. `if` clauses) that precede their execution.

We advise the usage of `unchecked` code blocks in these instances, optimizing their gas cost.

### RBS-03G: Inefficient Calculation of Repayment (Affected Lines: [L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128))

Based on the calculations in `ReaperBaseStrategyv4::harvest`, the value of `repayment` will be equal to `amountFreed` if the `roi` is negative. As such, we advise the assignment to be made directly to `amountFreed` minimizing the calculations within the code.

# Parting Note

More optimizations were identified, however, they were not able to be transcribed into legible text in time for the contest's duration. I am more than happy to provide them manually for the sponsor to review and consume within 24 hours of the contest's end.