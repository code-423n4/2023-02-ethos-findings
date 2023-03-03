## Gas report
### G-01 
### Title: 
Incorrect use of storage variable in _harvestCore() function of ReaperStrategyGranarySupplyOnly contract .

[Github link](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117-L124)

### Summary:
The current implementation of the function uses the storage keyword to create a pointer to an array in the contract's storage. However, this is unnecessary and will lead to higher gas costs . Using memory instead of storage would be more appropriate and efficient.

Code changes:

```solidity
for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
            address[2] storage step = steps[i]; // Should be memory variable
            IERC20Upgradeable startToken = IERC20Upgradeable(step[0]);
            uint256 amount = startToken.balanceOf(address(this));
            if (amount == 0) {
                continue;
            }
            _swapVelo(step[0], step[1], amount, VELO_ROUTER);
        }
}
```

should be changed to:

```
for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
            // Changed to memory since this storage pointer is not used to update value
            address[2] memory step = steps[i]; 
            IERC20Upgradeable startToken = IERC20Upgradeable(step[0]);
            uint256 amount = startToken.balanceOf(address(this));
            if (amount == 0) {
                continue;
            }
            _swapVelo(step[0], step[1], amount, VELO_ROUTER);
        }
```