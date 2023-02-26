# Gas Optimization and QA report

## Use require to save gas
Replace `assert` with `require` to return unused gas on failure.
instances:
```
Ethos-Core/contracts/BorrowerOperations.sol#L128    assert(MIN_NET_DEBT > 0)
Ethos-Core/contracts/BorrowerOperations.sol#L197     assert(vars.compositeDebt > 0);
Ethos-Core/contracts/BorrowerOperations.sol#L301    assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0))
Ethos-Core/contracts/BorrowerOperations.sol#L331 assert(_collWithdrawal <= vars.coll);
Ethos-Core/contracts/TroveManager.sol#L417 assert(_LUSDInStabPool != 0)
Ethos-Core/contracts/TroveManager.sol#L1224 assert(totalStakesSnapshot[_collateral] > 0);
Ethos-Core/contracts/TroveManager.sol#L1279 assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
Ethos-Core/contracts/TroveManager.sol#L1342  assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
Ethos-Core/contracts/TroveManager.sol#L1348  assert(index <= idxLast);
Ethos-Core/contracts/TroveManager.sol#L1414  assert(newBaseRate > 0);
Ethos-Core/contracts/TroveManager.sol#L1489  assert(decayedBaseRate <= DECIMAL_PRECISION)
Ethos-Core/contracts/StabilityPool.sol#L526 assert(_debtToOffset <= _totalLUSDDeposits);
Ethos-Core/contracts/StabilityPool.sol#L551  assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
Ethos-Core/contracts/StabilityPool.sol#L591 assert(newP > 0);
Ethos-Core/contracts/LUSDToken.sol#L312 assert(sender != address(0));
Ethos-Core/contracts/LUSDToken.sol#L313 assert(recipient != address(0));
Ethos-Core/contracts/LUSDToken.sol#L321 assert(account != address(0));
Ethos-Core/contracts/LUSDToken.sol#L329 assert(account != address(0))
Ethos-Core/contracts/LUSDToken.sol#L337 assert(owner != address(0))
Ethos-Core/contracts/LUSDToken.sol#L338 assert(spender != address(0));

```

## Place validation of function parameters at the start of the function
This way gas to process the function body would be preserved on validation failure.
Instances:
```
Ethos-Core/contracts/BorrowerOperations.sol#301    assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0))
```

## Use a two step procedure when changing critical positions.
In case of any error while changing key positions it can be easily recovered.
Instance:
```
Ethos-Core/contracts/LUSDToken.sol#L146 function updateGovernance(address _newGovernanceAddress) external {
Ethos-Core/contracts/LUSDToken.sol#L153 function updateGuardian(address _newGuardianAddress) external {
```

## Use scientific notation to save gas
Instance:
```
Ethos-Vault/contracts/ReaperVaultV2.sol#L125    lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6;
```
Recommend change to
```
lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 1e6;
```

## dangling pragma solidity compiler version
Instances:
```
Ethos-Vault/contracts/ReaperVaultV2.sol#L3 pragma solidity ^0.8.0;
Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3 pragma solidity ^0.8.0;
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3 pragma solidity ^0.8.0;
```
Recommend change to
```
pragma solidity 0.8.0;
```

## Usage of old version of solidity compiler on Ethos-Core contracts.
```
pragma solidity 0.6.11;
```
Those contract might be missing out of the recent security updates and optimizations.

## cache array length in function body when used multiple times
Instances:
```
Ethos-Core/contracts/LQTY/LQTYStaking.sol#L205-L206
amounts = new uint[](assets.length);//@audit - cache length 1 mload(3 gas) and place memory_offset(3 gas) - 6 gas
        for (uint i = 0; i < assets.length; i++) {//@audit - cache to save 3 gas per iteration.
Ethos-Core/contracts/LQTY/LQTYStaking.sol#L226-L227
uint[] memory amounts = new uint[](collaterals.length);//@audit - cache length 1 mload(3 gas) and place memory_offset(3 gas) - 6 gas
        for (uint i = 0; i < collaterals.length; i++) {//@audit - cache to save 3 gas per iteration.
```