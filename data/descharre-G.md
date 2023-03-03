# Summary
|ID     | Finding|  Gas saved | Instances|
|:----: | :---           |  :----:         |:----:         |
|G-01      |Make for loops unchecked| - |  12 |
|G-02      |Add unchecked block when operands can't under/overflow| 536 | 15 |
|G-03      |Don't write to memory when variable is only used once| 10 | 1 |
|G-04      |Unnecessary nonReentrant modifier| 2374 | 1 |
|G-05      |Save return value to memory instead of calling function multiple times| - | 1 |
|G-06      |function `_atLeastRole()` can be made more gas efficiënt| 300 | 1 |
|G-07      |Initialize a static array instead of a dynamic array| 300 | - |
|G-08      |Unnecessary if/require statements| - | 2 |
|G-09      |Internal functions only called once can be inlined to save gas| 30 | 1 |
|G-10      |Using double if/require statement instead of && consumes less gas| 15 | 7 |
|G-11      |Use the calldata variable when it has the same value as a state variable| 200 | 1 |
|G-12      |Bytes32 constant is cheaper than string constant| - | 4 |
|G-13      |Combine events to save gas| 5000 | 6 |
|G-14      |Amount is always equal to balance after - balance before| 980 | 1 |
# Details
## [G-01] Make for loops unchecked
The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow. The longer the array, the more gas you will save.

- [ActivePool.sol#L108-L113](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108-L113)
- [CollateralConfig.sol#L56-L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56-L73)
- [StabilityPool.sol#L351-L356](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L351-L356)
- [StabilityPool.sol#L397-L403](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L397-L403)
- [StabilityPool.sol#L640-L643](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L640-L643)
- [StabilityPool.sol#L810-L812](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810-L812)
- [StabilityPool.sol#L831-L836](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831-L836)
- [StabilityPool.sol#L859-L866](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L859-L866)
- [TroveManager.sol#L608-L650](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608-L650)
- [TroveManager.sol#L690-L709](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690-L709)
- [TroveManager.sol#L808-L851](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808-L851)
- [TroveManager.sol#L882-L893](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882-L893)

Unchecked for loop can be done like this:
```diff
-       for(uint256 i = 0; i < numCollaterals; i++) {
+       for(uint256 i = 0; i < numCollaterals;) {
            address collateral = collaterals[i];
            address vault = _erc4626vaults[i];
            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
            yieldGenerator[collateral] = vault;
+           unchecked {
+               i++;
+           }
        }
```
## [G-02] Add unchecked block when operands can't under/overflow
When operands can't underflow/overflow because of a previous require() or if statement, you can use an unchecked block.
The default "checked" behavior costs more gas when calculating, because under-the-hood the EVM does extra checks for over/underflow.
|Code     |Explanation| Function |  Gas saved |
|:----: | :---           |  :----:         |:----:         |
|[ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)|AllocBPS can never be higher than the totalAllocBPS| `updateStrategyAllocBPS()` |170 |
|[ReaperVaultV2.sol#L391](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L391)|Loss is decremented from value in the line above. <br>If loss would be really high it would already revert there.| `withdraw()` |90 |
|[ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)| Total allocated is always more than the allocation of one strategy| `updateStrategyAllocBPS()` |218 |
|[ReaperVaultV2.sol#L330](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L330)| When tokens are sent to the vault, balance after is always <br> greater than or equal to balance before| `deposit()` |58 |
|[ReaperStrategyGranarySupplyOnly.sol#L93](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93)| Can be done because of previous if check| `_liquidatePosition()` |- |
|[ReaperStrategyGranarySupplyOnly.sol#L100](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100)| Can be done because of previous if check| `_liquidatePosition()` |- |

This can also be done for divisions.

The expression type(int).min / (-1) is the only case where division causes an overflow. On average 35 gas saved:
- [ReaperVaultV2.sol#L334](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334)
- [ReaperVaultV2.sol#L231](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L231)
- [ReaperVaultV2.sol#L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L237)
- [ReaperVaultV2.sol#L365](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L365)
- [ReaperVaultV2.sol#L405](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L405)
- [ReaperVaultV2.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421)
- [ReaperVaultV2.sol#L440](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L440)
- [ReaperVaultV2.sol#L463](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L463)
- [ReaperVaultV2.sol#L466](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L466)
## [G-03] Don't write to memory when variable is only used once
When a storage variable is only used once, it's a waste of gas to write it to a memory variable. This adds some additional stack opcodes.
- [ReaperVaultV2.sol#L323](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L323): `deposit()` gas saved: 10
## [G-04] Unnecessary nonReentrant modifier
Using safeTransferFrom from the openZeppeling safeErc20 library is safe against reentrancy. Even if the attacker is able to reenter the function, he will only disadvantage himself

- [ReaperVaultV2.sol#L319](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L319): `deposit()` gas saved: 2374
## [G-05] Save return value to memory instead of calling function multiple times
Instead of calling a function multiple times it's more gas efficiënt if you call the function and save it to memory so you can use that later.
|Code     |Explanation| Function |  Gas saved |
|:----: | :---           |  :----:         |:----:         |
|[ReaperVaultV2.sol#L231-L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L231-L237)|`balance()` is called twice| `updateStrategyAllocBPS()` | - |
## [G-06] function `_atLeastRole()` can be made more gas efficiënt
Although the contract is not in scope, it's used in almost every function of ReaperVaultV2, ReaperBaseStrategyV4 and ReaperStrategyGranarySupplyOnly.

[ReaperAccessControl.sol#L21-L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/ReaperAccessControl.sol#L21-L40): saves 200-500 per function with access control depending on the role. A role higher in the cascading list will save more gas because it has to go through more loops.

The function can be made a lot smaller and more gas efficiënt
```solidity
    function _atLeastRole(bytes32 _role) internal view {
        bytes32[] memory cascadingAccessRoles = _cascadingAccessRoles();
        uint256 numRoles = cascadingAccessRoles.length;
        bool specifiedRoleFound = false;
        bool senderHighestRoleFound = false;

        // {_role} must be found in the {cascadingAccessRoles} array.
        // Also, msg.sender's highest role index <= specified role index.
        for (uint256 i = 0; i < numRoles; i = i.uncheckedInc()) {
            if (!senderHighestRoleFound && _hasRole(cascadingAccessRoles[i], msg.sender)) {
                senderHighestRoleFound = true;
            }
            if (_role == cascadingAccessRoles[i]) {
                specifiedRoleFound = true;
                break;
            }
        }

        require(specifiedRoleFound && senderHighestRoleFound, "Unauthorized access");
    }
```
Can be changed to:
```solidity
    function _atLeastRole(bytes32 _role) internal view {
        bytes32[] memory cascadingAccessRoles = _cascadingAccessRoles();
        uint256 numRoles = cascadingAccessRoles.length;
        
        // {_role} must be found in the {cascadingAccessRoles} array.
        // Also, msg.sender's highest role index <= specified role index.
        for (uint256 i = 0; i < numRoles; i = i.uncheckedInc()) {
            if(_hasRole(cascadingAccessRoles[i], msg.sender)){
                break;
            }
            if(_role == cascadingAccessRoles[i]){
                revert("Unauthorized access");
            }
        }
    }
```
## [G-07] Initialize a static array instead of a dynamic array
The function `_cascadingAccessRoles()` initializes a dynamic array. All the values are known beforehand so it's completely possible to initialize a static array. Saves on average 300 gas per function that has access control in Ethos Vault. Every time a function is called with access control, the array needs to get initialized.

[ReaperVaultV2.sol#L659-L667](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L659-L667)
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
can be changed to
```solidity
    function _cascadingAccessRoles() internal view override returns (bytes32[5] memory) {
        bytes32[5] memory cascadingAccessRoles = [DEFAULT_ADMIN_ROLE, ADMIN, GUARDIAN, STRATEGIST, DEPOSITOR ];
        return cascadingAccessRoles;
    }
```
Can also be done in [ReaperBaseStrategyv4.sol#L203-L211](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203-L211)

For this to work, the return type in the [interface](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/ReaperAccessControl.sol#L47) and the array in [`_atLeastRole()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/mixins/ReaperAccessControl.sol#L22) has to be adjusted to bytes32[5].
## [G-08] Unnecessary if/require statements
|Code     |Explanation| Function |  Gas saved |
|:----: | :---           |  :----:         |:----:         |
|[ReaperStrategyGranarySupplyOnly.sol#L177](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L177)|`_deposit()` is only called once in `_adjustPosition()`, there is <br> already a check that prevents it from being 0| `_deposit()` | - |
|[ReaperStrategyGranarySupplyOnly.sol#L188](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L188)|`_withdraw()` is only called once in `_liquidatePosition()`, there is <br> already a check that prevents it from being 0| `_withdraw()` | - |
## [G-09] Internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

This applies to [`_withdraw()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L187-L191) and [`_deposit()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176-L182).
## [G-10] Using double if/require statement instead of && consumes less gas
Saves around 15 gas for require statements and 40 for if statements
```diff
-       if (_isDebtIncrease && !isRecoveryMode) { 
+       if (_isDebtIncrease) { 
+         if (!isRecoveryMode) {      
            vars.LUSDFee = _triggerBorrowingFee(contractsCache.troveManager, contractsCache.lusdToken, _LUSDChange, _maxFeePercentage);
            vars.netDebtChange = vars.netDebtChange.add(vars.LUSDFee); // The raw debt change includes the fee
          }
+       }        
```
- [BorrowerOperations.sol#L311](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L311)
- [BorrowerOperations.sol#L337](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L337)
- [LUSDToken.sol#L347-L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347-L351)
- [LUSDToken.sol#L352-L357](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L352-L357)
- [TroveManager.sol#L616](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L616)
- [TroveManager.sol#L817](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L817)
- [TroveManager.sol#L1539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1539)
## [G-11] Use the calldata variable when it has the same value as a state variable
In the function `sendCollateral` there are 2 if checks to see if the parameter `_account` matches an address which is a state variable. If the statement is true they reuse the state variable instead of the parameter `_account`. This adds an extra 200 gas because the evm has to perform 2 hot loads. A better way is to use the `_account` parameter which uses an mload which costs only 3 gas.

[ActivePool.sol#L179-L184](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L179-L184)
```diff
    function sendCollateral(address _collateral, address _account, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        _rebalance(_collateral, _amount);
        collAmount[_collateral] = collAmount[_collateral].sub(_amount);
        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
        emit CollateralSent(_collateral, _account, _amount);

        if (_account == defaultPoolAddress) {
-           IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);
-           IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+           IERC20(_collateral).safeIncreaseAllowance(_account, _amount);
+           IDefaultPool(_account).pullCollateralFromActivePool(_collateral, _amount);
        } else if (_account == collSurplusPoolAddress) {
-           IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);
-           ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+           IERC20(_collateral).safeIncreaseAllowance(_account, _amount);
+           ICollSurplusPool(_account).pullCollateralFromActivePool(_collateral, _amount);
        } else {
            IERC20(_collateral).safeTransfer(_account, _amount);
        }
    }
```
## [G-12] Bytes32 constant is cheaper than string constant
If the string can fit into 32 bytes, then bytes32 is cheaper than string. This saves around 30k gas in deployments cost
- [ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)
- [BorrowerOperations.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21)
- [LUSDToken.sol#L32-L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32-L34)
- [StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150)
## [G-13] Combine events to save gas
In the function `setAddresses` there is for every changed address a seperate event. It's more gas efficient when you combine all these events.
```solidity
        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
        emit CollateralConfigAddressChanged(_collateralConfigAddress);
        emit TroveManagerAddressChanged(_troveManagerAddress);
        emit ActivePoolAddressChanged(_activePoolAddress);
        emit LUSDTokenAddressChanged(_lusdTokenAddress);
        emit SortedTrovesAddressChanged(_sortedTrovesAddress);
        emit PriceFeedAddressChanged(_priceFeedAddress);
        emit CommunityIssuanceAddressChanged(_communityIssuanceAddress);
```
Can be changed to
```solidity
    emit AdressesChanged(_borrowerOperationsAddress, _collateralConfigAddress, _troveManagerAddress,_activePoolAddress, _lusdTokenAddress, _sortedTrovesAddress, _priceFeedAddress, _communityIssuanceAddress );
```
This can be done in every contract with the function `setAddresses`.
- [StabilityPool.sol#L293-L300](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L293-L300): 5523 gas saved
- [ActivePool.sol#L115-L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L115-L120)
- [BorrowerOperations.sol#L155-L165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L155-L165)
- [TroveManager.sol#L280-L292](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L280-L292)
- [LQTYStaking.sol#L93-L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L93-L98)
## [G-14] Amount is always equal to balance after - balance before
There is no need to update _amount because it will always be equal to balAfter-balBefore. SafeTransferFrom will always send the exact amount, if this is not possible it will revert.

[ReaperVaultV2.sol#L330](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L330): `deposit()` gas saved: 980
```diff
    uint256 balBefore = token.balanceOf(address(this));
    token.safeTransferFrom(msg.sender, address(this), _amount);
-   uint256 balAfter = token.balanceOf(address(this));
-   _amount = balAfter - balBefore;
```