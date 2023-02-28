| Total Low issues |
|------------------|

| Risk   | Issues Details                                                                          | Number        |
|--------|-----------------------------------------------------------------------------------------|---------------|
| [L-01] | Low level calls with solidity version 0.8.14 and lower can result in optimiser bug      | 1             |
| [L-02] | No Storage Gap for Upgradeable contracts                                                | 2             |
| [L-03] | Loss of precision due to rounding                                                       | 10            |
| [L-04] | ERC4626Cloned 's implmentation is not fully up to EIP-4626's specification              | 1             |
| [L-05] | Multiple Pragma used                                                                    | All Contracts |
| [L-06] | Missing Event for initialize                                                            | 2             |
| [L-07] | Not all tokens support approve-max                                                      | 1             |
| [L-08] | Value is not validated to be different than the existing one	                           | 2             |
| [L-09] | Use require instead of assert                                                           | 20            |
| [L-10] | Misleading comment on `ActivePool.sol`                                                  | 1             |
| [L-11] | Integer overflow by unsafe casting                                                      | 3             |

| Total Non-Critical issues |
|---------------------------|
  
| Risk    | Issues Details                                                                                 | Number        |
|---------|------------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Include return parameters in NatSpec comments                                                  | All Contracts |
| [NC-02] | Non-usage of specific imports                                                                  | All Contracts |
| [NC-03] | Lines are too long                                                                             | 10            |
| [NC-04] | Lack of event emit                                                                             | 6             |
| [NC-05] | Function writing does not comply with the `Solidity Style Guide`                               | All Contracts |
| [NC-06] | Solidity compiler optimizations can be problematic                                             | 2             |
| [NC-07] | Mark visibility of `initialize()` functions as external                                        | 1             |
| [NC-08] | Use `bytes.concat()` and `string.concat()`                                                     | 1             |
| [NC-09] | Reusable require statements should be changed to a modifier                                    | 6             |
| [NC-10] | Use a more recent version of OpenZeppelin dependencies                                         | 1             |
| [NC-11] | The protocol should include NatSpec                                                            | All Contracts |
| [NC-12] | Constants in comparisons should appear on the left side                                        | 37            |
| [NC-13] | Use a more recent version of solidity                                                          | All Contracts |
| [NC-14] | Contracts should have full test coverage                                                       | All Contracts |
| [NC-15] | Signature Malleability of EVM's `ecrecover()`                                                  | 1             |
| [NC-16] | Add a timelock to critical functions                                                           | 4             |
| [NC-17] | Use `immutable` instead of `constant` for values such as a call to `keccak256()`               | 8             |
| [NC-18] | Need Fuzzing test                                                                              | All Contracts |
| [NC-19] | Generate perfect code headers every time                                                       | All Contracts |
| [NC-20] | Lock pragmas to specific compiler version                                                      | 4             |
| [NC-21] | Consider using `delete` rather than assigning zero to clear values                             | 22            |
| [NC-22] | Adding a `return` statement when the function defines a named return variable, is redundant    | 4             |
| [NC-23] | For functions, follow Solidity standard naming conventions                                     | All Contracts |
| [NC-24] | Events that mark critical parameter changes should contain both the old and the new value      | 4             |
| [NC-25] | Add NatSpec Mapping comment                                                                    | 20            |
| [NC-26] | Constants should be defined rather than using magic numbers                                    | 6             |
| [NC-27] | There is no need to cast a variable that is already an address, such as `address(x)`           | 1             |
| [NC-28] | Use SMTChecker                                                                                 |               |
| [NC-29] | Not using the named return variables anywhere in the function is confusing                     | 16            |
| [NC-30] | Use `uint256` instead `uint`                                                                   | 477           |
| [NC-31] | Use scientific notation rather than exponentiation                                             | 11            |

## [L-01] Low level calls with solidity version 0.8.14 and lower can result in optimiser bug

The protocol is using low level calls with solidity version 0.8.12 which can result in optimizer bug.

> Ref: https://medium.com/certora/overly-optimistic-optimizer-certora-bug-disclosure-2101e3f7994d

#### Lines of code 

```solidity
    function _chainID() private pure returns (uint256 chainID) {
        assembly {
            chainID := chainid()
        }
    }
```

- [Ethos-Core/contracts/LUSDToken.sol#L299](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L299)

#### Recommended Mitigation Steps

Consider Upgrading to 0.8.15 or more.

## [L-02] No Storage Gap for Upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

#### Lines of code 

- [Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)
- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

#### Recommended Mitigation Steps

Consider adding a storage gap at the end of the upgradeable contract:
```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-03] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

#### Lines of code 

```solidity
        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)

```solidity
        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L231](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L231)

```solidity
            uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L237](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L237)

```solidity
        return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L296](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L296)

```solidity
            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L334](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L334)

```solidity
        value = (_freeFunds() * _shares) / totalSupply();
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L365](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L365)

```solidity
                totalLoss <= ((value + totalLoss) * withdrawMaxLoss) / PERCENT_DIVISOR,
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L405](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L405)

```solidity
            return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421)

```solidity
        uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L463](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L463)

```solidity
            uint256 shares = supply == 0 ? performanceFee : (performanceFee * supply) / _freeFunds();
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L466](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L466)

## [L-04] ERC4626Cloned 's implmentation is not fully up to EIP-4626's specification

#### Description

Must return the maximum amount of shares mint would allow to be deposited to receiver and not cause a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary).

This assumes that the user has infinite assets, i.e. must not rely on balanceOf of asset. Could cause unexpected behavior in the future due to non-compliance with EIP-4626 standard.

#### Lines of code 

```solidity
    function maxDeposit(address) external view override returns (uint256 maxAssets) {
        if (emergencyShutdown || balance() >= tvlCap) return 0;
        if (tvlCap == type(uint256).max) return type(uint256).max;
        return tvlCap - balance();
    }
```

- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79-L83](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79-L83)

```solidity
    function maxMint(address) external view override returns (uint256 maxShares) {
        if (emergencyShutdown || balance() >= tvlCap) return 0;
        if (tvlCap == type(uint256).max) return type(uint256).max;
        return convertToShares(tvlCap - balance());
    }
```

- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122-L126](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122-L126)

#### Recommended Mitigation Steps

`maxMint()` and `maxDeposit()` should reflect the limitation of maxSupply.

## [L-05] Multiple Pragma used 

#### Description

Solidity pragma versions should be exactly same in all contracts. Currently some contracts use `0.6.11` and others use `^0.8.0`

#### Lines of code 

```solidity
pragma solidity 0.6.11;
```

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Use one version for all contracts.

## [L-06] Missing Event for initialize

#### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip.

#### Lines of code 

```solidity
    function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
    ) external override onlyOwner {
```

- [Ethos-Core/contracts/CollateralConfig.sol#L46-L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L76)

```solidity
    function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
```

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L84](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L84)

#### Recommended Mitigation Steps

Add Event-Emit

## [L-07] Not all tokens support approve-max

#### Description

Some tokens consider uint256 as just another value. If one of these tokens is used and the approval balance eventually reaches only a couple wei, nobody will be able to send funds because there won't be enough approval.

#### Lines of code 

```solidity
        IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74)

## [L-08] Value is not validated to be different than the existing one	

#### Description

The `updateCollateralRatios()` function is an admin function to update `_MCR` and `_CCR`, it must only lower the values as stated in the comments:

```solidity
    // Admin function to lower the collateralization requirements for a particular collateral.
    // Can only lower, not increase.
```

but in the bellow checks:

```solidity
        require(_MCR <= config.MCR, "Can only walk down the MCR");
        require(_CCR <= config.CCR, "Can only walk down the CCR");
```

the owner can also set the values to be the same as before which will waste a Gcoldsload of gas and cause multiple abnormal events to be emitted, will ultimately result in a no-op situation.

#### Lines of code 

- [Ethos-Core/contracts/CollateralConfig.sol#L91](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L91)
- [Ethos-Core/contracts/CollateralConfig.sol#L92](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L92)

#### Recommended Mitigation Steps

```solidity
    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR < config.MCR, "Can only walk down the MCR");
        require(_CCR < config.CCR, "Can only walk down the CCR");

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }
```

## [L-09] Use require instead of assert

#### Description

Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as `request()/revert()` did.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` statement when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

Assertion should be avoided even after solidity version 0.8.0, because its documentation states "The Assert function generates an error of type `Panic(uint256)`. Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake".

#### Lines of code 

```solidity
        assert(MIN_NET_DEBT > 0);
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128)

```solidity
        assert(vars.compositeDebt > 0);
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197)

```solidity
        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
```

- [BorrowerOperations.sol#L301](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301)

```solidity
        assert(_collWithdrawal <= vars.coll); 
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L331](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331)

```solidity
            assert(_LUSDInStabPool != 0);
```

- [Ethos-Core/contracts/TroveManager.sol#L417](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417)

```solidity
            assert(totalStakesSnapshot[_collateral] > 0);
```

- [Ethos-Core/contracts/TroveManager.sol#L1224](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224)

```solidity
        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
```

- [Ethos-Core/contracts/TroveManager.sol#L1279](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279)

```solidity
        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
```

- [Ethos-Core/contracts/TroveManager.sol#L1342](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342)

```solidity
        assert(index <= idxLast);
```

- [Ethos-Core/contracts/TroveManager.sol#L1348](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348)

```solidity
        assert(newBaseRate > 0); // Base rate is always non-zero after redemption
```

- [Ethos-Core/contracts/TroveManager.sol#L1414](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414)

```solidity
        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```

- [Ethos-Core/contracts/TroveManager.sol#L1489](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489)

```solidity
        assert(_debtToOffset <= _totalLUSDDeposits);
```

- [Ethos-Core/contracts/StabilityPool.sol#L526](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526)

```solidity
        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
```

- [Ethos-Core/contracts/StabilityPool.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551)

```solidity
        assert(newP > 0);
```

- [Ethos-Core/contracts/StabilityPool.sol#L591](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591)

```solidity
        assert(sender != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L312](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312)

```solidity
        assert(recipient != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L313](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L313)

```solidity
        assert(account != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L321](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321)

```solidity
        assert(account != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L329](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329)

```solidity
        assert(owner != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L337](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337)

```solidity
        assert(spender != address(0));
```

- [Ethos-Core/contracts/LUSDToken.sol#L338](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L338)

#### Recommended Mitigation Steps

Use `require()` instead of `assert()`

## [L-10] Misleading comment on `ActivePool.sol`

#### Description

There is a misleading comment on the `ActivePool.sol` that may mislead some users or auditors in the future:

```solidity
        // if abs(percentOfFinalBal - yieldingPercentage) > drift, we will need to deposit more or withdraw some
        vars.yieldingPercentage = yieldingPercentage[_collateral];
        vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
        vars.yieldingAmount = yieldingAmount[_collateral];
        if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
            // we will end up overallocated, withdraw some
            vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);
            vars.yieldingAmount = vars.yieldingAmount.sub(vars.toWithdraw);
            yieldingAmount[_collateral] = vars.yieldingAmount;
        } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
            // we will end up underallocated, deposit more
            vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);
            vars.yieldingAmount = vars.yieldingAmount.add(vars.toDeposit);
            yieldingAmount[_collateral] = vars.yieldingAmount;
        }
```

As you can see in the above code, `if abs(percentOfFinalBal - yieldingPercentage) > drift` we will withdraw some and only `if abs(yieldingPercentage - percentOfFinalBal) > drift` we will need to deposit more. 

#### Lines of code 

```solidity
// if abs(percentOfFinalBal - yieldingPercentage) > drift, we will need to deposit more or withdraw some
```

- [Ethos-Core/contracts/ActivePool.sol#L260](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L260)

#### Recommended Mitigation Steps

Change the NatSpec to :

```solidity
        // if abs(percentOfFinalBal - yieldingPercentage) > drift, we will need to withdraw some
        vars.yieldingPercentage = yieldingPercentage[_collateral];
        vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
        vars.yieldingAmount = yieldingAmount[_collateral];
        if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
            // we will end up overallocated, withdraw some
            vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);
            vars.yieldingAmount = vars.yieldingAmount.sub(vars.toWithdraw);
            yieldingAmount[_collateral] = vars.yieldingAmount;
        // if abs(yieldingPercentage - percentOfFinalBal) > drift, we will need to deposit more    
        } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
            // we will end up underallocated, deposit more
            vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);
            vars.yieldingAmount = vars.yieldingAmount.add(vars.toDeposit);
            yieldingAmount[_collateral] = vars.yieldingAmount;
        }
```

## [L-11] Integer overflow by unsafe casting

#### Description

Integer overflow by unsafe casting. It is necessary to safely convert between the different numeric types.

#### Lines of code 

```solidity
        vars.netAssetMovement = int256(vars.toDeposit) - int256(vars.toWithdraw) - int256(vars.profit);
```

- [Ethos-Core/contracts/ActivePool.sol#L277](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L277)

#### Recommended Mitigation Steps

Use OpenZeppelin [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) library.

## [NC-01] Include return parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`.
Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-02-ethos)

#### Recommended Mitigation Steps

Include `@return` parameters in NatSpec comments

## [NC-02] Non-usage of specific imports

#### Description

The current form of relative path import is not recommended for use because it can unpredictably pollute the namespace. Instead, the Solidity docs recommend specifying imported symbols explicitly. 

- https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-02-ethos)

#### Recommended Mitigation Steps

Use specific imports syntax per solidity docs recommendation.

## [NC-03] Lines are too long

#### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

> Ref: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Lines of code 

```solidity
    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L172](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172)

```solidity
    function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)

```solidity
    function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282)

```solidity
        (vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)

```solidity
        emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInNormalMode);
```

- [Ethos-Core/contracts/TroveManager.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351)

```solidity
            emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
```

- [Ethos-Core/contracts/TroveManager.sol#L393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393)

```solidity
            emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
```

- [Ethos-Core/contracts/TroveManager.sol#L407](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L407)

```solidity
            emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.collToSendToSP, TroveManagerOperation.liquidateInRecoveryMode);
```

- [Ethos-Core/contracts/TroveManager.sol#L428](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L428)

```solidity
    * The redeemer swaps (debt - liquidation reserve) LUSD for (debt - liquidation reserve) worth of collateral, so the LUSD liquidation reserve left corresponds to the remaining debt.
```

- [Ethos-Core/contracts/TroveManager.sol#L934](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L934)

```solidity
    // Return the nominal collateral ratio (ICR) of a given Trove, without the price. Takes a trove's pending coll and debt rewards from redistributions into account.
```

- [Ethos-Core/contracts/TroveManager.sol#L1044](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1044)

## [NC-04] Lack of event emit

#### Description

The below methods do not emit an event when the state changes, something that it's very important for dApps and users.

#### Lines of code 

```solidity
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
        distributionPeriod = _newDistributionPeriod;
    }
```

- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122)

```solidity
    function sendOath(address _account, uint _OathAmount) external override {
        _requireCallerIsStabilityPool();

        OathToken.transfer(_account, _OathAmount);
    }
```

- [Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124-L128](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124-L128)

```solidity
    function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
        _requireCallerIsBorrowerOperations();
        Troves[_borrower][_collateral].status = Status(_num);
    }
```

- [Ethos-Core/contracts/TroveManager.sol#L1562-L1565](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1562-L1565)

```solidity
    function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address");
        treasury = newTreasury;
    }

```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631)

```solidity
    function setEmergencyExit() external {
        _atLeastRole(GUARDIAN);
        emergencyExit = true;
        IVault(vault).revokeStrategy(address(this));
    }
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L160](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L160)

```solidity
    function setHarvestSteps(address[2][] calldata _newSteps) external {
        _atLeastRole(ADMIN);
        delete steps;

        uint256 numSteps = _newSteps.length;
        for (uint256 i = 0; i < numSteps; i = i.uncheckedInc()) {
            address[2] memory step = _newSteps[i];
            require(step[0] != address(0));
            require(step[1] != address(0));
            steps.push(step);
        }
    }
```

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L171](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L160-L171)

#### Recommended Mitigation Steps

Event-emit.

## [NC-05] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external / public / internal / private`
- `view / pure`

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Follow Solidity Style Guide.

## [NC-06] Solidity compiler optimizations can be problematic

#### Description

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### Lines of code 

```javaScript
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
```

- [Ethos-Vault/hardhat.config.js#L15-L19](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/hardhat.config.js#L15-L19)

```javaScript
                version: "0.4.23",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
            },
            {
                version: "0.5.17",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
            },
            {
                version: "0.6.11",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
```

- [Ethos-Core/hardhat.config.js#L41-L65](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/hardhat.config.js#L41-L65)

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [NC-07] Mark visibility of `initialize()` functions as external

#### Description

- If someone wants to extend via inheritance, it might make more sense that the overridden `initialize()` function calls the internal {...}_init function, not the parent public `initialize()` function.

- External instead of public would give more the sense of the `initialize()` functions to behave like a constructor (only called on deployment, so should only be called externally)

- Security point of view, it might be safer so that it cannot be called internally by accident in the child contract

- It might cost a bit less gas to use external over public

- It is possible to override a function from external to public ("opening it up") but it is not possible to override a function from public to external ("narrow it down").

> Ref: https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

#### Lines of code 

```solidity
    function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
    ) public initializer {
        gWant = _gWant;
        want = _gWant.UNDERLYING_ASSET_ADDRESS();
        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
        rewardClaimingTokens = [address(_gWant)];
    }
```

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72)

#### Recommended Mitigation Steps

Change the visibility of `initialize()` functions to external

## [NC-08] Use `bytes.concat()` and `string.concat()`

#### Description

Solidity version 0.8.4 introduces: 

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

```solidity
        bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 
                         domainSeparator(), keccak256(abi.encode(
                         _PERMIT_TYPEHASH, owner, spender, amount, 
                         _nonces[owner]++, deadline))));
```

- [Ethos-Core/contracts/LUSDToken.sol#L283-L286](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L283-L286)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()`

## [NC-09] Reusable require statements should be changed to a modifier

#### Description

Reusable require statements should be changed to a modifier.

#### Lines of code 

```solidity
            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
```

- [Ethos-Core/contracts/CollateralConfig.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66)

```solidity
        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
```

- [Ethos-Core/contracts/CollateralConfig.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L94)

```solidity
            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
```

- [Ethos-Core/contracts/CollateralConfig.sol#L69](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L69)

```solidity
        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
```

- [Ethos-Core/contracts/CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97)

```solidity
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)

```solidity
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)

#### Recommended Mitigation Steps

Use modifiers.

## [NC-10] Use a more recent version of OpenZeppelin dependencies

#### Description

For security, it is best practice to use the latest OpenZeppelin version.

#### Lines of code 

```javaScript
      "dependencies": {
        "@openzeppelin/contracts": "^4.7.3",
        "@openzeppelin/contracts-upgradeable": "^4.7.3",
        "dotenv": "^16.0.1"
      },
```

- [Ethos-Vault/package-lock.json#L11-L15](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package-lock.json#L11-L15)

#### Recommended Mitigation Steps

Old version of OpenZeppelin is used `(4.7.3)`, newer version can be used [`(4.8.0)`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/package-lock.json).

## [NC-11] The protocol should include NatSpec

#### Description

It is recommended that Solidity contracts are fully annotated using NatSpec, it is clearly stated in the Solidity official documentation.

- In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

- Some code analysis programs do analysis by reading NatSpec details, if they can't see the tags `(@param, @dev, @return)`, they do incomplete analysis.

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)
- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Include [`NatSpec`](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) comments in the codebase.

## [NC-12] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
        require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");
    }
```

#### Lines of code 

`Lines: 301, 534, 568`

- [Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol) :

`Lines: 428, 469, 489, 475, 685, 720, 751, 784, 795, 809`

- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

`Lines: 616, 817, 1127, 1142, 1215, 1238`

- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

`Lines: 152, 210, 227, 296, 331, 380, 466, 555`

- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

`Lines: 52, 67, 140, 184`

- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

`Lines: 121`

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

#### Recommended Mitigation Steps

```solidity
    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {
        require(0 == _collTopUp || 0 == _collWithdrawal, "BorrowerOperations: Cannot withdraw and add coll");
    }
```

## [NC-13] Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

#### Lines of code 

```solidity
pragma solidity 0.6.11;
```

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Old versions of Solidity are used `(0.8.0)`, `(0.6.11)`, newer version can be used `(0.8.17)`.

## [NC-14] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

#### Lines of code

```solidity
- What is the overall line coverage percentage provided by your tests?:  93
```

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [NC-15] Signature Malleability of EVM's `ecrecover()`

#### Description

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin's ECDSA library.

#### Lines of code 

```solidity
        address recoveredAddress = ecrecover(digest, v, r, s);
```

- [Ethos-Core/contracts/LUSDToken.sol#L287](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287)

#### Recommended Mitigation Steps

Use the ecrecover function from OpenZeppelin's ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

## [NC-16] Add a timelock to critical functions

#### Description

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate.

#### Lines of code 

```solidity
    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
        _requireValidCollateralAddress(_collateral);
        require(_bps <= 10_000, "Invalid BPS value");
        yieldingPercentage[_collateral] = _bps;
        emit YieldingPercentageUpdated(_collateral, _bps);
    }
```

- [Ethos-Core/contracts/ActivePool.sol#L125-L130](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L125-L130)

```solidity
    function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
        require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");
        yieldingPercentageDrift = _driftBps;
        emit YieldingPercentageDriftUpdated(_driftBps);
    }
```

- [Ethos-Core/contracts/ActivePool.sol#L132-L136](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132-L136)

```solidity
    function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
        _requireValidCollateralAddress(_collateral);
        yieldClaimThreshold[_collateral] = _threshold;
        emit YieldClaimThresholdUpdated(_collateral, _threshold);
    }
```

- [Ethos-Core/contracts/ActivePool.sol#L138-L142](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138-L142)

```solidity
    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
        yieldSplitTreasury = _treasurySplit;
        yieldSplitSP = _SPSplit;
        yieldSplitStaking = _stakingSplit;
        emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
    }
```

- [Ethos-Core/contracts/ActivePool.sol#L144-L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144-L150)

#### Recommended Mitigation Steps

Consider adding a timelock to the critical changes.

## [NC-17] Use `immutable` instead of `constant` for values such as a call to `keccak256()`

#### Description

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### Lines of code 

```solidity
    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73)

```solidity
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74)

```solidity
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75)

```solidity
    bytes32 public constant ADMIN = keccak256("ADMIN");
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76)

```solidity
    bytes32 public constant KEEPER = keccak256("KEEPER");
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49)

```solidity
    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50)

```solidity
    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51)

```solidity
    bytes32 public constant ADMIN = keccak256("ADMIN");
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52)

#### Recommended Mitigation Steps

Use `immutable` instead of `constants` 

## [NC-18] Need Fuzzing test

#### Description

Use should fuzzing test like Echidna. As Alberto Cuesta Canada said: 

> Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Ref: https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

## [NC-19] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [NC-20] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally. 

> Ref: https://swcregistry.io/docs/SWC-103

#### Lines of code 

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)

```solidity
pragma solidity ^0.8.0;
```

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/): Lock pragmas to specific compiler version. 

## [NC-21] Consider using `delete` rather than assigning zero to clear values    

#### Description

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

#### Lines of code 

`Lines: 253`

- [Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol)

`Lines: 529, 578, 756`

- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

`Lines: 387, 388, 468, 469, 505, 506, 1192, 1285, 1286, 1288, 1289`

- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

`Lines: 215, 370, 372, 538`

- [Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

`Lines: 100, 112, 117`

- [Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)


#### Recommended Mitigation Steps

Use the `delete` keyword.

## [NC-22] Adding a `return` statement when the function defines a named return variable, is redundant

#### Description

Adding a `return` statement when the function defines a named return variable, is redundant.

#### Lines of code 

```solidity
        return (collGainPerUnitStaked, LUSDLossPerUnitStaked);
```

- [Ethos-Core/contracts/StabilityPool.sol#L543](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L543)

```solidity
        return singleLiquidation;
```

- [Ethos-Core/contracts/TroveManager.sol#L353](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L353)

```solidity
        return singleLiquidation;
```

- [Ethos-Core/contracts/TroveManager.sol#L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L436)

```solidity
        return index;
```

- [Ethos-Core/contracts/TroveManager.sol#L1332](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1332)

## [NC-23] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Follow solidity standard [naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).

## [NC-24] Events that mark critical parameter changes should contain both the old and the new value

#### Description

Events that mark critical parameter changes should contain both the old and the new value.

#### Lines of code 

```solidity
        emit YieldingPercentageUpdated(_collateral, _bps);
```

- [Ethos-Core/contracts/ActivePool.sol#L129](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L129)

```solidity
        emit YieldingPercentageDriftUpdated(_driftBps);
```

- [Ethos-Core/contracts/ActivePool.sol#L135](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L135)

```solidity
        emit YieldClaimThresholdUpdated(_collateral, _threshold);
```

- [Ethos-Core/contracts/ActivePool.sol#L141](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L141)

```solidity
        emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
```

- [Ethos-Core/contracts/ActivePool.sol#L149](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L149)

#### Recommended Mitigation Steps

Add the old value to the event.

## [NC-25] Add NatSpec Mapping comment  

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
mapping (address => uint256) private _nonces;  
```

#### Lines of code 

`Lines: 35`

- [Ethos-Core/contracts/CollateralConfig.sol#L35](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)

`Lines: 54, 57, 58, 63, 64, 65`

- [Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

`Lines: 167, 179`

- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

`Lines: 88, 91, 94, 104, 105, 119, 120`

- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

`Lines: 28, 32, 35`

- [Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

#### Recommended Mitigation Steps

```solidity
/// @dev address(owner) => uint(nonce)
mapping (address => uint256) private _nonces;
```
## [NC-26] Constants should be defined rather than using magic numbers 

#### Description

Constants should be defined rather than using magic numbers.

#### Lines of code

```solidity
        require(_bps <= 10_000, "Invalid BPS value");
```

- [Ethos-Core/contracts/ActivePool.sol#L127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127)

```solidity
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
```

- [Ethos-Core/contracts/ActivePool.sol#L145](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L145)

```solidity
        vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
```

- [Ethos-Core/contracts/ActivePool.sol#L258](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L258)

```solidity
        vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
```

- [Ethos-Core/contracts/ActivePool.sol#L262](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L262)

```solidity
                vars.treasurySplit = vars.profit.mul(yieldSplitTreasury).div(10_000);
```

- [Ethos-Core/contracts/ActivePool.sol#L291](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L291)

```solidity
                vars.stakingSplit = vars.profit.mul(yieldSplitStaking).div(10_000);
```

- [Ethos-Core/contracts/ActivePool.sol#L296](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L296)

#### Recommended Mitigation Steps

Define a constant.

## [NC-27] There is no need to cast a variable that is already an address, such as `address(x)`

#### Description

There is no need to cast a variable that is already an `address`, such as `address(x)`, `x` is also `address`.

#### Lines of code 

```solidity
        (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
            address(want),
            address(this)
        );
```

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L232](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L232)

#### Recommended Mitigation Steps

```solidity
        (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
            want,
            address(this)
        );
```

## [NC-28] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Ref: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

#### Recommended Mitigation Steps

Use SMTChecker

## [NC-29] Not using the named return variables anywhere in the function is confusing

#### Description

Not using the named return variables anywhere in the function is confusing.

#### Lines of code

`Lines: 626`

- [Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

`Lines: 1316`

- [Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

`Lines: 89, 104`

- [Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

`Lines: 29, 37, 51, 66, 79, 96, 110, 122, 165, 220, 240, 269`

- [Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

#### Recommended Mitigation Steps

Consider changing the variable to be an unnamed one.

## [NC-30] Use `uint256` instead `uint`

#### Description

`477 results in 8 files`.

Some developers prefer to use `uint256` because it is consistent with other `uint` data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

For example if doing `bytes4(keccak('transfer(address, uint)’))`, you'll get a different method sig ID than `bytes4(keccak('transfer(address, uint256)’))` and smart contracts will only understand the latter when comparing method sig IDs.

#### Lines of code 

- [Ethos-Core/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts)

- [Ethos-Vault/contracts](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts)

#### Recommended Mitigation Steps

Use `uint256` instead `uint`.

## [NC-31] Use scientific notation rather than exponentiation

#### Description

While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist.

#### Lines of code

```solidity
        uint usdValue = _price.mul(_coll).div(10**_collDecimals);
```

- [Ethos-Core/contracts/BorrowerOperations.sol#L433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L433)

```solidity
            cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));
```

- [Ethos-Core/contracts/TroveManager.sol#L494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L494)

```solidity
            cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));
```

- [Ethos-Core/contracts/TroveManager.sol#L496](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L496)

```solidity
        uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);
```

- [Ethos-Core/contracts/TroveManager.sol#L1132](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1132)

```solidity
        uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);
```

- [Ethos-Core/contracts/TroveManager.sol#L1147](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1147)

```solidity
        uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);
```

- [Ethos-Core/contracts/TroveManager.sol#L1251](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1251)

```solidity
        uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);
```

- [Ethos-Core/contracts/TroveManager.sol#L1252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1252)

```solidity
    uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L40](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40)

```solidity
        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)

```solidity
        return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();
```

- [Ethos-Vault/contracts/ReaperVaultV2.sol#L296](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L296)

#### Recommended Mitigation Steps

Use scientific notation `(e.g. 1e18)` rather than exponentiation `(e.g. 10**18)`.
