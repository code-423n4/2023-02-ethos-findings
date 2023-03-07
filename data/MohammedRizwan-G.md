## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:| 
| [G&#x2011;01] | require() or revert() statements that check input arguments should be at the top of the function | 2 |
| [G&#x2011;02] | Splitting require() statements that use && saves gas | 4 |
| [G&#x2011;03] | [Ethos-Core contracts] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow  | 11 |
| [G&#x2011;04] | Avoid compound assignment operator in state variables | 21 |
| [G&#x2011;05] | Use require instead of assert | 17 |
| [G&#x2011;06] | Use mapping instead of arrays | - |
| [G&#x2011;07] | Use a safe and secure smart contract libraries | - |
| [G&#x2011;08] | Use a more recent version of solidity | 8 |

## Gas Optimizations
### [G&#x2011;01]  require() or revert() statements that check input arguments should be at the top of the function
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

In CollateralConfig.initialize( ) function, below checks should be at top which will check and verify the conditions and prevent any gas wastage and function failue.
There are 2 instances of this issue.

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

66            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");

---

69            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");

```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L66
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L69

### [G&#x2011;02]  Splitting require() statements that use && saves gas
&& Operator takes more gas, by splitting it in require() it Saves 16 gas per instance. If the gas Optimizer used at 200, instead of using the && operator in a single require statement to check multiple conditions, multiple require statements with 1 condition per require statement should be used to save gas.
There are 4 instances of this issue.

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

653         require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L653

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1539        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1539

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

348            _recipient != address(0) && 
349            _recipient != address(this),
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L348-L349

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

353            !stabilityPools[_recipient] &&
354            !troveManagers[_recipient] &&
355            !borrowerOperations[_recipient],
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L353-L355

### [G&#x2011;03]  [Ethos-Core contracts] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow 
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop. 
Issue:- Ethos-Core contracts uses solidity version 0.6.11. By using solidity version 0.8.0, the contracts can use unchecked keyword which can save considerable gas. 
There are 11 instances of this issue.

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56        for(uint256 i = 0; i < _collaterals.length; i++) { 
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56

```solidity
File: Ethos-Core/contracts/ActivePool.sol
         
108        for(uint256 i = 0; i < numCollaterals; i++) { 
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L108

```solidity
File: Ethos-Core/contracts/StabilityPool.sol
         
351        for (uint i = 0; i < numCollaterals; i++) { 
---

397        for (uint i = 0; i < numCollaterals; i++) { 
---

640        for (uint i = 0; i < assets.length; i++) { 
---

810        for (uint i = 0; i < collaterals.length; i++) { 
---

831        for (uint i = 0; i < collaterals.length; i++) { 
---

859        for (uint i = 0; i < numCollaterals; i++) { 
---
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L397
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L640
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L810
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L831
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L859

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
                                                          
206        for (uint i = 0; i < assets.length; i++) { 
---

228        for (uint i = 0; i < collaterals.length; i++) { 
---

240        for (uint i = 0; i < numCollaterals; i++) { 
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240

### [G&#x2011;04]  Avoid compound assignment operator in state variables
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).
There are 21 instances of this issue.

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

194        totalAllocBPS -= strategies[_strategy].allocBPS;
---

196        totalAllocBPS += _allocBPS;
---

214        totalAllocBPS -= strategies[_strategy].allocBPS;
---

390        value -= loss;
391        totalLoss += loss;214;
---

395        strategies[stratAddr].allocated -= actualWithdrawn;
396        totalAllocated -= actualWithdrawn;
---

444        stratParams.allocBPS -= bpsChange;
445        totalAllocBPS -= bpsChange;
---

450        stratParams.losses += loss;
451        stratParams.allocated -= loss;
452        totalAllocated -= loss;
---

505        strategy.gains += vars.gain;
---

514        strategy.allocated -= vars.debtPayment;
515        totalAllocated -= vars.debtPayment;
516        vars.debt -= vars.debtPayment; // tracked for return value
---

520        strategy.allocated += vars.credit;
521        totalAllocated += vars.credit;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L194
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L196
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L214
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L390-L391
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L395-L396
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L445
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L450-L452
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L505
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L514-L516
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L520-L521

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

133        toFree += profit;
---

141        roi -= int256(loss);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L141

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

128        repayment -= uint256(-roi);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L128

### [G&#x2011;05]  Use require instead of assert
The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.
They are quite similar as both check for conditions and if they are not met, would throw an error.
The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made. It does not refund gas. 
Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.
There are 17 instances of this issue.

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

128        assert(MIN_NET_DEBT > 0);
---

197        assert(vars.compositeDebt > 0);
---

301        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
---

331        assert(_collWithdrawal <= vars.coll); 
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L128
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L197
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L301
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L331

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1279       assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
---

1342        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
---

1348        assert(index <= idxLast);
---

1489        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1279
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1342
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1348
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1489

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

526        assert(_debtToOffset <= _totalLUSDDeposits);
---

551        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
---

591        assert(newP > 0);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L526
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L591

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

312        assert(sender != address(0));
313        assert(recipient != address(0));
---

321        assert(account != address(0));
---

329        assert(account != address(0));
---

337        assert(owner != address(0));
338        assert(spender != address(0));
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L312-L313
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L321
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L329
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L337-L338

### [G&#x2011;06]  Use mapping instead of arrays
1- Instead of arrays, mapping can be used to store the collaterals. Use of mapping shall be more gas optimized as looping of array is gas expensive.Arrays are packable and iterable, mappings are less expensive. It is advised to use mappings to manage lists of data in order to conserve gas. This is beneficial for both memory and storage. An integer index can be used as a key in a mapping to control an ordered list. Another advantage of mappings is that you can access any value without having to iterate through an array as would otherwise be necessary.
2- Avoid looping through length arrays, not only it will consume a lot of gas but if gas prices rise too much, it 
   can prevent your contract from being carried out beyond the block gas limit.

### [G&#x2011;07]  Use a safe and secure smart contract libraries 
It is seen that ALL Ethos-Core contracts with compiler version 0.6.11 uses own libraries as dependencies. It is recommended to use openzeppelin Libraries as they most secure and safe also these libraries are stateless contracts that don't store any state. when the public function of a library from the contract is called, the bytecode of that function doesn't get deployed with the smart  contract, and thus some gas cost can be saved BUT, If the libraries are written during the project, say own libraries as it is in this project case, These libraries need to be deployed them and shall be required to pay one time gas fee for deployement. If libraries from OpenZeppelin  are used, they will not add to your deployment cost.

### [G&#x2011;08]  Use a more recent version of solidity 
Recommend to use solidity compiler version ^0.8.0. This will safeguard from underflow and overflow conditions. This modification will make the contracts more gas optimized. safeMath library wont be explicitely required as ^0.8.0 version has already taken  the underflow and overflow conditions by default.
1- Use a solidity version of at least 0.8 to get default underflow/overflow checks.
2- Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining. 
3- Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.
4- Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than 
   revert()/require() strings.
5- Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

There are 8 instances of this issue.

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L3

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L3

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3      pragma solidity 0.6.11;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3
