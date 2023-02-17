# Gas Optimization Issues

| ID  | Issue | Contexts | Instances | Total Gas Saved |
| --- | ----- | -------- | --------- | --------------- |
| [G-01](#g-01-using-bools-for-storage-incurs-overhead) | Using bools for storage incurs overhead| 4 | 9 | 153900 |
| [G-02](#g-02-array-length-not-cached-outside-of-loop) | Array length not cached outside of loop | 4 | 8 | 24 |
| [G-03](#g-03-variables-initialized-with-default-value) | Variables initialized with default value | 7 | 22 | - |
| [G-04](#g-04-unsigned-integer-comparison-with--0) | Unsigned integer comparison with > 0 | 6 | 26 | - |
| [G-05](#g-05-long-revertrequire-strings) | Long revert/require strings | 3 | 7 | - |
| [G-06](#g-06-postfix-incrementdecrease-used) | Postfix increment/decrease used | 7 | 17 | - |
| [G-07](#g-07-mark-payable-functions-guaranteed-to-revert-when-called-by-normal-users) | Mark payable functions guaranteed to revert when called by normal users | 2 | 7 | 147 |
| [G-08](#g-08-use-assembly-to-check-for-address0) | Use assembly to check for address(0) | 5 | 15 | 90 |
| [G-09](#g-09-use-require-instead-of-assert-when-possible) | Use "require" instead of "assert" when possible | 4 | 20 | - |
| [G-10](#g-10-usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead) | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 2 | 2 | - |
| [G-11](#g-11-split-require-statements-that-use--to-save-gas) | Split require() statements that use && to save gas | 1 | 1 | - |
| [G-12](#g-12-x--y-costs-more-gas-than-x--x--y-for-state-variables) | x += y costs more gas than x = x + y for state variables | 3 | 22 | 2486 |
| [G-13](#g-13-when-possible-use-non-strict-comparison--andor--instead-of--) | When possible, use non-strict comparison >= and/or =< instead of > < | 9 | 64 | 960 |
| [G-14](#g-14-if-possible-use-private-rather-than-public-for-constants) | If possible, use private rather than public for constants | 10 | 30 | 102180 |

| Total issues | Total contexts | Total instances | Total minimum gas saved |
| ------------ | -------------- | --------------- | ----------------------- |
| 14           | 68             | 250             | 259787                  |

## [G-01] Using bools for storage incurs overhead

### Description

Use uint256 for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

### Findings

- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::54 =>         bool isCollIncrease;
  ::182 =>         bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
  ::290 =>         bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::28 =>         bool allowed;
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::62 =>     mapping (address => bool) public troveManagers;
  ::63 =>     mapping (address => bool) public stabilityPools;
  ::64 =>     mapping (address => bool) public borrowerOperations;
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::135 =>         bool recoveryModeAtStart;
  ::152 =>         bool backToNormalMode;
  ```

### References
- [OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27)



## [G-02] Array length not cached outside of loop

### Description

Caching the length eliminates the additional `DUP<N>` required to store the stack offset and converts each of them to a `DUP<N>` (3 gas).

### Findings

- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::56 =>         for(uint256 i = 0; i < _collaterals.length; i++) {
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::206 =>         for (uint i = 0; i < assets.length; i++) {
  ::228 =>         for (uint i = 0; i < collaterals.length; i++) {
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::640 =>         for (uint i = 0; i < assets.length; i++) {
  ::810 =>             for (uint i = 0; i < collaterals.length; i++) {
  ::831 =>         for (uint i = 0; i < collaterals.length; i++) {
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::808 =>         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
  ::882 =>         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
  ```

### References
- [[G‑14] `<array>`.length should not be looked up in every loop of a for-loop](https://code4rena.com/reports/2022-12-backed/#g14--arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)


## [G-03] Variables initialized with default value

### Description

Explicitly initialize a variable with its default value wastes gas

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::108 =>         for(uint256 i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::56 =>         for(uint256 i = 0; i < _collaterals.length; i++) {
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::206 =>         for (uint i = 0; i < assets.length; i++) {
  ::228 =>         for (uint i = 0; i < collaterals.length; i++) {
  ::240 =>         for (uint i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::351 =>         for (uint i = 0; i < numCollaterals; i++) {
  ::397 =>         for (uint i = 0; i < numCollaterals; i++) {
  ::640 =>         for (uint i = 0; i < assets.length; i++) {
  ::810 =>             for (uint i = 0; i < collaterals.length; i++) {
  ::831 =>         for (uint i = 0; i < collaterals.length; i++) {
  ::859 =>         for (uint i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::117 =>         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
  ::165 =>         for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::128 =>         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
  ::264 =>         for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
  ::369 =>             uint256 totalLoss = 0;
  ::371 =>             uint256 vaultBalance = 0;
  ::372 =>             for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::77 =>         for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
  ::100 =>         uint256 amountFreed = 0;
  ::112 =>         uint256 debt = 0;
  ::117 =>         uint256 repayment = 0;
  ```

### References
- [G001 - Don't Initialize Variables with Default Value](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)
- [Solidity tips and tricks to save gas and reduce bytecode size](https://mudit.blog/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size/)

## [G-04] Unsigned integer comparison with > 0

### Description

Comparisons done using `!= 0` are cheaper than `> 0` when dealing with unsigned integer types

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::278 =>         if (vars.netAssetMovement > 0) {
  ```
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::128 =>         assert(MIN_NET_DEBT > 0);
  ::197 =>         assert(vars.compositeDebt > 0);
  ::301 =>         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
  ::337 =>         if (!_isDebtIncrease && _LUSDChange > 0) {
  ::552 =>         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::152 =>         if (_LQTYamount > 0) {
  ::181 =>         if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
  ::191 =>         if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
  ::266 =>         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
  ::270 =>         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::591 =>         assert(newP > 0);
  ::870 =>         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
  ::874 =>         require(_amount > 0, 'StabilityPool: Amount must be non-zero');
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::424 =>             if (singleLiquidation.collSurplus > 0) {
  ::452 =>         if (_LUSDInStabPool > 0) {
  ::551 =>         require(totals.totalDebtInSequence > 0);
  ::563 =>         if (totals.totalCollSurplus > 0) {
  ::754 =>         require(totals.totalDebtInSequence > 0);
  ::766 =>         if (totals.totalCollSurplus > 0) {
  ::916 =>         if (_LUSD > 0) {
  ::920 =>         if (_collAmount > 0) {
  ::1224 =>             assert(totalStakesSnapshot[_collateral] > 0);
  ::1414 =>         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::502 =>         } else if (_roi > 0) {
  ::518 =>         } else if (vars.available > 0) {
  ```

### References
- [G003 - Use != 0 instead of > 0 for Unsigned Integer Comparison](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)


## [G-05] Long revert/require strings

### Description

Strings in `require()` / `revert()` longer than 32 bytes cost extra gas

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::107 =>         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
  ::133 =>         require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::150 =>         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
  ::321 =>         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
  ::436 =>         require(loss <= allocation, "Strategy loss cannot be greater than allocation");
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::98 =>         require(_amount <= balanceOf(), "Ammount must be less than balance");
  ::193 =>             "Upgrade cooldown not initiated or still ongoing"
  ```

### References
- [require() revert() Strings Longer Than 32 Bytes Cost Extra Gas](https://code4rena.com/reports/2022-12-caviar#g-03-requirerevert-strings-longer-than-32-bytes-cost-extra-gas)


## [G-06] Postfix increment/decrease used

### Description

The prefix increment / decrease expression returns the updated value after it's incremented while the postfix increment / decrease expression returns the original value.

Be careful when employing this optimization anytime the return value of the expression is utilized later; for instance, `uint a = i++` and `uint a = ++i` produce different values for `a`.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::108 =>         for(uint256 i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::56 =>         for(uint256 i = 0; i < _collaterals.length; i++) {
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::206 =>         for (uint i = 0; i < assets.length; i++) {
  ::228 =>         for (uint i = 0; i < collaterals.length; i++) {
  ::240 =>         for (uint i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::286 =>                          _nonces[owner]++, deadline))));
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::351 =>         for (uint i = 0; i < numCollaterals; i++) {
  ::397 =>         for (uint i = 0; i < numCollaterals; i++) {
  ::640 =>         for (uint i = 0; i < assets.length; i++) {
  ::810 =>             for (uint i = 0; i < collaterals.length; i++) {
  ::831 =>         for (uint i = 0; i < collaterals.length; i++) {
  ::859 =>         for (uint i = 0; i < numCollaterals; i++) {
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::608 =>         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
  ::690 =>         for (vars.i = 0; vars.i < _n; vars.i++) {
  ::808 =>         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
  ::882 =>         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
  ```
- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
  ```Solidity
  ::273 =>         if (x % y != 0) q++;
  ```

### References

- [Why does ++i cost less gas than i++?](https://ethereum.stackexchange.com/questions/133161/why-does-i-cost-less-gas-than-i#answer-133164)
- [Gas Optimizations for the Rest of Us](https://m1guelpf.blog/d0gBiaUn48Odg8G2rhs3xLIjaL8MfrWReFkjg8TmDoM)


## [G-07] Mark payable functions guaranteed to revert when called by normal users

### Description

If a function modifier, like onlyOwner, is applied, the function will revert if a regular user attempts to pay it. Making the function payable will save valid callers money on gas since the compiler won't do checks to see if a payment was made.

The extra opcodes avoided are `CALLVALUE(2)`, `DUP1(3)`, `ISZERO(3)`, `PUSH2(3)`, `JUMPI(10)`, `PUSH1(3)`, `DUP1(3)`, `REVERT(0)`, `JUMPDEST(1)`, `POP(2)` which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::125 =>     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
  ::132 =>     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
  ::138 =>     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
  ::144 =>     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
  ::214 =>     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
  ```
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
  ```Solidity
  ::101 =>     function fund(uint amount) external onlyOwner {
  ::120 =>     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
  ```

### References
- [[G‑11]  Functions guaranteed to revert when called by normal users can be marked](https://code4rena.com/reports/2022-12-backed/#g11--functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)


## [G-08] Use assembly to check for address(0)

### Description

By checking for `address(0)` using assembly language, you can avoid the use of more gas-expensive operations such as calling a smart contract or reading from storage. This can save 6 gas per instance.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::93 =>         require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::312 =>         assert(sender != address(0));
  ::313 =>         assert(recipient != address(0));
  ::321 =>         assert(account != address(0));
  ::325 =>         emit Transfer(address(0), account, amount);
  ::329 =>         assert(account != address(0));
  ::333 =>         emit Transfer(account, address(0), amount);
  ::337 =>         assert(owner != address(0));
  ::338 =>         assert(spender != address(0));
  ::348 =>             _recipient != address(0) &&
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::294 =>         owner = address(0);
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::167 =>             require(step[0] != address(0));
  ::168 =>             require(step[1] != address(0));
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::151 =>         require(_strategy != address(0), "Invalid strategy address");
  ::629 =>         require(newTreasury != address(0), "Invalid address");
  ```
  
### References
- [EVM Codes](https://www.evm.codes/)


## [G-09] Use "require" instead of "assert" when possible

### Description

If `assert()` returns a false assertion, it compiles to the invalid opcode `0xfe`, which eats up all the gas left and completely undoes the changes. `require()` compiles to `0xfd`, which is the opcode for a `REVERT`, indicating that it will return the remaining gas if it returns a false assertion.

### Findings

- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::128 =>         assert(MIN_NET_DEBT > 0);
  ::197 =>         assert(vars.compositeDebt > 0);
  ::301 =>         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
  ::331 =>         assert(_collWithdrawal <= vars.coll);
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::312 =>         assert(sender != address(0));
  ::313 =>         assert(recipient != address(0));
  ::321 =>         assert(account != address(0));
  ::329 =>         assert(account != address(0));
  ::337 =>         assert(owner != address(0));
  ::338 =>         assert(spender != address(0));
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::526 =>         assert(_debtToOffset <= _totalLUSDDeposits);
  ::551 =>         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
  ::591 =>         assert(newP > 0);
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::417 =>             assert(_LUSDInStabPool != 0);
  ::1224 =>             assert(totalStakesSnapshot[_collateral] > 0);
  ::1279 =>         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
  ::1342 =>         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
  ::1348 =>         assert(index <= idxLast);
  ::1414 =>         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
  ::1489 =>         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
  ```

### References
- [Assert() vs Require() in Solidity – Key Difference & What to Use](https://codedamn.com/news/solidity/assert-vs-require-in-solidity)


## [G-10] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

### Description

Gas consumption can be greater if you use items that are less than 32 bytes in size. This is such that the EVM can only handle 32 bytes at once. In order to increase the element's size from 32 bytes to the necessary amount, the EVM must do extra operations if it is lower than that. When necessary, it is advised to utilize a bigger size and then downcast.

### Findings

- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::35 =>     uint8 constant internal _DECIMALS = 18;
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::35 =>     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
  ```

### References
- [Layout of State Variables in Storage | Solidity docs](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html#layout-of-state-variables-in-storage)
- [GAS OPTIMIZATIONS ISSUES by Gokberk Gulgun](https://hackmd.io/@W1m6lTsFT5WAy9C_lRTX_g/rkr5Laoys)


## [G-11] Split require() statements that use && to save gas

### Description

It is advised to use two `require` instead of using one `require` with the operator `&&` to save gas.

### Findings

- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::653 =>             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
  ```

### References
- [[G-04] SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS](https://code4rena.com/reports/2022-12-caviar/#g-04-splitting-require-statements-that-use--saves-gas)


## [G-12] x += y costs more gas than x = x + y for state variables

### Description

Gas can be saved by substituting the addition operator with plus-equals, same for minus. Saves 113 gas per instance.

### Findings

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::133 =>             toFree += profit;
  ::141 =>         roi -= int256(loss);
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::168 =>         totalAllocBPS += _allocBPS;
  ::194 =>         totalAllocBPS -= strategies[_strategy].allocBPS;
  ::196 =>         totalAllocBPS += _allocBPS;
  ::214 =>         totalAllocBPS -= strategies[_strategy].allocBPS;
  ::390 =>                     value -= loss;
  ::391 =>                     totalLoss += loss;
  ::395 =>                 strategies[stratAddr].allocated -= actualWithdrawn;
  ::396 =>                 totalAllocated -= actualWithdrawn;
  ::444 =>                 stratParams.allocBPS -= bpsChange;
  ::445 =>                 totalAllocBPS -= bpsChange;
  ::450 =>         stratParams.losses += loss;
  ::451 =>         stratParams.allocated -= loss;
  ::452 =>         totalAllocated -= loss;
  ::505 =>             strategy.gains += vars.gain;
  ::514 =>                 strategy.allocated -= vars.debtPayment;
  ::515 =>                 totalAllocated -= vars.debtPayment;
  ::516 =>                 vars.debt -= vars.debtPayment; // tracked for return value
  ::520 =>             strategy.allocated += vars.credit;
  ::521 =>             totalAllocated += vars.credit;
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::128 =>                 repayment -= uint256(-roi);
  ```

### References
- [`<x> += <y>`  Costs More Gas Than `<x> = <x> + <y>` For State Variables](https://code4rena.com/reports/2022-12-caviar/#g-01-x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [StateVarPlusEqVsEqPlus.md](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)


## [G-13] When possible, use non-strict comparison >= and/or =< instead of > <

### Description

Non-strict inequalities are cheaper than strict ones due to some supplementary checks (`ISZERO`, 3 gas). It will save 15–20 gas.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::252 =>         if (vars.profit < yieldClaimThreshold[_collateral]) {
  ::264 =>         if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
  ::269 =>         } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
  ::278 =>         if (vars.netAssetMovement > 0) {
  ::281 =>         } else if (vars.netAssetMovement < 0) {
  ```
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::128 =>         assert(MIN_NET_DEBT > 0);
  ::197 =>         assert(vars.compositeDebt > 0);
  ::337 =>         if (!_isDebtIncrease && _LUSDChange > 0) {
  ::552 =>         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
  ```
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
  ```Solidity
  ::86 =>         if (lastIssuanceTimestamp < lastDistributionTime) {
  ::106 =>         if (lastIssuanceTimestamp < lastDistributionTime) {
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::152 =>         if (_LQTYamount > 0) {
  ::266 =>         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
  ::270 =>         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
  ```
- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)
  ```Solidity
  ::279 =>         if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::583 =>         } else if (currentP.mul(newProductFactor).div(DECIMAL_PRECISION) < SCALE_FACTOR) {
  ::591 =>         assert(newP > 0);
  ::742 =>         if (epochSnapshot < currentEpoch) { return 0; }
  ::768 =>         if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;}
  ::870 =>         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
  ::874 =>         require(_amount > 0, 'StabilityPool: Amount must be non-zero');
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::397 =>         } else if ((_ICR > _100pct) && (_ICR < _MCR)) {
  ::424 =>             if (singleLiquidation.collSurplus > 0) {
  ::452 =>         if (_LUSDInStabPool > 0) {
  ::493 =>         if (_collDecimals < LiquityMath.CR_CALCULATION_DECIMALS) {
  ::495 =>         } else if (_collDecimals > LiquityMath.CR_CALCULATION_DECIMALS) {
  ::551 =>         require(totals.totalDebtInSequence > 0);
  ::563 =>         if (totals.totalCollSurplus > 0) {
  ::651 =>             else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {
  ::694 =>             if (vars.ICR < _MCR) {
  ::754 =>         require(totals.totalDebtInSequence > 0);
  ::766 =>         if (totals.totalCollSurplus > 0) {
  ::853 =>             else if (vars.backToNormalMode && vars.ICR < vars.collMCR) {
  ::886 =>             if (vars.ICR < _MCR) {
  ::916 =>         if (_LUSD > 0) {
  ::920 =>         if (_collAmount > 0) {
  ::1160 =>         return (rewardSnapshots[_borrower][_collateral].collAmount < L_Collateral[_collateral]);
  ::1224 =>             assert(totalStakesSnapshot[_collateral] > 0);
  ::1386 =>         return TCR < _CCR;
  ::1414 =>         assert(newBaseRate > 0); // Base rate is always non-zero after redemption
  ::1450 =>         require(redemptionFee < _collateralDrawn);
  ::1539 =>         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::80 =>         if (wantBalance > _debt) {
  ::92 =>         if (wantBal < _amountNeeded) {
  ::99 =>         if (_amountNeeded > liquidatedAmount) {
  ::131 =>         if (totalAssets > allocated) {
  ::135 =>         } else if (totalAssets < allocated) {
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::234 =>         if (stratCurrentAllocation > stratMaxAllocation) {
  ::236 =>         } else if (stratCurrentAllocation < stratMaxAllocation) {
  ::368 =>         if (value > token.balanceOf(address(this))) {
  ::400 =>             if (value > vaultBalance) {
  ::420 =>         if (lockedFundsRatio < DEGRADATION_COEFFICIENT) {
  ::499 =>         if (_roi < 0) {
  ::502 =>         } else if (_roi > 0) {
  ::509 =>         if (vars.available < 0) {
  ::518 =>         } else if (vars.available > 0) {
  ::525 =>         if (vars.credit > vars.freeWantInStrat) {
  ::527 =>         } else if (vars.credit < vars.freeWantInStrat) {
  ::534 =>         if (vars.lockedProfitBeforeLoss > vars.loss) {
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::113 =>         if (availableCapital < 0) {
  ::120 =>             if (amountFreed < debt) {
  ::122 =>             } else if (amountFreed > debt) {
  ::127 =>             if (roi < 0) {
  ::192 =>             upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
  ```

### References
- [Solidity Gas Optimizations Tricks](https://betterprogramming.pub/solidity-gas-optimizations-and-tricks-2bcee0f9f1f2)


## [G-14] If possible, use private rather than public for constants

### Description

If necessary, the values can be obtained from the verified contract source code; alternatively, if there are many values, a single getter function that returns a tuple containing the values of all currently-public constants can be used. An example: [FraxlendPair.sol#L156-L178](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178).

The compiler doesn't have to write non-payable getter functions for deployment calldata, store the value's bytes somewhere other than where it will be used, or add another entry to the method ID table, which saves 3406-3606 gas in deployment gas.

### Findings

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
  ```Solidity
  ::30 =>     string constant public NAME = "ActivePool";
  ```
- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
  ```Solidity
  ::21 =>     string constant public NAME = "BorrowerOperations";
  ```
- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
  ```Solidity
  ::21 =>     uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
  ::25 =>     uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%
  ```
- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
  ```Solidity
  ::19 =>     string constant public NAME = "CommunityIssuance";
  ```
- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
  ```Solidity
  ::23 =>     string constant public NAME = "LQTYStaking";
  ```
- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
  ```Solidity
  ::150 =>     string constant public NAME = "StabilityPool";
  ::197 =>     uint public constant SCALE_FACTOR = 1e9;
  ```
- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
  ```Solidity
  ::48 =>     uint constant public SECONDS_IN_ONE_MINUTE = 60;
  ::53 =>     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
  ::54 =>     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
  ::55 =>     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
  ::61 =>     uint constant public BETA = 2;
  ```
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
  ```Solidity
  ::24 =>     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
  ::25 =>     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
  ::27 =>     IAaveProtocolDataProvider public constant DATA_PROVIDER =
  ::29 =>     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
  ```
- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
  ```Solidity
  ::40 =>     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
  ::41 =>     uint256 public constant PERCENT_DIVISOR = 10000;
  ::73 =>     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
  ::74 =>     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  ::75 =>     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  ::76 =>     bytes32 public constant ADMIN = keccak256("ADMIN");
  ```
- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
  ```Solidity
  ::23 =>     uint256 public constant PERCENT_DIVISOR = 10_000;
  ::24 =>     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
  ::25 =>     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;
  ::49 =>     bytes32 public constant KEEPER = keccak256("KEEPER");
  ::50 =>     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
  ::51 =>     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
  ::52 =>     bytes32 public constant ADMIN = keccak256("ADMIN");
  ```

### References
- [[G‑16]  Using private rather than public for constants, saves gas](https://code4rena.com/reports/2022-12-backed/#g16--using-private-rather-than-public-for-constants-saves-gas)
