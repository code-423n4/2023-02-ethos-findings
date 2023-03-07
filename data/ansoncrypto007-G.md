### Emitting a storage value instead of memory one.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581

```
578    function updateTvlCap(uint256 _newTvlCap) public {
579        _atLeastRole(ADMIN);
580        tvlCap = _newTvlCap;
581        emit TvlCapUpdated(tvlCap);
582    }
```

### Reading a storage variable twice. Instead, a local variable could be used.
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395
##### ReaperVaultV2.sol._withdraw(): strategyBal should be used when updating strategies[stratAddr].allocated.
```
379                uint256 strategyBal = strategies[stratAddr].allocated;
380                if (strategyBal == 0) {
381                 continue;
382                }
383
384                uint256 remaining = value - vaultBalance;
385                uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
386                uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
387
388                // Withdrawer incurs any losses from withdrawing as reported by strat
389                if (loss != 0) {
390                    value -= loss;
391                    totalLoss += loss;
392                    _reportLoss(stratAddr, loss);
393                }
394
395                strategies[stratAddr].allocated -= actualWithdrawn;
```
'-=' reads `strategies[stratAddr].allocated` one more time. Instead, `strategyBal` could be used.
`strategies[stratAddr].allocated = strategyBal - actualWithdrawn;`


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L156
##### ReaperVaultV2.sol.addStrategy(): totalAllocBPS should be cached in memory.
```
156        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
156
158        strategies[_strategy] = StrategyParams({
159            activation: block.timestamp,
160            feeBPS: _feeBPS,
161            allocBPS: _allocBPS,
162            allocated: 0,
163            gains: 0,
164            losses: 0,
165            lastReport: block.timestamp
166        });
167
168        totalAllocBPS += _allocBPS;
```
`+=` reads `totalAllocBPS` one more time. So `totalAllocBPS` is read twice in addStrategy() function. Instead, a local variable could be used.