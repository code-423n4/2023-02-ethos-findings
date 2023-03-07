# Summary
## Gas Optimizations List:
| Number |Issues Details|Instances|Gas Saved|
|:--:|:-------|:--:|:--:|
|[G-1]| Reconstructing mapping struct would save on extra storage slots | 4 | 198900 |
|[G-2]| One time set variables can be defined as immutable | 64 | 134400 |
|[G-3]| LUSDToken can be optimized using ERC20 solmate implementation | 1 | ~ |
|[G-4]| Extra unecessary check overflow, when having a require or if check | 1 | 200 |
|[G-5]| Caching reused storage variables | 33 | 3300 |
|[G-6]| Using return variable area to assign returned value | 45 | 135 |
|[G-7]| Reordering if checks to save on extra check | 1 | 3 |
|[G-8]| Define variables outside of the for loop | 11 | ~ |
|[G-9]| Operations safe to be unchecked | 18 | 981 |

 **~ refers to the fact that the Gas Saved would vary**

### [G-1] Reconstructing mapping struct would save on extra storage slots

In multiple cases, the mappings with the same key are not being efficiently packed, due to the fact that they are being defined seperately. by combining them together in a single mapping, we would be able to tightly pack them together, saving a couple of Gsset storage slots.


3 instances - 3 files

1. ReaperVaultV2.sol: (Saving 3 (Gsset+Gcoldsload) = 3 * (20000+2100) = 66300 gas)
```solidity
    // from:
    struct StrategyParams {
        uint256 activation; // Activation block.timestamp // uint64 is enough
        uint256 feeBPS; // Performance fee taken from profit, in BPS // max value is 2000 uint16
        uint256 allocBPS; // Allocation in BPS of vault's total assets // max value is 10000 uint16
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
        uint256 lastReport; // block.timestamp of the last time a report occured // uint64 is enough
    }
    mapping(address => StrategyParams) public strategies;

    // to:
    struct StrategyParams {
        uint64 activation; // Activation block.timestamp // it's a timestamp <=> uint64
        uint64 lastReport; // block.timestamp of the last time a report occured // it's a timestamp <=> uint64
        uint64 feeBPS; // Performance fee taken from profit, in BPS // max value is 2000 therefore uint16
        uint64 allocBPS; // Allocation in BPS of vault's total assets // max value is 10000 therefore uint16
        uint256 allocated; // Amount of capital allocated to this strategy
        uint256 gains; // Total returns that Strategy has realized for Vault
        uint256 losses; // Total losses that Strategy has realized for Vault
    }
    mapping(address => StrategyParams) public strategies;
```
2. PriceFeed.sol: (Saving 1 (Gsset+Gcoldsload) = (20000+2100) = 22100 gas)
```solidity
    // from
    mapping (address => bytes32) public tellorQueryId;  // collateral => Tellor query ID
    mapping (address => uint) public lastGoodPrice;
    mapping (address => AggregatorV3Interface) public priceAggregator;  // collateral => Mainnet Chainlink aggregator
    mapping (address => Status) public status;

    // to
    struct PriceFeedParams {
        bytes32 tellorQueryId; // 256 bits
        uint256 lastGoodPrice; // 256 bits
        address aggregator; // 160 bits
        Status status; // 8 bits
    }
    mapping (address => PriceFeedParams) public priceFeed;
```
3. CollateralConfig.sol: (Saving 3 (Gsset+Gcoldsload) = 3 * (20000+2100) = 66300 gas)
```solidity
    // from
    struct Config {
        bool allowed;
        uint256 decimals;
        uint256 MCR;
        uint256 CCR;
    }

    // to
    struct Config {
        uint96 MCR;
        uint96 CCR;
        uint8 decimals;
        bool allowed;
    }
```
4. LUSDToken.sol: (Saving 2 (Gsset+Gcoldsload) = 2 * (20000+2100) = 44200 gas)
```solidity
    // from
    mapping(address => bool) public troveManagers;
    mapping(address => bool) public stabilityPools;
    mapping(address => bool) public borrowerOperations;

    // to
    struct Roles {
        bool isTroveManager;
        bool isStabilityPool;
        bool isBorrowerOperation;
    }
    mapping (address => Roles) public addresses;
```

### [G-2] One time set variables can be defined as immutable

storage variables that are only setup one-time, they should be declared as immutable when possible, to be stored in the contract code, instead of storage, which would be cheaper to access. it would save 2100 gas difference on each storage access. it is being missued by setAddresses functions or assigned in the constructor on multiple contexts:

12 instances - 12 files

| Instances |File | # Gcoldsload saved|Gas Saved|
|:--:|:-------|:--:|:--:|
|[I-1]| ActivePool.sol | 8 | 16800 |
|[I-2]| BorrowerOperations.sol | 12 | 25200 |
|[I-3]| CollSurplusPool.sol | 4 | 8400 |
|[I-4]| DefaultPool.sol | 3 | 6300 |
|[I-5]| PriceFeed.sol | 2 | 4200 |
|[I-6]| SortedTroves.sol | 2 | 4200 |
|[I-7]| StabilityPool.sol | 8 | 16800 |
|[I-8]| TroveManager.sol | 13 | 27300 |
|[I-9]| LQTYStaking.sol | 6 | 12600 |
|[I-10]| ReaperStrategyGranarySupplyOnly.sol | 2 | 4200 |
|[I-11]| ReaperVaultV2.sol | 2 | 4200 |
|[I-12]| ReaperBaseStrategyv4.sol | 2 | 4200 |
|[Total]| - | 64 | 134400 |

### [G-3] LUSDToken can be optimized using ERC20 solmate implementation

inspired by [solmate ERC20 implementation](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC20.sol), there are savings that can be used while not compromising on security, changes mainly affect the following functions:

LUSDToken.sol:

```solidity
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) external override returns (bool) {
        _requireValidRecipient(recipient);
        _transfer(sender, recipient, amount);

        // save on unecessary allowance deduction when approval is maxxed out
        if (amount != type(uint256).max) {
            _approve(
                sender,
                msg.sender,
                _allowances[sender][msg.sender].sub(
                    amount,
                    "ERC20: transfer amount exceeds allowance"
                )
            );
        }

        return true;
    }
    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal {
        assert(sender != address(0));
        assert(recipient != address(0));

        _balances[sender] = _balances[sender].sub(
            amount,
            "ERC20: transfer amount exceeds balance"
        );

        // Cannot overflow because the sum of all user
        // balances can't exceed the max uint256 value.

        _balances[recipient] = _balances[recipient] + amount;
        emit Transfer(sender, recipient, amount);
    }

    function _mint(address account, uint256 amount) internal {
        assert(account != address(0));
        // tbd gas suggest gas efficient ERC20
        _totalSupply = _totalSupply.add(amount);

        // Cannot overflow because the sum of all user
        // balances can't exceed the max uint256 value.
        _balances[account] = _balances[account] + amount;
        emit Transfer(address(0), account, amount);
    }

    function _burn(address account, uint256 amount) internal {
        assert(account != address(0));

        _balances[account] = _balances[account].sub(
            amount,
            "ERC20: burn amount exceeds balance"
        );

        // Cannot underflow because a user's balance
        // will never be larger than the total supply.
        _totalSupply = _totalSupply - amount;
        emit Transfer(account, address(0), amount);
    }
```

### [G-4] Extra unecessary check overflow, when having a require or if check

there is an instance, where SafeMath.sub is unecessarily used, when it was preceeded with an assert check which would always ensure it would never underflow, this small optimization would save 200 gas.

1 instance - 1 file

File: StabilityPool.sol

```solidity
    function _updateRewardSumAndProduct(
        address _collateral,
        uint256 _collGainPerUnitStaked,
        uint256 _LUSDLossPerUnitStaked
    ) internal {
        uint256 currentP = P;
        uint256 newP;

        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
        // ...
        // -> you may substracrt right away, it's already validated in the assert check
        uint256 newProductFactor = DECIMAL_PRECISION - _LUSDLossPerUnitStaked;
        // ...
``` 

### [G-5] Caching reused storage variables

calling into storage variable multiple times, would incur a warm storage sload, it is advised to cache storage variables locally, so it can be reused. It would save on a 1 Gwarmaccess which is 100 gas per cached storage variable.

33 instances - 3 files

File: TroveManage.sol

```solidity
    // saves 1 Gcoldsload
    function _updateStakeAndTotalStakes(address _borrower, address _collateral)
        internal
        returns (uint256)
    {
        uint256 newStake = _computeNewStake(
            _collateral,
            Troves[_borrower][_collateral].coll
        );
        uint256 oldStake = Troves[_borrower][_collateral].stake;
        uint256 newTotalStakes = totalStakes[_collateral].sub(oldStake).add(
            newStake
        ); // 1 Gcoldsload
        Troves[_borrower][_collateral].stake = newStake;
        totalStakes[_collateral] = newTotalStakes;

        emit TotalStakesUpdated(_collateral, newTotalStakes);

        return newStake;
    }
```

File: ReaperVaultERC4626.sol

```solidity
    // saves in total 6 Gcoldsload
    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
        uint _totalSupply = totalSupply(); // 1 Gcoldsload
        uint _freeFunds = _freeFunds(); // consumes (2 + 3) Gcoldsload = 5 Gcoldsload
        if (_totalSupply == 0 || _freeFunds == 0) return assets;
        return (assets * _totalSupply) / _freeFunds;
    }

    // saves on 1 Gcoldsload
    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
        uint _totalSupply = totalSupply(); // 1 Gcoldsload
        if (_totalSupply == 0) return shares;
        return (shares * _freeFunds()) / _totalSupply;
    }

    // saves on 4 Gcoldsload
    function maxDeposit(address) external view override returns (uint256 maxAssets) {
        uint _balance = balance(); // 2 Gcoldsload
        uint _tvlCap = tvlCap; // 1 Gcoldsload - with 2 instances saved
        if (emergencyShutdown || _balance >= _tvlCap) return 0;
        if (_tvlCap == type(uint256).max) return type(uint256).max;
        return _tvlCap - _balance;
    }

    // saves on 4 Gcoldsload
    function maxMint(address) external view override returns (uint256 maxShares) {
        uint _balance = balance(); // 2 Gcoldsload
        uint _tvlCap = tvlCap; 1 Gcoldsload - with 2 instances saved
        if (emergencyShutdown || _balance >= _tvlCap) return 0;
        if (_tvlCap == type(uint256).max) return type(uint256).max;
        return convertToShares(_tvlCap - _balance);
    }

    // saves on 1 Gcoldsload
    function previewMint(uint256 shares) public view override returns (uint256 assets) {
        uint _totalSupply = totalSupply(); // 1 Gcoldsload
        if (_totalSupply == 0) return shares;
        assets = roundUpDiv(shares * _freeFunds(), _totalSupply);
    }

    // saves in total 6 Gcoldsload
    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
        uint _totalSupply = totalSupply(); // 1 Gcoldsload
        uint _freeFunds = _freeFunds(); // consumes (2 + 3) Gcoldsload = 5 Gcoldsload
        if (_totalSupply == 0 || _freeFunds == 0) return 0;
        shares = roundUpDiv(assets * _totalSupply, _freeFunds);
    }
```

File: ReaperVaultERC4626.sol

```solidity
    // saves on 1 Gcoldsload
    function addStrategy(
        address _strategy,
        uint256 _feeBPS,
        uint256 _allocBPS
    ) external {
        // ...
        address _vault = IStrategy(_strategy).vault(); // 1 Gcoldsload
        require(address(this) == _vault, "Strategy's vault does not match");
        require(address(token) == _vault, "Strategy's want does not match");
    }

    // saves on 1 Gcoldsload
    function revokeStrategy(address _strategy) external {
        if (msg.sender != _strategy) {
            _atLeastRole(GUARDIAN);
        }

        uint _allocBPS = strategies[_strategy].allocBPS; // 1 Gcoldsload
        if (_allocBPS == 0) {
            return;
        }

        totalAllocBPS -= _allocBPS;
        strategies[_strategy].allocBPS = 0;
        emit StrategyRevoked(_strategy);
    }

    // saves on 2 Gcoldsload
    function availableCapital() public view returns (int256) {
        address stratAddr = msg.sender;
        uint256 _totalAllocBPS = totalAllocBPS; // 1 Gcoldsload

        if (totalAllocBPS == 0 || emergencyShutdown) {
            return -int256(strategies[stratAddr].allocated);
        }
        uint256 _balance = balance(); // 1 Gcoldsload
        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * _balance) / PERCENT_DIVISOR;
        // ...
        uint256 vaultMaxAllocation = (_totalAllocBPS * _balance) / PERCENT_DIVISOR;
        // ...
    }

    // saves on 2 Gcoldsload
    function getPricePerFullShare() public view returns (uint256) {
        uint256 _totalSupply = totalSupply(); // 1 Gcoldsload
        uint256 _decimals = decimals(); // 1 Gcoldsload
        return _totalSupply == 0 ? 10**_decimals : (_freeFunds() * 10**_decimals) / _totalSupply;
    }

    // saves on 1 Gcoldsload
    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {

        // ...
        uint256 _totalSupply = totalSupply(); // 1 Gcoldsload
        if (_totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * _totalSupply) / freeFunds; // use "freeFunds" instead of "pool"
        }
        // ...
    }

    // saves on 1 Gcoldsload
    function _calculateLockedProfit() internal view returns (uint256) {
        uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;
        if (lockedFundsRatio < DEGRADATION_COEFFICIENT) {
            uint256 _lockedProfit = lockedProfit; // 1 Gcoldsload
            return _lockedProfit - ((lockedFundsRatio * _lockedProfit) / DEGRADATION_COEFFICIENT);
        }

        return 0;
    }

    // saves on 2 Gcoldsload
    function _reportLoss(address strategy, uint256 loss) internal {
        // ...
        uint256 _totalAllocBPS = totalAllocBPS; // saves on 2 Gcoldsload

        if (_totalAllocBPS != 0) {
            // reduce strat's allocBPS proportional to loss
            uint256 bpsChange = Math.min((loss * _totalAllocBPS) / totalAllocated, stratParams.allocBPS);

            // If the loss is too small, bpsChange will be 0
            if (bpsChange != 0) {
                stratParams.allocBPS -= bpsChange;
                totalAllocBPS = _totalAllocBPS - bpsChange;
            }
        }

        // ...
    }
```

### [G-6] Using return variable area to assign returned value

The return variable would always be there to be used to assign values which would be returned, so instead of defining a local variable which would be at the end assigned to the return variable. it could just use the return variable right away, saving 3 gas in the process, (it ain't much, but honest work :) ), which would also greatly improve on readability. It would save for 45 instances 135 gas.

45 instances - 8 files


- BorrowerOperations._triggerBorrowingFee()
- BorrowerOperations._getUSDValue()
- BorrowerOperations._updateTroveFromAdjustment()
- BorrowerOperations._getNewNominalICRFromTroveChange()
- BorrowerOperations._getNewICRFromTroveChange()
- BorrowerOperations._getNewTCRFromTroveChange()
- BorrowerOperations._getNewTroveAmounts()
- StabilityPool._computeLQTYPerUnitStaked()
- StabilityPool._getSingularCollateralGain()
- StabilityPool._getLQTYGainFromSnapshots()
- StabilityPool.getDepositorLQTYGain()
- StabilityPool.getCompoundedLUSDDeposit()
- StabilityPool._getCompoundedDepositFromSnapshots()
- PriceFeed._scaleChainlinkPriceByDigits()
- PriceFeed._scaleTellorPriceByDigits()
- PriceFeed._storeTellorPrice()
- PriceFeed._storeChainlinkPrice()
- SortedTroves._descendList()
- SortedTroves._ascendList()
- SortedTroves._findInsertPosition()
- TroveManager.getNominalICR()
- TroveManager.getCurrentICR()
- TroveManager._getCurrentTroveAmounts()
- TroveManager.getPendingCollateralReward()
- TroveManager.getPendingLUSDDebtReward()
- TroveManager._updateStakeAndTotalStakes()
- TroveManager._computeNewStake()
- TroveManager.updateBaseRateFromRedemption()
- TroveManager._calcRedemptionFee()
- TroveManager._calcDecayedBaseRate()
- TroveManager.increaseTroveColl()
- TroveManager.decreaseTroveColl()
- TroveManager.increaseTroveDebt()
- TroveManager.decreaseTroveDebt()
- SafeMath.add()
- SafeMath.sub()
- SafeMath.mul()
- SafeMath.div()
- SafeMath.mod()
- LQTYStaking.getPendingCollateralGain()
- LQTYStaking._getPendingLUSDGain()
- ReaperVaultV2._chargeFees()
- ReaperVaultV2.report()
- ReaperVaultV2._cascadingAccessRoles()
- ReaperBaseStrategyv4._cascadingAccessRoles()

### [G-7] Reordering if checks to save on extra check

in the case, we have a check which would exit the execution flow, it should prioritized to be checked, as long as it doesn't affect the business logic.

1 instance - 1 file

File: LiquityMath.sol

```solidity
    function _decPow(uint256 _base, uint256 _minutes)
        internal
        pure
        returns (uint256)
    {
        if (_minutes == 0) { // prioritizing an exit check
            return DECIMAL_PRECISION;
        }

        if (_minutes > 525600000) {
            _minutes = 525600000;
        } // cap to avoid overflow

        // ...
    }
```

### [G-8] Define variables outside of the for loop

By defining local variables outside the for loop, we get to reuse the same variable, instead of clearing and using new ones.

11 instances - 6 files

- ActivePool.setAddresses()
- CollateralConfig.initialize()
- PriceFeed.setAddresses()
- StabilityPool.provideToSP()
- StabilityPool.withdrawFromSP()
- StabilityPool._updateDepositAndSnapshots()
- StabilityPool._requireNoUnderCollateralizedTroves()
- TroveManager._getTotalsFromLiquidateTrovesSequence_RecoveryMode()
- LQTYStaking._getPendingCollateralGain()
- LQTYStaking._updateUserSnapshots()
- LQTYStaking._sendCollGainToUser()

### [G-9] Operations safe to be unchecked

There are some operations that are safe to be unchecked from overflow, so it would be good to uncheck those operations to save on extra gas.

| Operation | # Instances | Gas Saved / Instance | Total Gas Saved |
|:--:|:--:|:--:|:--:|
|[Sub]|17|54|918|
|[Inc]|1|63|63|
|Total|-|117|981|

library File: ReaperMathUtils (First we modify it to support unchecked substraction)

```solidity
library ReaperMathUtils {
    /**
     * @notice For doing an unchecked increment of an index for gas optimization purposes
     * @param _i - The number to increment
     * @return The incremented number
     */
    function uncheckedInc(uint256 _i) internal pure returns (uint256) {
        unchecked {
            return _i + 1;
        }
    }

    /**
     * @notice For doing an unchecked substraction of _b from _a for gas optimization purposes
     * @param _a - The first number
     * @param _b - The sceond number
     * @return The substraction result
     */
    function uncheckedSub(uint256 _a, uint256 _b) internal pure returns (uint256) {
        unchecked {
            return _a - _b;
        }
    }
}
```

18 instances - 4 files

1. File: ReaperBaseStrategyv4.sol

```solidity
    function harvest() external override returns (int256 roi) {
        // ...
        if (amountFreed < debt) {
            roi = -int256(debt.uncheckedSub(amountFreed));
        } else if (amountFreed > debt) {
            roi = int256(amountFreed.uncheckedSub(debt));
        }
        // ...
    }
```

2. File: ReaperStrategyGranarySupplyOnly.sol

```solidity
    function _adjustPosition(uint256 _debt) internal override {
        // ...
        uint256 toReinvest = wantBalance.uncheckedSub(_debt);
        // ...
    }

    function _liquidatePosition(uint256 _amountNeeded)
    internal
        override
        returns (uint256 liquidatedAmount, uint256 loss)
    {
        // ...
        _withdraw(_amountNeeded.uncheckedSub(wantBal));
        // ...
        loss = _amountNeeded.uncheckedSub(liquidatedAmount);
        // ...
    }

    function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {
        // ...
        uint256 profit = totalAssets.uncheckedSub(allocated);
        // ...
        roi = -int256(allocated.uncheckedSub(totalAssets));
        // ...
    }
```

3. File: ReaperVaultERC4626.sol

```solidity
    function maxDeposit(address) external view override returns (uint256 maxAssets) {
        if (emergencyShutdown || balance() >= tvlCap) return 0;
        if (tvlCap == type(uint256).max) return type(uint256).max;
        return tvlCap.uncheckedSub(balance());
    }

    function maxMint(address) external view override returns (uint256 maxShares) {
        if (emergencyShutdown || balance() >= tvlCap) return 0;
        if (tvlCap == type(uint256).max) return type(uint256).max;
        return convertToShares(tvlCap.uncheckedSub(balance()));
    }

    function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {
        require(y != 0, "Division by 0");

        uint256 q = x / y;
        if (x % y != 0) q.uncheckedInc();
        return q;
    }
```

4. File: ReaperVaultV2.sol

```solidity
    function availableCapital() public view returns (int256) {
        // ...
            return -int256(stratCurrentAllocation.uncheckedSub(stratMaxAllocation));
        // ...
            uint256 available = stratMaxAllocation.uncheckedSub(stratCurrentAllocation);
        // ...
            available = Math.min(available, vaultMaxAllocation.uncheckedSub(vaultCurrentAllocation));
        // ...
    }

    function _withdraw(
        uint256 _shares,
        address _receiver,
        address _owner
    ) internal nonReentrant returns (uint256 value) {
        // ...
        uint256 remaining = value.uncheckedSub(vaultBalance);
        // ...
    }

    function report(int256 _roi, uint256 _repayment) external returns (uint256) {
        // ...
        token.safeTransfer(vars.stratAddr, vars.credit.uncheckedSub(vars.freeWantInStrat));
        // ...
        token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat.uncheckedSub(vars.credit));
        // ...
        lockedProfit = vars.lockedProfitBeforeLoss.uncheckedSub(vars.loss);
        // ...
    }
```