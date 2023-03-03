Total of 30 issue either Low or NC.
---------
1 - delete doesn’t delete mapping in struct

The _updateDepositAndSnapshots() function in SP contract deletes a depositSnapshots struct, but the mapping in the Snapshots struct is not deleted. Values from this “deleted” mapping can still be accessed and there are no comments, function names, or other indicators that this is known. This issue is documented in Solidity’s security considerations documentation:
https://docs.soliditylang.org/en/v0.8.10/security-considerations.html#clearing-mappings

The delete operation occurs on 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L813

```
        if (_newValue == 0) {
            for (uint i = 0; i < collaterals.length; i++) {
                delete depositSnapshots[_depositor].S[collaterals[i]];
            }
            delete depositSnapshots[_depositor];
            emit DepositSnapshotUpdated(_depositor, 0, collaterals, amounts, 0);
            return;
        }
```

The mapping that still exists after the delete on:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L179

```
struct Snapshots {
        mapping (address => uint) S;
        uint P;
        uint G;
        uint128 scale;
        uint128 epoch;
    }

```
2 - Improper emit for events.

2.1 - The function "stake" inside LQTYStaking.sol emit the event "StakingGainsWithdrawn" before actually sending the accumulated LUSD and collateral gains to the caller. In case of transfer failure, this event will be emit already which is misleading.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L131

```
        lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);

        emit StakeChanged(msg.sender, newStake);
        emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);

         // Send accumulated LUSD and collateral gains to the caller
        if (currentStake != 0) {
            lusdToken.transfer(msg.sender, LUSDGain);
            _sendCollGainToUser(collGainAssets, collGainAmounts);
        }
    }
```

2.2 - Same on `StabilityPool` the function `_sendCollateralGainToDepositor` emit the event `CollateralSent` before the transfer occurs

```
    function _sendCollateralGainToDepositor(address _collateral, uint _amount) internal {
        if (_amount == 0) {return;}
        uint newCollAmount = collAmounts[_collateral].sub(_amount);
        collAmounts[_collateral] = newCollAmount;
        emit StabilityPoolCollateralBalanceUpdated(_collateral, newCollAmount);
        emit CollateralSent(_collateral, msg.sender, _amount); //@audit-info emit event before checking if the transfer success.

        IERC20(_collateral).safeTransfer(msg.sender, _amount);
    }

```

3 - Assert instead require to validate user inputs

Contracts use assert() instead of require() in multiple places. Assert is recommended to be used to check for internal errors, or to check invariants.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L420
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1227
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1282
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1417
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L338
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L128
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591


4 - Event is not emitted

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L190-L201

The active pool contract contains an event `ActivePoolLUSDDebtUpdated` that is used in the function `increaseLUSDDebt` and `decreaseLUSDDebt` However, this event is not emitted using the emit keyword, causing a compiler error. 

```
function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); //@audit missing emit keyword
    }

    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); //@audit missing emit keyword
    }

```

5 - Missing event for critical function

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120

The contract `CommunityIssuance` has the function `updateDistributionPeriod` which update the `distributionPeriod`, this function should have an event emitted.

```
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
        distributionPeriod = _newDistributionPeriod;
    }
```

6 - Missing zero balance check.

6.1 - The function `_payOutLQTYGains` inside StabilityPool doesn't verify that the user LQTYGain is not equal to zero before actually calling `sendOath()`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L840-L843

```
    function _payOutLQTYGains(ICommunityIssuance _communityIssuance, address _depositor) internal {
        uint depositorLQTYGain = getDepositorLQTYGain(_depositor); // @audit add an if statment to check depositorLQTYGain != 0
        _communityIssuance.sendOath(_depositor, depositorLQTYGain);
        emit LQTYPaidToDepositor(_depositor, depositorLQTYGain);
    }
``` 

6.2 - The function `unstake` on `LQTYStaking.sol` must check that either `_LQTYamount != 0 || LUSDGain !=0` otherwise it should revert with an error that amount is zero or the user doesn't has LUSD gain to claim yet. it's possible to call the function with 0 amount and with no LUSD to claim an attacker can spam the event logs.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142-L173

```
function unstake(uint _LQTYamount) external override {
        uint currentStake = stakes[msg.sender];
        _requireUserHasStake(currentStake);

        // Grab any accumulated ETH and LUSD gains from the current stake
        (address[] memory collGainAssets, uint[] memory collGainAmounts) = _getPendingCollateralGain(msg.sender);
        uint LUSDGain = _getPendingLUSDGain(msg.sender);
        //@audit should add require here to check either _LQTYamount != 0 || LUSDGain !=0
        _updateUserSnapshots(msg.sender);

        if (_LQTYamount > 0) {
            uint LQTYToWithdraw = LiquityMath._min(_LQTYamount, currentStake);

            uint newStake = currentStake.sub(LQTYToWithdraw);

            // Decrease user's stake and total LQTY staked
            stakes[msg.sender] = newStake;
            totalLQTYStaked = totalLQTYStaked.sub(LQTYToWithdraw);
            emit TotalLQTYStakedUpdated(totalLQTYStaked);

            // Transfer unstaked LQTY to user
            lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);

            emit StakeChanged(msg.sender, newStake);
        }

        emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);

        // Send accumulated LUSD and ETH gains to the caller
        lusdToken.transfer(msg.sender, LUSDGain);
        _sendCollGainToUser(collGainAssets, collGainAmounts);
    }
```

6.3 - The function `stake` on `LQTYStaking.sol` must check that `LUSDGain != 0` before sending LUSDGain on line 135

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L134-L138

```
        if (currentStake != 0) { // check that LUSDGain != 0 as well
            lusdToken.transfer(msg.sender, LUSDGain);
            _sendCollGainToUser(collGainAssets, collGainAmounts);
        }
```


7. Redundant Code Components

7.1 - The function `moveCollateralGainToTrove` not used anywhere and never get called by Stabilitypool
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L248-L251

```
    function moveCollateralGainToTrove(address _borrower, address _collateral, uint _collAmount, address _upperHint, address _lowerHint) external override {
        _requireCallerIsStabilityPool();
        _adjustTrove(_borrower, _collateral, _collAmount, 0, 0, false, _upperHint, _lowerHint, 0);
    }

```

7.2 - The function `isEmpty` not used anywhere and never get called.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L234

```
function isEmpty(address _collateral) public view override returns (bool) {
        return data[_collateral].size == 0;
    }
```

7.3 - The check on line 1087 which invoke `_requireTroveIsActive` is useless as this check already made inside `hasPendingRewards` function.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1084

```
function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
        if (hasPendingRewards(_borrower, _collateral)) {
            _requireTroveIsActive(_borrower, _collateral); //@audit no need for this line.

.....
....
```

```
function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {

        if (Troves[_borrower][_collateral].status != Status.active) {return false;}
       
        return (rewardSnapshots[_borrower][_collateral].collAmount < L_Collateral[_collateral]);
    }
```

8. Correct the error message

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L525
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L129

Display a correct message indicates the collateral address it not allowed.

```
    function _requireValidCollateralAddress(address _collateral) internal view {
        require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address"); //@audit chnage this to print correct error like address is not allowed.
    }
```

```
    modifier checkCollateral(address _collateral) {
        require(collateralConfig[_collateral].allowed, "Invalid collateral address"); //@audit chnage the error message to indicate the address is not allowed
        _;
    }
```


