# Using bitwise-not `~` instead of `-(i+1)` saves gas
Calculating `-(i+1)` instead of `~` costs **9 more gas**, because of the extra `ADD` , `SUB` and `PUSH`s that can be replaced by `NOT` to achieve the same result.

`startIdx = uint(-(_startIdx + 1));` => `startIdx = uint(~_startIdx);`

*There is 1 instance of this issue:*
```solidity
File: MultiTroveGetter.sol
44:             startIdx = uint(-(_startIdx + 1));
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L44

# Check `Require()` / `Revert()` statements that use less gas first
Checks that cost less gas, such as checking function parameters, should be placed before more expensive checks such as ones that invoke `msg.sender`, because the function will be able to revert before checking more expensive statements. In this case, `require(msg.sender == vault, "Only vault can withdraw");` should be placed after the other `require()` statements which use less gas, to prevent wasting around **24680 gas**.

*There is 1 instance of this issue:*
```solidity
File: ReaperBaseStrategyv4.sol
95:     function withdraw(uint256 _amount) external override returns (uint256 loss) {
96:         require(msg.sender == vault, "Only vault can withdraw");
97:         require(_amount != 0, "Amount cannot be zero");
98:         require(_amount <= balanceOf(), "Ammount must be less than balance");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L98

# Expressions for constant values such as calling `KECCAK256()` should use `immutable` rather than `constant`
Constant values for function calls result in increased gas costs as `immutable`s are evaluated at deployment time, in contrast to `constants` which are executed at runtime for each invocation.

*There are 8 instances of this issue:*
```solidity
File: ReaperVaultV2.sol
73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:     bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76
```solidity
File: ReaperBaseStrategyv4.sol
49:     bytes32 public constant KEEPER = keccak256("KEEPER");
50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:     bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52

# Use `unchecked{}` for subtraction operations where operands cannot underflow because of a previous `if` statement
Using `unchecked{}` for operations that cannot underflow because of a previous `if` statement saves gas as it ignores the overflow/underflow checks.

*There are 15 instances of this issue:*
```solidity
File: MultiTroveGetter.sol
50:         if (startIdx >= sortedTrovesSize) {
51:             _troves = new CombinedTroveData[](0);
52:         } else {
53:             uint maxCount = sortedTrovesSize - startIdx; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/MultiTroveGetter.sol#L50-L53
```solidity
File: PriceFeed.sol
509:         uint price;
510:         if (_answerDigits >= TARGET_DIGITS) {
511:             // Scale the returned price value down to Liquity's target precision
512:             price = _price.div(10 ** (_answerDigits - TARGET_DIGITS)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
513:         }
514:         else if (_answerDigits < TARGET_DIGITS) {
515:             // Scale the returned price value up to Liquity's target precision
516:             price = _price.mul(10 ** (TARGET_DIGITS - _answerDigits)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L509-L516
```solidity
File: PriceFeed.sol
522:         uint256 price = _price;
523:         if (TARGET_DIGITS > TELLOR_DIGITS) {
524:             price = price.mul(10**(TARGET_DIGITS - TELLOR_DIGITS)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
525:         } else if (TARGET_DIGITS < TELLOR_DIGITS) {
526:             price = price.div(10**(TELLOR_DIGITS - TARGET_DIGITS)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L522-L526
```solidity
File: TroveManager.sol
492:         uint cappedCollPortion = _entireTroveDebt.mul(_MCR).div(_price);
493:         if (_collDecimals < LiquityMath.CR_CALCULATION_DECIMALS) {
494:             cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
495:         } else if (_collDecimals > LiquityMath.CR_CALCULATION_DECIMALS) {
496:             cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS)); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L492-L496
```solidity
File: ReaperStrategyGranarySupplyOnly.sol
79:         uint256 wantBalance = balanceOfWant();
80:         if (wantBalance > _debt) {
81:             uint256 toReinvest = wantBalance - _debt; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L79-L81
```solidity
File: ReaperStrategyGranarySupplyOnly.sol
91:         uint256 wantBal = balanceOfWant();
92:         if (wantBal < _amountNeeded) {
93:             _withdraw(_amountNeeded - wantBal); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L91-L93
```solidity
File: ReaperStrategyGranarySupplyOnly.sol
099:         if (_amountNeeded > liquidatedAmount) {
100:             loss = _amountNeeded - liquidatedAmount; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L099-L100
```solidity
File: ReaperStrategyGranarySupplyOnly.sol
File: ReaperStrategyGranarySupplyOnly.sol
131:         if (totalAssets > allocated) {
132:             uint256 profit = totalAssets - allocated; /// @audit use unchecked{} as it cannot underflow because of previous if statement
133:             toFree += profit;
134:             roi = int256(profit);
135:         } else if (totalAssets < allocated) {
136:             roi = -int256(allocated - totalAssets); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L131-L136
```solidity
File: ReaperVaultV2.sol
234:         if (stratCurrentAllocation > stratMaxAllocation) {
235:             return -int256(stratCurrentAllocation - stratMaxAllocation); /// @audit use unchecked{} as it cannot underflow because of previous if statement
236:         } else if (stratCurrentAllocation < stratMaxAllocation) {
237:             uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
238:             uint256 vaultCurrentAllocation = totalAllocated;
239: 
240:             if (vaultCurrentAllocation >= vaultMaxAllocation) {
241:                 return 0;
242:             }
243: 
244:             uint256 available = stratMaxAllocation - stratCurrentAllocation; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L234-L244
```solidity
File: ReaperVaultV2.sol
374:                 if (value <= vaultBalance) {
375:                     break;
376:                 }
377: 
378:                 address stratAddr = withdrawalQueue[i];
379:                 uint256 strategyBal = strategies[stratAddr].allocated;
380:                 if (strategyBal == 0) {
381:                     continue;
382:                 }
383: 
384:                 uint256 remaining = value - vaultBalance; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L374-L384
```solidity
File: ReaperVaultV2.sol
525:         if (vars.credit > vars.freeWantInStrat) {
526:             token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat); /// @audit use unchecked{} as it cannot underflow because of previous if statement
527:         } else if (vars.credit < vars.freeWantInStrat) {
528:             token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L525-L528
```solidity
File: ReaperVaultV2.sol
534:         if (vars.lockedProfitBeforeLoss > vars.loss) {
535:             lockedProfit = vars.lockedProfitBeforeLoss - vars.loss; /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L534-L535
```solidity
File: ReaperBaseStrategyv4.sol
119:             uint256 amountFreed = _liquidateAllPositions();
120:             if (amountFreed < debt) {
121:                 roi = -int256(debt - amountFreed); /// @audit use unchecked{} as it cannot underflow because of previous if statement
122:             } else if (amountFreed > debt) {
123:                 roi = int256(amountFreed - debt); /// @audit use unchecked{} as it cannot underflow because of previous if statement
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L119-L123

# For state variables, `<x> += <y>` costs more gas than `<x> = <x> + <y>`

This saves around **13 gas** for each execution.

*There are 8 instances of this issue:*
```solidity
totalAllocBPS += _allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168
```solidity
totalAllocBPS -= strategies[_strategy].allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194
```solidity
totalAllocBPS += _allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196
```solidity
totalAllocBPS -= strategies[_strategy].allocBPS;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214
```solidity
totalAllocBPS -= bpsChange;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445
```solidity
totalAllocated -= loss;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452
```solidity
totalAllocated -= vars.debtPayment;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515
```solidity
totalAllocated += vars.credit;
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521

# Use a more recent version of Solidity
Using `^0.8.0` as opposed to `0.6.11`, which is used in many contracts, saves gas as several operations are optimized in the newer versions of Solidity. For example, accessing data in `calldata`, reduced gas cost when emitting event data and the introduction of `immutable` state variables save gas.
