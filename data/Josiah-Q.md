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
## USE MORE RECENT VERSIONS OF SOLIDITY
Lower versions like 0.8.7, 0.8.13 etc are being used in numerous contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Please visit the versions security fix list in the link below for detailed info:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

## NATSPEC SHOULD BE ADEQUATE
As denoted in the link below:

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). As clearly stated in the Solidity official documentation, in complex projects such as deFi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

## USE `UINT256` INSTEAD OF `UINT`
There are many instances of `uint` throughout all the codebases. In favor of explicitness, consider replacing all instances of `uint` with `uint256`.

## SOLIDITY COMPILER OPTIMIZATIONS COULD BE PROBLEMATIC
```
hardhat.config.js:
  29  module.exports = {
  30:   solidity: {
  31:     compilers: [
  32:       {
  33:         version: "0.8.0",
  34:         settings: {
  35:           optimizer: {
  36:               enabled: true,
  37:               runs: 1000000
  38
            }
```
Description: Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

Recommendation: Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## CONTRACT LAYOUT ON FUNCTION WRITINGS COMPLIANCE WITH SOLIDITY'S STYLE GUIDE
As denoted in Solidity's Style Guide:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Where possible, consider adhering to the above guidelines for all contract instances entailed.

## COMPILER VERSION PRAGMA SPECIFICITY
Non-library contracts and interfaces should avoid using floating pragmas ^0.8.0. Doing this may be a security risk for the actual application implementation itself. For instance, a known vulnerable compiler version may accidentally be selected or a security tool might fallback to an older compiler version leading to checking a different EVM compilation that is ultimately deployed on the blockchain.

## ZERO REPAYMENT AND COLLATERAL SENDING
In BorrowerOperations.sol, `_repayLUSD()` and `_activePool.sendCollateral()` in `_moveTokensAndCollateralfromAdjustment()` are permitted to proceed when `addColl()`, `moveCollateralGainToTrove()`, and `withdrawColl()` will have `_LUSDChange` inputted as zero whereas `withdrawLUSD()` and `repayLUSD()` will have zero `_collChange` associated. 

Suggested fix:

[BorrowerOperations.sol#L476-L502](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476-L502)

```
    function _moveTokensAndCollateralfromAdjustment
    (
        IActivePool _activePool,
        ILUSDToken _lusdToken,
        address _borrower,
        address _collateral,
        uint _collChange,
        bool _isCollIncrease,
        uint _LUSDChange,
        bool _isDebtIncrease,
        uint _netDebtChange
    )
        internal
    {
        if (_isDebtIncrease) {
            _withdrawLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange, _netDebtChange);
        } else {
            if (_LUSDChange != 0) _repayLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange);
        }

        if (_isCollIncrease) {
            IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);
            _activePoolAddColl(_activePool, _collateral, _collChange);
        } else {
            if (_collChange != 0) _activePool.sendCollateral(_collateral, _borrower, _collChange);
        }
    }
```
## THE USE OF DELETE TO RESET VARIABLES
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

Here are some of the instances found.

[TroveManager.sol#L1192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1192)

[TroveManager.sol#L1285-L1289](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1285-L1289)