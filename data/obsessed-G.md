# Report

## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | Yield distribution parameters can easily be stored in type uint16 | 4 |
### [GAS-1] Yield distribution parameters can easily be stored in type uint16
Yield distribution and yieldingPercentageDrift variables according to comments and documentation can easily be stored in type uint16 and thus be stored together in the same slot to save gas. Moreover if uint16 state variable will be called and later on we try to access another uint16 state variable (which is in the same storage slot) it will be treated as a warm storage (even if its our first contact with that variable) and thus it will cost only 100 gas instead of 2100 gas to use it for the first time.

*Instances (4)*:
```solidity
File: ActivePool.sol

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L49-L54

    uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS

    // Yield distribution params, must add up to 10k
    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
    uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

```