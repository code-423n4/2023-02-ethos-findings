## Impractical upper bound limit in `_requireValidMaxFeePercentage()`
In `_requireValidMaxFeePercentage()` of BorrowerOperations.sol, the upper bound of `_maxFeePercentage` is `DECIMAL_PRECISION`, i.e. 1e18 (or 100%). However, when [`getBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1471-L1473) is eventually invoked by [`_triggerBorrowingFee()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L421), the output returned will at most be equal to [`MAX_BORROWING_FEE`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1467), which is [5%](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L55). As such, `_requireValidMaxFeePercentage()` should be refactored as follows, regardless of the minimal impact this has on [`_requireUserAcceptsFee()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L90), to be more expediently in line with the logic flow entailed:

[File: BorrowerOperations.sol#L648-L656](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L648-L656)       

```diff
    function _requireValidMaxFeePercentage(uint _maxFeePercentage, bool _isRecoveryMode) internal pure {
        if (_isRecoveryMode) {
-            require(_maxFeePercentage <= DECIMAL_PRECISION,
+            require(_maxFeePercentage <= 5e15,
                "Max fee percentage must less than or equal to 100%");
        } else {
-            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");
+            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= 5e15,
                "Max fee percentage must be between 0.5% and 100%");
        }
    }
``` 
## Uninitialized `baseRate` till the first call on `redeemCollateral()`
The impact, though low, is twofold. First off, the output of `getBorrowingFee()` is always going to be `BORROWING_FEE_FLOOR`, i.e. [0.5%](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/Dependencies/LiquityBase.sol#L30) because `_baseRate == 0`:

[File: TroveManager.sol#L1464-L1469](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1464-L1469)

```solidity
    function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {
        return LiquityMath._min(
            BORROWING_FEE_FLOOR.add(_baseRate),
            MAX_BORROWING_FEE
        );
    }
```
Next, when `baseRate` finally gets assigned via [`updateBaseRateFromRedemption()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1408-L1417), increasing the `baseRate` based on the amount redeemed, as a proportion of total supply, is going to be minimally negligible unless the redemption amount is significantly sizable. And, if subsequently LUSD borrowing operations exceed redemptions, `baseRate` is as good as zero comparatively in [5e15, 5e16].

In another words, this makes the complexity of introducing a decaying factor pegged to a half-life of 720 minutes pretty much obsolete considering the borrowing rate stays always at 0.5%.  

Consider having `_calcDecayedBaseRate()` refactored as follows if it is going to take a while before the first redemption is made. Better yet, return `0` if `baseRate` falls below a reasonable dust value:

[File: TroveManager.sol#L1509-L1514](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1509-L1514)
     
```diff
    function _calcDecayedBaseRate() internal view returns (uint) {
+        if (baseRate == 0) return 0;

        uint minutesPassed = _minutesPassedSinceLastFeeOp();
        uint decayFactor = LiquityMath._decPow(MINUTE_DECAY_FACTOR, minutesPassed);

        return baseRate.mul(decayFactor).div(DECIMAL_PRECISION);
    }
```
## Inadequate sanity checks
In BorrowerOperations.sol, the require statement of `_requireSingularCollChange()` serving to ensure either a collateral change or a debt change is not going to prevent zero changes on both instances. Consider refactoring the affected function as follows:

[File: BorrowerOperations.sol#L533-L535](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533-L535)

```diff
    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
        require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
+        require(_collTopUp != 0 || _collWithdrawal != 0, "BorrowerOperations: Cannot withdraw and add coll");
    }
```
Note: The above refactoring is recommended since `_requireNonZeroAdjustment()` that ensues may not revert if `_collTopUp` and `_collWithdrawal` have been zero inputted with `_LUSDChange != 0`:

[File: BorrowerOperations.sol#L537-L539](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L537-L539)

```solidity
    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {
        require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");
    }
```
## Use a more recent version of solidity
The protocol adopts version 0.6.11 on writing some of the smart contracts. For better security, it is best practice to use the latest Solidity version, 0.8.17.

Security fix list where the protocol could benefit from, e.g. SafeMath in the versions can be found in the link below:

https://github.com/ethereum/solidity/blob/develop/Changelog.md

Here are the contract instances entailed:

[File: CollateralConfig.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3)
[File: BorrowerOperations.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3)
[File: TroveManager.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)
[File: ActivePool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3)
[File: StabilityPool.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3)
[File: CommunityIssuance.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)
[File: LQTYStaking.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)
[File: LUSDToken.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3)

## Modularity on import usages
For cleaner Solidity code in conjunction with the rule of modularity and modular programming, use named imports with curly braces instead of adopting the global import approach.

For instance, the import instances below could be refactored conforming to most other instances in the code bases as follows:

[File: CollateralConfig.sol#L5-L8](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L5-L8)

```diff
- import "./Dependencies/CheckContract.sol";
+ import {CheckContract} from "./Dependencies/CheckContract.sol";
- import "./Dependencies/Ownable.sol";
+ import {Ownable} from "./Dependencies/Ownable.sol";
- import "./Dependencies/SafeERC20.sol";
+ import {SafeERC20} from "./Dependencies/SafeERC20.sol";
- import "./Interfaces/ICollateralConfig.sol";
+ import {ICollateralConfig} from "./Interfaces/ICollateralConfig.sol";
```
## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Consider equipping all contracts with complete set of NatSpec to better facilitate users/developers interacting with the protocol's smart contracts.

## Return statement on named returns

Functions with named returns should not have a return statement to avoid unnecessary confusion.

For instance, the following `_getTCR()` may be refactored as follows:

```diff
    function _getTCR(address _collateral, uint _price, uint256 _collateralDecimals) internal view returns (uint TCR) {
        uint entireSystemColl = getEntireSystemColl(_collateral);
        uint entireSystemDebt = getEntireSystemDebt(_collateral);

        TCR = LiquityMath._computeCR(entireSystemColl, entireSystemDebt, _price, _collateralDecimals);

-        return TCR;
    }
``` 
## Lines too long
Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible.

Here are some of the instances entailed:

[File: BorrowerOperations.sol#L172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172)
[File: BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)
[File: BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282)
[File: BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)
[File: BorrowerOperations.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511)

## Inadequate checks in `initialize()` of CollateralConfig.sol
When initializing config struct variables, the function does not check if `_MCRs[i]` is smaller than `_CCRs[i]`. Consider having the function refactored as follows to avoid input error considering these state variables can only be initialized once:

[File: CollateralConfig.sol#L66-L71](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66-L71)

```diff
            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
            config.MCR = _MCRs[i];

            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
            config.CCR = _CCRs[i];

+            require(_CCRs[i] >= _MCRs[i], "CCR below MCR");
```
## `uint256` over `uint`
Across the codebase, there are numerous instances of `uint`, as opposed to `uint256`. In favor of explicitness, consider replacing all instances of `uint` with `uint256`.

Here are the contract instances entailed:

[File: BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)
[File: TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)
[File: ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)
[File: StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)
[File: CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
[File: LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
[File: LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

## Typo mistakes
[File: BorrowerOperations.sol#L651](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L651)

```diff
-                "Max fee percentage must less than or equal to 100%");
+                "Max fee percentage must be less than or equal to 100%");
```
## Non-compliant contract layout with Solidity's Style Guide
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the `view` and `pure` functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.0. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

Here are the contract instances entailed:

[File: ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
[File: ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
[File: ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)
[File: ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

## Camel/snake casing variable names
Non-constant state variables should also be conventionally camel cased where possible. If snake case is preferred/adopted, consider lower casing them.

Here are some of the instances entailed:

[File: TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
86:    mapping (address => mapping (address => Trove)) public Troves;

116:    mapping (address => address[]) public TroveOwners;
```
## Keeper for in time update of key variables
Consider adopting a keeper for updating time sensitive state variables. For instance, `L_Collateral[_collateral]` and `L_LUSDDebt[_collateral]` that have not been [updated](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1262) via in time liquidations may have users missing out on getting a higher rewards when [closing their troves](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L386). Where possible, make it remarkably noticeable front-end counting down key updates to be carried out so that users could make a sound decision in making the associated calls.