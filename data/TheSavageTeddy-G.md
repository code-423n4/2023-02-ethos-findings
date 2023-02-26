# Using bitwise-not `~` instead of `-(i+1)` saves gas
Calculating `-(i+1)` instead of `~` costs **9 more gas**, because of extra instructions that achieve the same results.

`startIdx = uint(-(_startIdx + 1));` => `startIdx = uint(~_startIdx);`

There is 1 instance of this issue:
```
startIdx = uint(-(_startIdx + 1));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L44