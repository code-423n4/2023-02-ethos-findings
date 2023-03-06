|  | Issue |  Saved Gas for each call| 
| --- | --- | --- | 
| [G-01] | Function call pointing to an external contract not needed | 110.000 |
| [G-02] | Function call not needed | 100 |
| [G-03] | Optimize nesting of **if statements** | 55 | 
| [G-04] | Removing of **if statement** | 3500 |
| [G-05] | Assignment of same storage variable to two local variables | 180 |
| [G-06] | Use local variable instead of storage pointer | 160 |
| [G-07] | Returned value not used | 95 |
| [G-08] | Cache external contracts calls in parent function | 1000 |
| [G-09] | Internal function called by only one parent function | 10 |


# [G-01] Function call pointing to an external contract not needed 
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L105
When calling function `_withdrawUnderlying(balanceOfPool())` there's no need to call `balanceOfPool()` as parameter because `_withdrawUnderlying` already withdraw the minimum between the required `_withdrawAmount` and `balanceOfPool()`.
Therefore is enough to pass type(uint256).max as argument in order to save one external call.
   
    function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
        _withdrawUnderlying(balanceOfPool());
        return balanceOfWant();
    }

    function _withdrawUnderlying(uint256 _withdrawAmount) internal {
        uint256 withdrawable = balanceOfPool();
        _withdrawAmount = MathUpgradeable.min(_withdrawAmount, withdrawable);
        ILendingPool(ADDRESSES_PROVIDER.getLendingPool()).withdraw(address(want), _withdrawAmount, address(this));
    }

    function balanceOfPool() public view returns (uint256) {
        (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
            address(want),
            address(this)
        );
        return supply;
    }

By changing from `_withdrawUnderlying(balanceOfPool());` to `_withdrawUnderlying(type(uint256).max);` it will save around **110.000gas**.
Tested with Tenderly forking optimism chain and calling of  IAaveProtocolDataProvider(0x9546f673ef71ff666ae66d01fd6e7c6dae5a9995).getUserReserveData(0x68f180fcce6836688e9084f035309e29bf0a2095, 0x7abef6a3e7b4d5e7e33584bbb7cb60ecbe0f0581)


# [G-02] Function call not needed 
Ethos-Vault/contracts/ReaperVaultERC4626.sol#L184
In the following function there's no need to check if `totalSupply() == 0` because if it is 0 and `_freeFunds()` is != 0 it will return 0 anyway.

    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return 0;
        shares = roundUpDiv(assets * totalSupply(), _freeFunds());
    }

Please consider to change as follow:

    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
        if (_freeFunds() == 0) return 0;
        shares = roundUpDiv(assets * totalSupply(), _freeFunds());
    }

> This change could save around **100gas** each call.

# [G-03] Optimize if statements nesting
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L99
In function `_liquidatePosition` check at L99 ` if (_amountNeeded > liquidatedAmount)` can never be satisfied by the else statement part because `liquidatedAmount = _amountNeeded;`.
Therefore it's suggested to move the `if (_amountNeeded > liquidatedAmount) {loss = _amountNeeded - liquidatedAmount;}` inside the first if statement to save this if check when it's not needed.
Therefore the code should be changed  as follows:
ACTUAL IMPLEMENTATION

    function _liquidatePosition(uint256 _amountNeeded)
        internal
        override
        returns (uint256 liquidatedAmount, uint256 loss)
    {
        uint256 wantBal = balanceOfWant();
        if (wantBal < _amountNeeded) {
            _withdraw(_amountNeeded - wantBal);
            liquidatedAmount = balanceOfWant();
        } else {
            liquidatedAmount = _amountNeeded;
        }

        if (_amountNeeded > liquidatedAmount) {
            loss = _amountNeeded - liquidatedAmount;
        }
    }

NEW IMPLEMENTATION

    function _liquidatePosition(uint256 _amountNeeded)
        internal
        override
        returns (uint256 liquidatedAmount, uint256 loss)
    {
        uint256 wantBal = balanceOfWant();
        if (wantBal < _amountNeeded) {
            _withdraw(_amountNeeded - wantBal);
            liquidatedAmount = balanceOfWant();
            if (_amountNeeded > liquidatedAmount) {loss = _amountNeeded - liquidatedAmount;}
        } else {
            liquidatedAmount = _amountNeeded;
        }
    }

This change saves around **55gas** when not entering in the if statement. 


# [G-04] Removing of **if statement**
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L120-L124
The following function snippet regarding the case of emegencyExit  can be optimized by saving the last if statement.
Last statement is `if (roi<0)` which can happen only when `if (amountFreed < debt)` is **TRUE**. 
Moreover `repayment -= uint256(-roi);` is equivalent to `repayment = debt - uint256(-roi);` which in turn is equivalent to `repayment = debt -(debt-amountFreed);`
ORIGINAL SNIPPET

        uint256 repayment = 0;
        if (emergencyExit) {
            uint256 amountFreed = _liquidateAllPositions();
            if (amountFreed < debt) {
                roi = -int256(debt - amountFreed); //roi will be always < 0
            } else if (amountFreed > debt) {
                roi = int256(amountFreed - debt); //roi will be always < 0
            }
            repayment = debt;  
            if (roi < 0) {
                repayment -= uint256(-roi);
            }

NEW SNIPPET

        uint256 repayment = 0;
        if (emergencyExit) {
            uint256 amountFreed = _liquidateAllPositions();
            if (amountFreed < debt) {
                roi = -int256(debt - amountFreed);
                repayment = amountFreed;
            } else if (amountFreed > debt) {
                roi = int256(amountFreed - debt);
                repayment = debt;
            }

The above change will save around **3500gas** 

# [G-05] Assignment of same storage variable to two local variables 
Ethos-Core/contracts/ActivePool.sol#263 267 and 272
Inside function _rebalance the same storage variable is assigned to two temporary variables.
At line 243: `vars.currentAllocated = yieldingAmount[_collateral];`
At line 263: `vars.yieldingAmount = yieldingAmount[_collateral];`
Last instance where `vars.currentAllocated` is used is before `vars.yieldingAmount` therefore there's no need to use 2 different memory variables in the same function.
Therefore it can be used memory variable from L243 `vars.currentAllocated = yieldingAmount[_collateral];` on every occurence 

By removing the call of storage variable yieldingPercentage[_collateral] again: `vars.yieldingPercentage = yieldingPercentage[_collateral];` the gas saving is around **50gas** each call

Moreover the code included in lines 267 and 272 is not needed because they calculate an amount equal to `vars.finalYieldingAmount`
This is because `vars.yieldingPercentage == vars.currentAllocated`. 
By analyzing check line 267: `vars.yieldingAmount = vars.yieldingAmount.sub(vars.toWithdraw);` 
which is equivalent to `vars.currentAllocated.sub(vars.currentAllocated.sub(vars.finalYieldingAmount))` 
which corresponds to `vars.finalYieldingAmount` therefore `yieldingAmount[_collateral] = vars.finalYieldingAmount` 
Therefore the snippet from #L264 to #L274 can be changed as follows:

        if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) { 
            // we will end up overallocated, withdraw some
            vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);
            yieldingAmount[_collateral] = vars.finalYieldingAmount; 
        } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
            // we will end up underallocated, deposit more
            vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);
            yieldingAmount[_collateral] = vars.finalYieldingAmount;
        }
This additional change can save **130gas** each call

# [G-06] Use local variable instead of storage pointer
Ethos-Vault/contracts/ReaperVaultV2.sol#L451
It can be used the cached allocated value, Instead of the following `stratParams.allocated -= loss;`
Substitute with: `stratParams.allocated = allocation - loss;`

This change can save **160gas** each call

# [G-07] Returned value not used
Ethos-Vault/contracts/ReaperVaultV2.sol#363
_withdraw function `returns (uint256 value)` but there's no function using the returned value.
Therefore by removing `returns (uint256 value)` from function header and moving local variable declaration `uint256 value` inside function body it saves aroung **95gas** each call.

# [G-08] Cache external contracts calls in parent function
Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203 and 225
In the above line both functions `__getPendingCollateralGain(address _user) internal view` and `_updateUserSnapshots(address _user) internal` make an address cache by doing an external contract call: `assets = collateralConfig.getAllowedCollaterals();` 
These two functions are called toghether in two occurunces, it is suggested to cache the addess in parent functions and pass the `address assets` as parameter.

Here below an example of change in function unstake Ethos-Core/contracts/LQTY/LQTYStaking.sol#L142

    function unstake(uint _LQTYamount) external override {
        ....
        address[] memory assets = collateralConfig.getAllowedCollaterals();
        uint[] memory collGainAmounts = _getPendingCollateralGain(msg.sender, assets); 
        ...
        _updateUserSnapshots(msg.sender, assets);

Where 

    function _getPendingCollateralGain(address _user, address[] memory assets) internal view returns (uint[] memory amounts)
    function _updateUserSnapshots(address _user, address[] memory assets) internal

By doing so it will save around **1.000gas** per call.

# [G-09] Internal function called by only one parent function
Ethos-Core/contracts/LQTY/LQTYStaking.sol#L144
Function `_requireUserHasStake(currentStake);` is called by function `unstake()` only.
By removing this function call and moving checking at beginning of `unstake()`  will save around **10gas** per call.
The function should look like: 

     function unstake(uint _LQTYamount) external override {
        uint currentStake = stakes[msg.sender];
        require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake'); 
        ...

Same issue is at #L269 with function `_requireNonZeroAmount` called by `stake` function only.

# [G-10] Checking of toFree > 0 before calling  _liquidatePosition
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L136
It is suggestedested to use an if statement to check `if(toFree >0)` before calling  `_liquidatePosition(toFree);` because it will return `loss == 0` and `amountFreed == 0`
This would save a function call. Therefore it should be written as:
        
        uint256 amountFreed;
        uint256 loss; = 
        if(toFree > 0 ) {
            (amountFreed, loss) = _liquidatePosition(toFree);
        }
        repayment = MathUpgradeable.min(_debt, amountFreed);
        roi -= int256(loss); 

# [G-11] Check amount > before trasferring tokens
Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171,172, 135, 136
Suggested to check `if (LUSDGain > 0)` and  `if collGainAmounts > 0` before calling transfer tokens functions.


# [G-12] depositAll() in ReaperVaultV2 is useless
Ethos-Vault/contracts/ReaperVaultV2.sol#L302
Function `_deposit` can be called by DEPOSITOR only, because of `_atLeastRole(DEPOSITOR);` check.
ActivePool has the role of DEPOSITOR and doesn't implement `depositAll()` function call.

    function depositAll() external {
        _deposit(token.balanceOf(msg.sender), msg.sender);
    }
Remove `depositAll()` if there's not plan to be used to save gas upon deployment 

