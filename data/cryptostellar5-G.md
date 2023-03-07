

|Sno.|Issue|Instances|Gas Savings|
|---|---|---|---|
|[G-01]|++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS|15|600
|[G-02]|REQUIRE or REVERT STRINGS LONGER THAN 32 BYTES COST EXTRA GAS|47|141
|[G-03]|SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS|4|12
|[G-04]|USING REQUIRE() INSTEAD OF ASSERT() HELPS SAVE GAS|20|
|[G-05]|X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES|21|2373
|[G-06]|USING FIXED BYTES IS CHEAPER THAN USING STRING|6|variable
|[G-07]|USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY|14|42
|[G-08]|USING 10\*\*X FOR CONSTANTS ISN'T GAS EFFICIENT|1|


### G-01 ++I or I++ SHOULD BE UNCHECKED{++I} or UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

*Number of Instances Identified: 15*

The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves **30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked)**.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
108: for(uint256 i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol

```
56: for(uint256 i = 0; i < _collaterals.length; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
351: for (uint i = 0; i < numCollaterals; i++) {
397: for (uint i = 0; i < numCollaterals; i++) {
640: for (uint i = 0; i < assets.length; i++) {
810: for (uint i = 0; i < collaterals.length; i++) {
831: for (uint i = 0; i < collaterals.length; i++) {
859: for (uint i = 0; i < numCollaterals; i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
608: for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
690: for (vars.i = 0; vars.i < _n; vars.i++) {
808: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
882: for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```
206: for (uint i = 0; i < assets.length; i++) {
228: for (uint i = 0; i < collaterals.length; i++) {
240: for (uint i = 0; i < numCollaterals; i++) {
```


### G-02 REQUIRE or REVERT STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

*Number of Instances Identified: 47*

Each extra chunk of bytes past the original 32 [incurs an MSTORE](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings) which costs 3 gas

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
107: require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
133: require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
145: require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
321-325: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == defaultPoolAddress,
            "ActivePool: Caller is neither BO nor Default Pool");
    }
329-335: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
    }
338-342: require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager");
    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
525: require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");
529: require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");
530: require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");
534: require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
538: require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
543: require(status == 1, "BorrowerOps: Trove does not exist or is closed");
552: require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");
561-564: require(
            !_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),
            "BorrowerOps: Operation not permitted during Recovery Mode"
        );
568: require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");
617: require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");
621: require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");
625: require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");
629: require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");
633: require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");
637: require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");
641: require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");
645:  require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");
650-651: require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must less than or equal to 100%")
653-654: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
134-137: require(
            msg.sender == guardianAddress || msg.sender == governanceAddress,
            "LUSD: Caller is not guardian or governance"
        );
347-351: require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address" //@audit-info longer than 32
        );
352-357: require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps" //@audit-info longer than 32
        );
362: require(msg.sender == borrowerOperationsAddress, "LUSDToken: Caller is not BorrowerOperations");
367-372: require(
            troveManagers[msg.sender] ||
            stabilityPools[msg.sender] ||
            borrowerOperations[msg.sender],
            "LUSD: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool"
        );
377: require(msg.sender == stabilityPoolAddress, "LUSD: Caller is not the StabilityPool");
384-386: require(
            troveManagers[msg.sender] || stabilityPools[msg.sender],
            "LUSD: Caller is neither TroveManager nor StabilityPool");
390: require(msg.sender == governanceAddress, "LUSDToken: Caller is not Governance");
394: require(!mintingPaused, "LUSDToken: Minting is currently paused");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
849: require( msg.sender == address(activePool), "StabilityPool: Caller is not ActivePool");
853: require(msg.sender == address(troveManager), "StabilityPool: Caller is not TroveManager");
870: require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
874: require(_amount > 0, 'StabilityPool: Amount must be non-zero');
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```
133: require(msg.sender == stabilityPoolAddress, "CommunityIssuance: caller is not SP");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

```
253-258: require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool"
        );
262: require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
266: require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');
270: require(_amount > 0, 'LQTYStaking: Amount must be non-zero');
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
150: require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
321: require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
619: require(degradation <= DEGRADATION_COEFFICIENT, "Degradation cannot be more than 100%");
```





### G-03 SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS

*Number of Instances Identified: 4*

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by **3 gas**

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
653-654: require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%"); 
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
347-351: require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address" 
        );
352-257: require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps" 
        );
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
1539: require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

### G-04 USING REQUIRE() INSTEAD OF ASSERT() HELPS SAVE GAS

*Number of Instances Identified: 20*


Between solc 0.4.10 and 0.8.0, `require()` used `REVERT` (0xfd) opcode which refunded remaining gas on failure while `assert()` used `INVALID` (0xfe) opcode which consumed all the supplied gas. (see [https://docs.soliditylang.org/en/v0.8.1/control-structures.html#error-handling-assert-require-revert-and-exceptions](https://docs.soliditylang.org/en/v0.8.1/control-structures.html#error-handling-assert-require-revert-and-exceptions)).  
`require()` should be used for checking error conditions on inputs and return values while `assert()` should be used for invariant checking (**properly functioning code should never reach a failing assert statement, unless there is a bug in your contract you should fix**).  
From the current usage of `assert`, my guess is that they can be replaced with `require`, unless a `Panic` really is intended.

Here are the `assert` locations:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301:  assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
311: assert(_collWithdrawal <= vars.coll);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
312: assert(sender != address(0));
313: assert(recipient != address(0));
321: assert(account != address(0));
329: assert(account != address(0));
337: assert(owner != address(0));
338: assert(spender != address(0));
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
417: assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);
```
### G-05 X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

*Number of Instances Identified: 21*

Using the addition operator instead of plus-equals saves **[113 gas]([https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)**

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
133: toFree += profit;
141: roi -= int256(loss);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
168:  totalAllocBPS += _allocBPS;
194: totalAllocBPS -= strategies[_strategy].allocBPS;
196: totalAllocBPS += _allocBPS;
214: totalAllocBPS -= strategies[_strategy].allocBPS;
390: value -= loss;
391: totalLoss += loss;
395: strategies[stratAddr].allocated -= actualWithdrawn;
396: totalAllocated -= actualWithdrawn;
444: stratParams.allocBPS -= bpsChange;
445: totalAllocBPS -= bpsChange;
450: stratParams.losses += loss;
451: stratParams.allocated -= loss;
452: totalAllocated -= loss;
505: strategy.gains += vars.gain;
514: strategy.allocated -= vars.debtPayment;
515: totalAllocated -= vars.debtPayment;
516: vars.debt -= vars.debtPayment;
520: strategy.allocated += vars.credit;
521: totalAllocated += vars.credit;
```


### G-06 USING FIXED BYTES IS CHEAPER THAN USING STRING

*Number of Instances Identified: 6*

As a rule of thumb, use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

```
30: string constant public NAME = "ActivePool";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

```
21: string constant public NAME = "BorrowerOperations";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol

```
32: string constant internal _NAME = "LUSD Stablecoin";
33: string constant internal _SYMBOL = "LUSD";
34: string constant internal _VERSION = "1";
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

```
19: string constant public NAME = "CommunityIssuance";
```




### G-07 USING BOTH NAMED RETURNS AND A RETURN STATEMENT ISNT NECESSARY

*Number of Instances Identified: 14*

Removing unused named returns variables can reduce gas usage (MSTOREs/MLOADs) and improve code clarity. To save gas and improve code quality: consider using only one of those.
Each MLOAD and MSTORE costs `3 additional gas`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol

```
511: returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol

```
1316: returns (uint index)
1321: returns (uint128 index)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

```
104: returns (uint256 amountFreed)
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol

```
29: returns (address assetTokenAddress)
37: returns (uint256 totalManagedAssets)
51: returns (uint256 shares)
66: returns (uint256 assets)
79: returns (uint256 maxAssets)
96: returns (uint256 shares)
122: returns (uint256 maxShares)
165: returns (uint256 maxAssets)
220: returns (uint256 maxShares)
240: returns (uint256 assets)
```



### G-08 USING 10\*\*X FOR CONSTANTS ISN'T GAS EFFICIENT

*Number of Instances Identified: 1*

In Solidity, a `constant` expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form `10**X` can be rewritten as `1eX` to save the gas cost from the calculation with the exponentiation operator `**`.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol

```
40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18;
```
