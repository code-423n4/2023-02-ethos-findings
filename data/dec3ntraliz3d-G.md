## Gas report
### G-01 

Incorrect use of storage variable in _harvestCore() function of `ReaperStrategyGranarySupplyOnly` contract .

[Link to the code on github](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117-L124)

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

```solidity
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

### G-02

The `_cascadingAccessRoles()` function in `ReaperVaultV2.sol` can be optimized by changing its state mutability from `view` to `pure` since the function does not access any storage variables. This change will not only make the function more efficient but also reduce gas costs.

[Link to the code on github](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L659-L667)

   ```solidity
   function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {
        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
        cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;
        cascadingAccessRoles[1] = ADMIN;
        cascadingAccessRoles[2] = GUARDIAN;
        cascadingAccessRoles[3] = STRATEGIST;
        cascadingAccessRoles[4] = DEPOSITOR;
        return cascadingAccessRoles;
    }
```

change the function header to 

   ```solidity
   
function _cascadingAccessRoles() internal pure override returns (bytes32[] memory) {
        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
        cascadingAccessRoles[0] = DEFAULT_ADMIN_ROLE;
        cascadingAccessRoles[1] = ADMIN;
        cascadingAccessRoles[2] = GUARDIAN;
        cascadingAccessRoles[3] = STRATEGIST;
        cascadingAccessRoles[4] = DEPOSITOR;
        return cascadingAccessRoles;
    }
```


### G-03

In the `setAddresses()` function of ActivePool.sol, an `SLOAD` can be saved by using a function parameter. When calling the `getAllowedCollaterals()` function, use `_collateralConfigAddress` instead of `collateralConfigAddress`, and hence save an `SLOAD`.

[Link to the code on github](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L105)
Current code 

   ```solidiy
        collateralConfigAddress = _collateralConfigAddress;
        // omitted ..

        address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
```

Change to 

   ```solidiy
        collateralConfigAddress = _collateralConfigAddress;
        // omitted ..

        address[] memory collaterals = ICollateralConfig(_collateralConfigAddress).getAllowedCollaterals();
```
