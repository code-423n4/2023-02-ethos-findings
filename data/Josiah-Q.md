## MODERN MODULARITY ON IMPORT USAGES
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, it is recommended using named imports with curly braces (limiting to the needed instances if possible) instead of adopting the global import approach.

Suggested fix for a contract instance as an example:

[ReaperVaultERC4626.sol#L5-L6](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L5-L6)

```
import {ReaperVaultV2} from "./ReaperVaultV2.sol";
import {IERC4626Functions} from "./interfaces/IERC4626Functions.sol";
```

## CODE SIMPLIFICATION
Unused or unneeded code lines can be removed from the function code block for better readability and logic flow. For example, no fee will be minted to LQTYStaking.sol when the system is in recovery mode. [`_requireValidMaxFeePercentage(_maxFeePercentage, isRecoveryMode)`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648-L656) in BorrowerOperations.sol should therefore have its `isRecoveryMode` parameter and associated code lines removed just like it has been adopted in RedemptionHelper.sol.

Suggested fix:

```
    function _requireValidMaxFeePercentage(uint _maxFeePercentage) internal pure {
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
    }
```

With the above fix implemented, the functions, [`openTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648-L656) and [`_adjustTrove()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L293) calling it internally should also be refactored to:

```
        _requireValidMaxFeePercentage(_maxFeePercentage);
```

## BASERATE REMAINS ZERO UNTIL IT GETS INITIALIZED
The default value of `baseRate` remains `0` in TroveManager.sol until the first call on [`redeemCollateral()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1016-L1040) is successfully executed. Apparently, it is assigned `redeemedLUSDFraction.div(BETA)` in [`updateBaseRateFromRedemption()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1397-L1423) when the atomic call is routed back by RedemptioHelper.sol. 

As such, an early sanity check whether or not `basRate == 0` should be implemented to avoid unnecessary long atomic call when `getBorrowingFee()` is externally called from [`_triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L421) in BorrowerOperations.sol.  

Suggested fix:

Note: This fix is facilitated by the free getter function on the public state variable, [baseRate](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L63).

```diff
    function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {
-        _troveManager.decayBaseRateFromBorrowing(); // decay the baseRate state variable
+        if (_troveManager.baseRate() != 0) _troveManager.decayBaseRateFromBorrowing(); // decay the baseRate state variable
        uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);

        _requireUserAcceptsFee(LUSDFee, _LUSDAmount, _maxFeePercentage);
        
        // Send fee to LQTY staking contract
        lqtyStaking.increaseF_LUSD(LUSDFee);
        _lusdToken.mint(lqtyStakingAddress, LUSDFee);

        return LUSDFee;
    }
``` 
## THE USE OF ASSERT()
As denoted by the Solidity Documentations link below:

https://docs.soliditylang.org/en/v0.8.17/control-structures.html

"Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic."

The protocol uses `assert()` in multiple places of the code bases. Despite the strip of gas saving benefits for Solidity version ^0.8.0, `require()` should still be used for checking error conditions on inputs and return values.

Additionally, some of the assertions are truly unneeded considering the conditions associated are guaranteed to hold true. 

For example, in [`decayBaseRateFromBorrowing()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489) of TroveManager.sol, `decayedBaseRate` starts off as `0` till the first collateral redemption is made. It is impossible to exceed [`DECIMAL_PRECISION`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/BaseMath.sol#L6) (1e18) because [`MINUTE_DECAY_FACTOR`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53) (<1e18) ensures [`decayFactor`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1511) will be less than 1e18 too. This is also evident from the fact that [`redeemedLUSDFraction.div(BETA)`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1408-L1411) is way less than 1, which is the initial value of baseRate. This conclusively makes [`baseRate.mul(decayFactor).div(DECIMAL_PRECISION)`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1513) less than 1e18 prior to getting it assigned to [`decayedBaseRate`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1488).

Suggested fix:

Remove all similarly unneeded `assert()` from the codebase and use `require()` if need be.

## BORROWING FEE UPPER BOUND
The output result of [` _triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419-L430) ranges from 0.5% - 5%. For this reason, [_requireValidMaxFeePercentage()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648-L656) should have a its upper limit reasonably changed to 5% instead of 100%.

Suggested fix:

```
    function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {
        if (_isRecoveryMode) {
            require(_maxFeePercentage <= 5e15,
                "Max fee percentage must less than or equal to 5%");
        } else {
            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= 5e15,
                "Max fee percentage must be between 0.5% and 5%");
        }
    }
```