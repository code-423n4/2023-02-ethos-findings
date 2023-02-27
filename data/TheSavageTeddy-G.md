# Using bitwise-not `~` instead of `-(i+1)` saves gas
Calculating `-(i+1)` instead of `~` costs **9 more gas**, because of the extra `ADD` , `SUB` and `PUSH`s that can be replaced by `NOT` to achieve the same result.

`startIdx = uint(-(_startIdx + 1));` => `startIdx = uint(~_startIdx);`

There is 1 instance of this issue:
```
startIdx = uint(-(_startIdx + 1));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L44

# For state variables, `<x> += <y>` costs more gas than `<x> = <x> + <y>`
There are 8 instances of this issue:
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
totalAllocBPS -= strategies[_strategy].allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214
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

# Check `Require()` / `Revert()` statements that use less gas first
Checks that cost less gas, such as checking function parameters, should be placed before more expensive checks such as ones that invoke `msg.sender`, because the function will be able to revert before checking more expensive statements. In this case, `require(msg.sender == vault, "Only vault can withdraw");` should be placed after the other `require()` statements which use less gas. This saves roughly **24680 gas**.

```
    function withdraw(uint256 _amount) external override returns (uint256 loss) {
        require(msg.sender == vault, "Only vault can withdraw");
        require(_amount != 0, "Amount cannot be zero");
        require(_amount <= balanceOf(), "Ammount must be less than balance");
```

# Use a more recent version of Solidity
Using `^0.8.0` as opposed to `0.6.11` saves gas as several operations are optimized in the newer versions of Solidity.

