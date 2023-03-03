- in LQTYStaking.stake() and LQTYStaking.unstake(), the use of safeTransferFrom and safeTransfer is unnecessary, LQTY is compliant with ERC20, so it's safe to use transferFrom and transfer, which would always behave as expected and return a valid boolean on each operation.

- you may use in LQTYStaking.stake() sendToLQTYStaking instead of transferFrom, since it can only be accessed by staking contract and it would save on extra gas to set the approval amount and emitting approval event.

- in ReaperVaultV2, reconstruct mapping struct to the needed storage space to save on Gsset which would save on 3 storage slots:

```
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
- in PriceFeed, combine priceAggregator and status mappings to save on extra Gsset. on the following 4 mappings in total to improve on the readability:
```
    struct PriceFeedParams {
        bytes32 tellorQueryId; // 256 bits
        uint256 lastGoodPrice; // 256 bits
        address aggregator; // 160 bits
        Status status; // 8 bits
    }
    mapping (address => PriceFeedParams) public priceFeed;
```
- in CollateralConfig, we can save on 3 Gsset for collateralConfig mapping:
```
    struct Config {
        uint96 MCR;
        uint96 CCR;
        uint8 decimals;
        bool allowed;
    }
```
- in LUSDToken, we can save on 2 Gsset for 3 mappings:
```
    struct Roles {
        bool isTroveManager;
        bool isStabilityPool;
        bool isBorrowerOperation;
    }
    mapping (address => Roles) public addresses;
```
- setAddresses function for the variables that won't change, they can be set in the constructor as immutable variables to reduce their gas consumption. (e.g: LUSD address...)

contract instances: ActivePool, BorrowerOperations, CollSurplusPool, DefaultPool, HintHelpers, PriceFeed, RedemptionHelper, StabilityPool, TroveManager.

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
