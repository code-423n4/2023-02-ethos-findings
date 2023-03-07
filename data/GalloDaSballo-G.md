# Executive Summary

There's an abundant amount of opportunity for refactorings, the below report offers extremely high savings opportunity with minimal work by reducing SSTOREs and SLOADs

All benchmarks are approximations based on counting storage slots.

Savings for Core are in the tens of thousands

Savings for Vault are in the hundreds of thousands

# Default Pool

## Gas: Can be refactored to avoid giving allowance, and savings tens of thousands of gas - 20k+
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/DefaultPool.sol#L90-L91

```solidity
        IERC20(_collateral).safeIncreaseAllowance(activePool, _amount);
        IActivePool(activePoolAddress).pullCollateralFromBorrowerOperationsOrDefaultPool(_collateral, _amount);
```

Can just transfer, saving 20k gas per instance

# ActivePool

## Pack generator, amount and treshold into one struct by using u16 - 4.2k per balance
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L246

By packing them into one struct you can save 4.2k per rebalance (2 cold SLOADs)

# RedemptionHelper

## Gas - By packing you can avoid doing 2 SLOADs and calls - 2.1k+
Also have a function "getConfig" or smth

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/RedemptionHelper.sol#L120-L121

```solidity
        totals.collDecimals = collateralConfigCached.getCollateralDecimals(_collateral);
        totals.collMCR = collateralConfigCached.getCollateralMCR(_collateral);
```

## Gas - Cache token `lusdToken` - 100 gas
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/RedemptionHelper.sol#L124-L128

```solidity
        _requireLUSDBalanceCoversRedemption(lusdToken, _redeemer, _LUSDamount);

        totals.totalLUSDSupplyAtStart = getEntireSystemDebt(_collateral);
        // Confirm redeemer's balance is less than total LUSD supply
        assert(lusdToken.balanceOf(_redeemer) <= totals.totalLUSDSupplyAtStart);
```


# Vault

## Using smaller variables to save Storage Slots can save tens of thousands of gas - 8 Slots = 16.8k+ per read and 160k+ per store

ReaperVaultV2 could benefit by using smaller uints for most values

```solidity
    uint256 public tvlCap;

    uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
    uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
    uint256 public lastReport; // block.timestamp of last report from any strategy

    uint256 public immutable constructionTime;
    bool public emergencyShutdown;

    // The token the vault accepts and looks to maximize.
    IERC20Metadata public immutable token;

    // Max slippage(loss) allowed when withdrawing, in BPS (0.01%)
    uint256 public withdrawMaxLoss = 1;
    uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
    uint256 public lockedProfit; // how much profit is locked and cant be withdrawn
```

Can be refactored to

```solidity
    uint64 public tvlCap; // Multiply the cap by 10 ** decimals and save a ton of space

    uint16 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
    uint16 public totalAllocated; // Amount of tokens that have been allocated to all strategies

    uint40 public lastReport; // block.timestamp of last report from any strategy
    bool public emergencyShutdown;

    // Max slippage(loss) allowed when withdrawing, in BPS (0.01%)
    uint16 public withdrawMaxLoss = 1;
    uint16 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
    uint16 public lockedProfit; // how much profit is locked and cant be withdrawn
```



## Packing of Distribution Types - 2.1k

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/libraries/DistributionTypes.sol#L6-L12

```solidity
    struct RewardsConfigInput {
        uint88 emissionPerSecond;
        uint256 totalSupply;
        uint32 distributionEnd;
        address asset;
        address reward;
    }
```

Can be refactored with no precision loss to:

```solidity
    struct RewardsConfigInput {
        uint256 totalSupply;
        uint32 distributionEnd;
        address asset;
        address reward;
        uint88 emissionPerSecond;
    }
```

Which will save an additional storage slot


## Packing on StrategyParams - 6.3k+

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L25-L33

```solidity
    struct StrategyParams {
        uint256 activation; // Activation block.timestamp
        uint256 feeBPS; // Performance fee taken from profit, in BPS
        uint256 allocBPS; // Allocation in BPS of vault's total assets
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
        uint256 lastReport; // block.timestamp of the last time a report occured
    }
```

Can be refactored with zero precision loss to:

```solidity
    struct StrategyParams {
        uint40 activation; // Activation block.timestamp
        uint16 feeBPS; // Performance fee taken from profit, in BPS
        uint16 allocBPS; // Allocation in BPS of vault's total assets
        uint40 lastReport; // block.timestamp of the last time a report occured
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
    }
```

Which would save 3 slots, saving 3 SSTOREs when setting the values as well as `report`ing

This could be further reduced by scaling `allocated`, `gains` and `losses` by using a multiplication factor such as 1e18, which will cause negligible precision loss at the advantage of saving one additional storage slot

## Can use unchecked - 80

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81-L82

```solidity
            uint256 toReinvest = wantBalance - _debt;

```

There are plenty of instances where the check before the operation ensures no overflow can happen, in those cases you can use `unchecked` and save around 50 / 80 gas