### [N-01] ```require()``` should be used instead of ```assert()```


#### Impact
Prior to solidity version 0.8.0, hitting an assert consumes the remainder of the transaction’s available gas rather than returning it, as ```require()```/```revert()``` do. ```assert()``` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that “The assert function creates an error of type Panic(uint256). … Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix”.


#### Findings:
```
Ethos-Core/contracts/LUSDToken.sol:L312        assert(sender != address(0));

Ethos-Core/contracts/LUSDToken.sol:L313        assert(recipient != address(0));

Ethos-Core/contracts/LUSDToken.sol:L321        assert(account != address(0));

Ethos-Core/contracts/LUSDToken.sol:L329        assert(account != address(0));

Ethos-Core/contracts/LUSDToken.sol:L337        assert(owner != address(0));

Ethos-Core/contracts/LUSDToken.sol:L338        assert(spender != address(0));

Ethos-Core/contracts/TroveManager.sol:L417            assert(_LUSDInStabPool != 0);

Ethos-Core/contracts/TroveManager.sol:L1224            assert(totalStakesSnapshot[_collateral] > 0);

Ethos-Core/contracts/TroveManager.sol:L1279        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

Ethos-Core/contracts/TroveManager.sol:L1342        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

Ethos-Core/contracts/TroveManager.sol:L1348        assert(index <= idxLast);

Ethos-Core/contracts/TroveManager.sol:L1414        assert(newBaseRate > 0); // Base rate is always non-zero after redemption

Ethos-Core/contracts/TroveManager.sol:L1489        assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0;

Ethos-Core/contracts/BorrowerOperations.sol:L128        assert(MIN_NET_DEBT > 0);

Ethos-Core/contracts/BorrowerOperations.sol:L197        assert(vars.compositeDebt > 0);

Ethos-Core/contracts/BorrowerOperations.sol:L301        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

Ethos-Core/contracts/BorrowerOperations.sol:L331        assert(_collWithdrawal <= vars.coll); 

Ethos-Core/contracts/StabilityPool.sol:L526        assert(_debtToOffset <= _totalLUSDDeposits);

Ethos-Core/contracts/StabilityPool.sol:L551        assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

Ethos-Core/contracts/StabilityPool.sol:L591        assert(newP > 0);

```

### [N-02] Unused named returns


#### Impact
using both named returns and return statement is not neccessary, removing one of those can improve code clarity. ( see `@audit`)



#### Findings:
1.
```
TroveManager.sol:

 function _liquidateNormalMode(
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        address _borrower,
        uint _LUSDInStabPool
    )
        internal
        returns (LiquidationValues memory singleLiquidation) // @audit NC: unused named returns
    {
        <redacted>
        return singleLiquidation; // @audit NC: unused named returns
    }

```

2.
```
TroveManager.sol:

function _liquidateRecoveryMode(
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        address _borrower,
        uint _ICR,
        uint _LUSDInStabPool,
        uint _TCR,
        uint _price,
        uint256 _MCR
    )
        internal
        returns (LiquidationValues memory singleLiquidation) // @audit NC: unused named returns
    {
	<redacted>	
        return singleLiquidation; // @audit NC: unused named returns
    }

```
3.
```
TroveManager.sol:

    function _addLiquidationValuesToTotals(LiquidationTotals memory oldTotals, LiquidationValues memory singleLiquidation)
    internal pure returns(LiquidationTotals memory newTotals) {  // @audit NC: unused named returns

        <redacted>

        return newTotals;  // @audit NC: unused named returns
    }
```
4.
```
TroveManager.sol:

    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) { // @audit NC: unused named returns
        <redacted>

        return index; // @audit NC: unused named returns
    }
```
5.
```
StabilityPool.sol:


    function _computeRewardsPerUnitStaked(
        address _collateral,
        uint _collToAdd,
        uint _debtToOffset,
        uint _totalLUSDDeposits
    )
        internal
        returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked) // @audit NC: unused named returns
    {
        <redacted>

        return (collGainPerUnitStaked, LUSDLossPerUnitStaked); // @audit NC: unused named returns
    }

```

### [N-03] Missing events emission for setAddresses functions

#### Impact
It is noted that the function `setAddresses()` in `ActivePool.sol` is missing `TreasuryAddressChanged` & `LQTYStakingAddressChanged` events. ( see `@audit`)


#### Findings:
```
ActivePool.sol:

    function setAddresses(
        address _collateralConfigAddress,
        address _borrowerOperationsAddress,
        address _troveManagerAddress,
        address _stabilityPoolAddress,
        address _defaultPoolAddress,
        address _collSurplusPoolAddress,
        address _treasuryAddress,
        address _lqtyStakingAddress,
        address[] calldata _erc4626vaults
    )
        external
        onlyOwner
    {
        require(!addressesSet, "Can call setAddresses only once");

        checkContract(_collateralConfigAddress);
        checkContract(_borrowerOperationsAddress);
        checkContract(_troveManagerAddress);
        checkContract(_stabilityPoolAddress);
        checkContract(_defaultPoolAddress);
        checkContract(_collSurplusPoolAddress);
        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
        checkContract(_lqtyStakingAddress);

        collateralConfigAddress = _collateralConfigAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        troveManagerAddress = _troveManagerAddress;
        stabilityPoolAddress = _stabilityPoolAddress;
        defaultPoolAddress = _defaultPoolAddress;
        collSurplusPoolAddress = _collSurplusPoolAddress;
        treasuryAddress = _treasuryAddress;
        lqtyStakingAddress = _lqtyStakingAddress;

        address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();
        uint256 numCollaterals = collaterals.length;
        require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");
        for(uint256 i = 0; i < numCollaterals; i++) {
            address collateral = collaterals[i];
            address vault = _erc4626vaults[i];
            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");
            yieldGenerator[collateral] = vault;
        }

        emit CollateralConfigAddressChanged(_collateralConfigAddress);
        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
        emit TroveManagerAddressChanged(_troveManagerAddress);
        emit StabilityPoolAddressChanged(_stabilityPoolAddress);
        emit DefaultPoolAddressChanged(_defaultPoolAddress);
        emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);

	// @audit NC: missing emit TreasuryAddressChanged(_treasuryAddress);
	// @audit NC: missing emit LQTYStakingAddressChanged(_lqtyStakingAddress);

        addressesSet = true;
    }
```

### [L-01] Integer Overflow in `setYieldDistributionParams()` function in `ActivePool.sol`. 

#### Impact
The contract is using solidity version 0.6.11 which doesn't come with integer overflow checks by default, therefore the addition in require statement can go overflow and overflowed values can be set and might lead to unexpected results in other functions.

#### Findings:
```
    function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
        require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS"); // @audit: This equation can overflow 
        yieldSplitTreasury = _treasurySplit;
        yieldSplitSP = _SPSplit;
        yieldSplitStaking = _stakingSplit;
        emit YieldDistributionParamsUpdated(_treasurySplit, _SPSplit, _stakingSplit);
    }

```

#### Recommendations:

Use the SafeMath.sol `.add()` function.

