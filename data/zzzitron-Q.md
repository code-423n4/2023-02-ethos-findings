## Summary

Risk | Title
---|---
NC01 | Wrong documentation concerning the term "Total active debt" and/or wrong calculation of the total active debt
NC02 | Duplicate import statements for the same contract
NC03 | Unused function in the `BorrowerOperations` contract
NC04 | Unused state variables in `StabilityPool.sol`
NC05 | Inconsistent use of the types uint and uint256
NC06 | Bad readability of source code, using `uint256(-1)`
NC07 | Inconsistent and hard to read value assignment of `10000` to the const variable `PERCENT_DIVISOR`
L01 | `BorrowerOperations` and `RedemptionHelper` are setting the trove status with raw values
L02 | Mixed compiler versions in `Ethos-Vault`
L03 | Old compiler version in `Ethos-Core`
L04 | No possibility to transfer ownership
L05 | Possible frontrunning issues when liquidating a trove or when adding collateral to a trove
L06 | Missing NatSpec tags in function comments
L07 | Overflow in function `ActivePool.setYieldDistributionParams` due to the usage of unsafe math
L08 | Unsafe arithmetic operation used in `ActivePool._rebalance`
L09 | Unsafe cast from uint256 to int256 in `ActivePool._rebalance`
L10 | Frontrunnable `ReaperBaseStrategyv4.harvest` function can result in getting a bad slippage
L11 | Deprecated usage of emitting an Event
L12 | Misleading naming of variable `ActivePool.yieldingPercentage` and `LocalVariables_rebalance.yieldingPercentage`
L13 | Misleading function name `ActivePool._requireCallerIsBOorTroveMorSP`
L14 | Bad english in comment in `ReaperVaultV2.sol`
L15 | Misleading and wrong comment in `ReaperVaultV2.sol`

## Non-critical

### NC01: Wrong documentation concerning the term "Total active debt" and/or wrong calculation of the total active debt

https://docs.reaper.farm/ethos-reserve-bounty-hunter-documentation/useful-information/glossary

In the documentation there is a glossary section which describes the term "Total active debt":

```
Total active debt: the sum of active debt over all Troves. Equal to the LUSD in the ActivePool.
```

There are a few things problematic and misleading:

1) The description of the term "Total active debt" simplifies and doesn't capture the fact that the total active debt is in relation to a certain collateral:

```solidity
ActivePool
// audit Note: LUSDDebt is the variable that tracks the total active debt in relation
// to a certain collateral, which is why it is of type mapping (address => uint256), where the address
// is the address of the collateral for which the debt is tracked.
42    mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker
```

2) What's especially misleading and wrong in the current glossary is, that the second sentence "Equal to the LUSD in the ActivePool" suggests, that it is the LUSD balance of the ActivePool contract, calculated like this:

```solidity
lusdToken.balanceOf(address(activePool));
```
In reality it is not the LUSD balance of the ActivePool, but it is tracked via the state variable: LUSDDebt[collateral];
 
As a suggestion, the glossary should read:

```
 Total active debt: the sum of active debt for a certain collateral over all Troves.
 Equal to the LUSD debt tracked in the ActivePool.
 
 Technical note:
 The "Total active debt" for a collateral is stored in the state variable
 `ActivePool.LUSDDebt` which is of type mapping (address => uint256)
 and which maps the collateral address to the uint256 amount of total active debt for the collateral.
```

Current technical implementation:

```solidity
// ActivePool
// audit Note: The function below returns the total active debt for _collateral
164    function getLUSDDebt(address _collateral) external view override returns (uint) {
165        _requireValidCollateralAddress(_collateral);
166        return LUSDDebt[_collateral];
167    }
```

### NC02: Duplicate import statements for the same contract

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L5

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L8

`IBorrowerOperations.sol` is imported twice in the `StabilityPool.sol` on line 5 and line 8. There is no need to import the same contract twice, so one of these lines should be deleted.

### NC03: Unused function in the `BorrowerOperations` contract

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L248

The function `moveCollateralGainToTrove` is unused. According to its comment and to its require statement `_requireCallerIsStabilityPool`, it can only be called by the `StabilityPool`. But the `StabilityPool` contract is not calling this function.

### NC04: Unused state variables in `StabilityPool.sol`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L152

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L160

There are unused state variables in `StabilityPool.sol`:

- The state variable `borrowerOperations` (line 152) is unused. It is only initialized in the `StabilityPool.setAddresses` function, but after initializing it there is no further usage. It is recommended to remove this state variable and also to remove the IBorrowerOperations interface from the `StabilityPool` contract since both are not used or needed.

- The state variable `lqtyTokenAddress` (line 160) is unused. It is only declared, but nowhere initialized or used. After deploying the `StabilityPool` contract, `lqtyTokenAddress` will always point to address 0, therefore it's recommended to remove this state variable.

### NC05: Inconsistent use of the types uint and uint256

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L731-L732

On line 731 the `_price` function parameter is of type `uint`, which is consistent to most other function parameters, with the exception of the parameter  `_collateralDecimals` on line 732, which is of type `uint256`.

It is recommended to use `uint` for the type of the param `_collateralDecimals`, to make it consistent.

### NC06: Bad readability of source code, using `uint256(-1)`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L258

It's recommended to use `type(uint256).max` for better readability instead of `uint256(-1)`.

### NC07: Inconsistent and hard to read value assignment of `10000` to the const variable `PERCENT_DIVISOR`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L41

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23

In the `ReaperVaultV2` contract the value assigned to `PERCENT_DIVISOR` is hard to read, since it's lacking an underscore, and it should be consistent to the `ReaperBaseStrategyv4` contract.

```solidity
// ReaperVaultV2
41    uint256 public constant PERCENT_DIVISOR = 10000;
```

```solidity
// ReaperBaseStrategyv4
23    uint256 public constant PERCENT_DIVISOR = 10_000;
```

## Low

### L01: `BorrowerOperations` and `RedemptionHelper` are setting the trove status with raw values

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L219

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L397

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/RedemptionHelper.sol#L266

`BorrowerOperations.sol` is calling the function `setTroveStatus` (line 219) with the raw value 1 as the third function parameter, to set the trove to the Status.active (`TroveManager.sol` line 1564).

Similar issues exist in `BorrowerOperations.sol` line 397 and `RedemptionHelper.sol` line 266, where the function `closeTrove` is called with hardcoded raw values.

When the `Status` enum (`TroveManager.sol` line 68) changes in the future due to further system development, it will be hard to track all the necessary fixes in the source code.

Recommendation to fix:

```solidity
// BorrowerOperations
// openTrove
219         contractsCache.troveManager.setTroveStatus(msg.sender, _collateral, uint(Status.active));
```

```solidity
// RedemptionHelper
// _redeemCollateralFromTrove
266            troveManager.closeTrove(_borrower, _collateral, uint(Status.closedByRedemption));
```

```solidity
// BorrowerOperations
// closeTrove
397:         troveManagerCached.closeTrove(msg.sender, _collateral, uint(Status.closedByOwner));
```



### L02: Mixed compiler versions in `Ethos-Vault`

Compiler version 0.8.11:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/interfaces/IRewardsController.sol#L2

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/interfaces/IRewardsDistributor.sol#L2

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/libraries/DistributionTypes.sol#L2


Compiler version 0.8.0:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

ReaperStrategyGranarySupplyOnly using IRewardsController:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L207

`IRewardsController.sol` is using compiler version 0.8.11. `ReaperStrategyGranarySupplyOnly.sol` is using compiler version 0.8.0.

It's a good practice to use a single, fixed compiler version among a single codebase.

### L03: Old compiler version in `Ethos-Core`

The Ethos-Core is using the old solidity compiler version 0.6.11 which is an outdated version. The compiler version should be updated to at least version 0.8.0.

Be aware that updating to solidity compiler version to 0.8.0 can break the code, since arithmetic overflow and underflow will revert.

For more information of breaking changes when updating the compiler version, check here:

https://docs.soliditylang.org/en/v0.8.19/080-breaking-changes.html

### L04: No possibility to transfer ownership

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/Ownable.sol#L17-L66

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L125

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L85-L89

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101

The `Ownable` contract in the `Ethos-Core` doesn't offer a method to transfer the ownership. Most contracts that are using the `Ownable` contract, renounce the ownership at the end of the initializer function. For example in the function `StabilityPool.setAddresses` at line 302 `_renounceOwnership()` is called.

Not all contracts in `Ethos-Core` renounce the ownership, because they want to protect access to some functions with the `onlyOwner` modifier: `ActivePool`, `CollateralConfig`, `CommunityIssuance`. All of them keep the owner address, and it might be important and helpful to be able to transfer the ownership to another address in the future:

- Currently, the owner account is a single point of failure, because it isn't possible to assign it to another address.

- Additionally, if the owner account is compromised, there is no way to transfer the ownership.

### L05: Possible frontrunning issues when liquidating a trove or when adding collateral to a trove

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L310

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L242

Both liquidating a trove and adding collateral to a trove can be frontrunned.

Example of frontrunning liquidations:

`Alice` tries to call the `liquidate` function (line 310 TroveManager.sol) to liquidate `Bob's` trove. `Bob`, who observes the transactions, frontruns `Alice's` liquidate transaction, by calling the function `addColl` (line 242 BorrowerOperations.sol) to add enough collateral to his trove, so that his trove can't be liquidated any more. Resulting in `Alice's` valid attempt to liquidate `Bob's` trove to fail, since it was frontrunned.

Example of frontrunning someone who wants to add collateral to his trove:

`Bob's` trove is in danger to be liquidated, so he wants to add collateral to remove the risk that his trove gets liquidated. `Bob` calls the function `addColl` (line 242 BorrowerOperations.sol) but he gets frontrunned by `Alice` who calls the function `liquidate` (line 310 TroveManager.sol) to liquidate `Bob's` trove, before `Bob` can add enough collateral that would have avoided that his trove gets liquidated.


### L06: Missing NatSpec tags for function comments

There are a few functions in the codebase that are using NatSpac tags like @notice and @params. But most functions are missing these NatSpac tags.

As an example, the function `ReaperStrategyGranarySupplyOnly.initialize` (line 62) has comments on line 60 with the NatSpac tag @notice, describing what this function does.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L60-L62

Most other functions inside the `ReaperStrategyGranarySupplyOnly` contract don't have any NatSpac tags. This is not very consistent and it makes it harder to understand what certain functions are doing, because the comments are missing.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L74-L230

Most contract functions inside `Ethos-Core` and `Ethos-Vault` don't have @notice, @param and @return NatSpac tags. Some examples:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L410

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L310

Etherscan (etherscan.io) uses the information from the NatSpac tags, but if they are missing, it will not work for Etherscan.

### L07: Overflow in function `ActivePool.setYieldDistributionParams` due to the usage of unsafe math

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L145

The function `setYieldDistributionParams` uses a require statement (line 145) which checks, that the three function parameters are equal to 10000, in order to make sure that the splits add up to 10000 BPS.

The problem is that this can overflow, because unsafe math is used and because the `ActivePool` contract uses the old solidity compiler version 0.6.11, which doesn't prevent overflows.

Here is an example test contract that proves the issue. Inside the test function `setParamsToOverflow`, `_treasurySplit` is set to type(uint256).max, `_SPSplit` is set to 1 and `_stakingSplit` is set to 10_000. When this function is called, it doesn't revert which proves the overflow.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity 0.6.11;

contract TestOverflow {
    function setParamsToOverflow() public pure {
        // This function does not revert, the last require in this function passes 
        // due to the overflow.

        uint256 _treasurySplit = type(uint256).max;
        uint256 _SPSplit = 1;

        // Check overflow:
        require(_treasurySplit + _SPSplit == 0, "Did not overflow");

        uint256 _stakingSplit = 10_000;
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
    }
}
```

The above example contract `TestOverflow` also illustrates, that it is possible to set the value of a split param (`_SPSplit` or `_stakingSplit` or `_treasurySplit`) to the value of type(uint256).max and the other two split params to the values 1 and 10000. This should not be possible.

The recommended fix of this issue is to use the safe `add` function, because there is no reason that the operation should overflow:

```solidity
// ActivePool
145        require(_treasurySplit.add(_SPSplit).add(_stakingSplit) == 10_000, "Splits must add up to 10000 BPS");
```

### L08: Unsafe arithmetic operation used in `ActivePool._rebalance`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L277

When calculating `vars.netAssetMovement`, an unsafe arithmethic operation is used on line 277, which can cause an underflow. It is recommended to use the `sub` function from the SafeMath library, because there is no reason that it should underflow.

### L09: Unsafe cast from uint256 to int256 in `ActivePool._rebalance`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L277

The uint256 variables `vars.toDeposit`, `vars.toWithdraw`, `vars.profit` are all cast to int256 (line 277). These casts from uint256 to int256 can cause an overflow, if one of the values of these variables is higher than max(int256).max.

It would be better to use the `SafeCast` library from OpenZepplin to do the cast.


### L10: Frontrunnable `ReaperBaseStrategyv4.harvest` function can result in getting a bad slippage

The `ReaperBaseStrategyv4.harvest` function calls `ReaperBaseStrategyv4._harvestCore` function.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L131

Then the `ReaperBaseStrategyv4._harvestCore` function calls the `VeloSolidMixin._swapVelo` function.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L124

Then the `VeloSolidMixin._swapVelo` function calls `router.swapExactTokensForTokens` with the value 0 for the function parameter `amountOutMin`, which means that there is no check for slippage. If the trade results in a bad deal, there is no protection against frontrunning/slippage.

The `router` is the `VeloRouter` and the address on the optimism is `0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9`. 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/mixins/VeloSolidMixin.sol#L41

https://optimistic.etherscan.io/address/0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9#code

The issue is that if a malicious actor frontruns the call of the `ReaperBaseStrategyv4.harvest` function, to force a bad deal. This would be done for example when the malicious actor tries to execute a sandwich-attack.

As a mitigation, the `amountOutMin` value for the `router.swapExactTokensForTokens` function should not be 0, but should be based on the price from the oracle.

### L11: Deprecated usage of emitting an Event

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L194

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L201

In the `ActivePool` contract (line 194, line 201), the `ActivePoolLUSDDebtUpdated` event is invoked without the `emit` prefix. 

It is deprecated to invoke events without the `emit` prefix, and usually results in a compiler warning.

Also, the `emit` keyword was introduced to make events stand out with regards to regular function calls.

### L12: Misleading naming of variable `ActivePool.yieldingPercentage` and `LocalVariables_rebalance.yieldingPercentage`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L44

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L228


The variable name `ActivePool.yieldingPercentage` is misleading, because it is not the percentage rate but the BPS rate of collateral to use for yield farming.

The same issue exists with `LocalVariables_rebalance.yieldingPercentage` (line 228), which is also the BPS rate.

It's recommended to rename:

`ActivePool.yieldingPercentage` -> rename to `ActivePool.yieldingBPS`
`LocalVariables_rebalance.yieldingPercentage` -> rename to `LocalVariables_rebalance.yieldingBPS`

### L13: Misleading function name `ActivePool._requireCallerIsBOorTroveMorSP`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L327

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L332

The function name suggests, that it is required that the caller is either: `BorrowerOperations` (BO), `TroveManager` (TroveM) or the `StabilityPool` (SP). This is misleading, because the require statement also checks whether the caller (msg.sender) is the `RedemptionHelper` (see line 332).

It's recommended to rename:

`ActivePool._requireCallerIsBOorTroveMorSP` -> rename to `ActivePool._requireCallerIsBOorTroveMorSPorRH`

Also, the revert message should be updated to:

```solidity
// ActivePool
334            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool nor RedemptionHelper");
```

### L14: Bad english in comment in `ReaperVaultV2.sol`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L274

The comment on line 274 is using the word "value", but it would be more clear and understandable when using the word "amount" or "balance" instead. It's not the "value" of the token that is calculated by the function `ReaperVaultV2.balance`, but the "amount" or "balance" of the token. That would also be in accordance to the function name, which is "balance".

Recommendation:

```solidity
// ReaperVaultV2
274     * @dev It calculates the total underlying balance of {token} held by the system.
```

### L15: Misleading and wrong comment in `ReaperVaultV2.sol`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L181

On line 155 and 181 in the `ReaperVaultV2` contract, the error message from the require statement reads:

"Fee cannot be higher than 20 BPS".

It should read: "Fee cannot be higher than 2000 BPS" or "Fee cannot be higher than 20 percent"


Because the result from the calculation `PERCENT_DIVISOR` / 5 is 2000 BPS, since `PERCENT_DIVISOR` is defined as:

```solidity
// ReaperVaultV2
41    uint256 public constant PERCENT_DIVISOR = 10000;
```