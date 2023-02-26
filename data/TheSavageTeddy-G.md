# Using bitwise-not `~` instead of `-(i+1)` saves gas
Calculating `-(i+1)` instead of `~` costs **9 more gas**, because of the extra `ADD` , `SUB` and `PUSH`s that can be replaced by `NOT` to achieve the same result.

`startIdx = uint(-(_startIdx + 1));` => `startIdx = uint(~_startIdx);`

There is 1 instance of this issue:
```
startIdx = uint(-(_startIdx + 1));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L44

# For state variables, `<x> += <y>` costs more gas than `<x> = <x> + <y>`
There are 7 instances of this issue:
```
totalAllocBPS += _allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168
```
totalAllocBPS -= strategies[_strategy].allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194
```
totalAllocBPS += _allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196
```
totalAllocBPS -= bpsChange;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445
```
totalAllocated -= loss;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452
```
totalAllocated -= vars.debtPayment;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515
```
totalAllocated += vars.credit;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521