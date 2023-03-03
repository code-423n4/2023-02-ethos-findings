- setAddresses function for the variables that won't change, they need to be set in the constructor as immutable variables to reduce their gas consumption. (e.g: LUSD address...)

contract instances: ActivePool, BorrowerOperations, CollSurplusPool, DefaultPool, HintHelpers, PriceFeed, RedemptionHelper, StabilityPool, TroveManager

- LUSDToken can be further optimized, inspired by solmate ERC20 implementation, there are savings that can be used while not compromising on security:

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

- in StabilityPool._updateRewardSumAndProduct , you may subtract without the need to use SafeMath.sub, since it's already has been checked for underflow by the assert check.

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
        // -> you may substracrt right away, since it's already validated in the assert check
        uint256 newProductFactor = DECIMAL_PRECISION - _LUSDLossPerUnitStaked;
``` 
- cache reused storage variable.
instance:
file: TroveManage.sol
```solidity
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
        );
        Troves[_borrower][_collateral].stake = newStake;
        totalStakes[_collateral] = newTotalStakes;
        // tbd gas assigned to local variable
        emit TotalStakesUpdated(_collateral, newTotalStakes);

        return newStake;
    }
```
- use return variable area to assign returned values instead of caching them on a separate variable:

instances:

BorrowerOperations._triggerBorrowingFee()
BorrowerOperations._getUSDValue()
BorrowerOperations._getNewNominalICRFromTroveChange()
BorrowerOperations._getNewICRFromTroveChange()
BorrowerOperations._getNewTCRFromTroveChange()
StabilityPool._computeLQTYPerUnitStaked()
StabilityPool._getSingularCollateralGain()
StabilityPool._getLQTYGainFromSnapshots()
... many instances to mention :)

- assigning final constant values, in LiquityMath.decMul(), DECIMAL_PRECISION / 2 will always be equal to 5e+17, so it's best to assign the final value, instead of being each time unnecessarily computed.

instances:
LiquityMath.decMul() // using 5e17 instead of 1e18
LiquityMath._computeNominalCR() // using type(uint256).max instead of 2**256 -1
LiquityMath._computeCR() // using type(uint256).max instead of 2**256 -1

- move the if check that ends the execution to be first, in LiquityMath._decPow():
```solidity
    function _decPow(uint256 _base, uint256 _minutes)
        internal
        pure
        returns (uint256)
    {
        if (_minutes == 0) {
            return DECIMAL_PRECISION;
        }

        if (_minutes > 525600000) {
            _minutes = 525600000;
        } // cap to avoid overflow
        // ...
    }
```
