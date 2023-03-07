### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |

| Total Found Issues | 22 |
|:--:|:--:|

### Low Risk Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-1] | The function `updateDistributionPeriod` should not allow distributionPeriod to be set as zero. This problem leads to the wrong calculation of rewardPerSecond in the function `fund` duo to division by zero. As rewardPerSecond is zero, this will lead to the wrong calculation of issuance in the function `issueOath` and here the function will actually issue a wrong amount of Oath to the stability pool. | 1 |
| [L-2] | The function `setAddresses` in CommunityIssuance is one time callable and the given addresses can't be changed after that, as there is no other function to change the address of the `stabilityPool` on a later plan. | 1 |


| Total Low Risk Issues | 2 |
|:--:|:--:|

### Non-Critical Template
| Count | Explanation | Instances |
|:----:|:-------|:--:|
| [N-1] | Both of the functions `transfer` and `transferFrom` does two address(0) checks on the recipient | 1 |
| [N-2] | Imported console in `LUSDToken.sol` | 1 |
| [N-3] | State variable `mintingPaused` is applied with its default value | 2 |
| [N-4] | A simple check can be applied, instead of creating a whole function doing its purpose | 2 |
| [N-5] | The function `sendOath` should check, if the contract has the balance of Oath tokens to do the transfer | 1 |
| [N-6] | The executed code in the function `setAddresses` can be moved in the constructor, as once called the function can't be called a second time | 1 |
| [N-7] | TODO applied in the contracts | 2 |
| [N-8] | Constructor lacks address(0) check | 2 |


| Total Non-Critical Issues | 8 |
|:--:|:--:|

### Refactor Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-1] | Use require instead of assert | 20 |
| [R-2] | `_requireMintingNotPaused` can be used in `pauseMinting`, so the function can be called only when the minting isn't already paused | 1 |
| [R-3] | Use delete to clear variables instead of zero assignment | 11 |
| [R-4] | The function `_requireUserHasDeposit` isn't needed as its used only one time, and the check can be applied in the core function | 1 |
| [R-5] | Some brackets on if statements can be removed, as the code will still work as intended | 9 |
| [R-6] | Numeric values having to do with time should use time units for readability | 1 |
| [R-7] | Some number values can be refactored with _ | 1 |

| Total Refactor Issues | 7 |
|:--:|:--:|

### Ordinary Issues Template
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-1] | Outdated Compiler Version | 12 |
| [O-2] | Events is missing indexed fields | 9 |
| [O-3] | Use a more recent pragma version | 12 |
| [O-4] | Hardcoded values can't be changed | 4 |
| [O-5] | Create your own import names instead of using the regular ones | 12 |


| Total Ordinary Issues | 5 |
|:--:|:--:|

### [L-1] The function `updateDistributionPeriod` should not allow distributionPeriod to be set as zero. This problem leads to the wrong calculation of rewardPerSecond in the function `fund` duo to division by zero. As rewardPerSecond is zero, this will lead to the wrong calculation of issuance in the function `issueOath` and here the function will actually issue a wrong amount of Oath to the stability pool.
The function `updateDistributionPeriod` is used only by the owner in order to change the distributionPeriod. The problem here is that the function allows the distributionPeriod to be changed to zero, which leads to issues occurring for the calculation of `rewardPerSecond` and `issuance`. The purpose of the function `fund` is to fund the contract and update the distribution, the problem here arises if the distributionPeriod is set as zero, duo to that any amount send by the owner will result to zero rewardPerSecond, as in the calculation the amount is divided by distributionPeriod (which is zero). As rewardPerSecond is zero, this leads to another problem in the core function `issueOath`, which is called by the pool in order to issue a set amount of Oath to the stability pool. As rewardPerSecond is zero, the calculation of issuance will equal zero as well duo to the mul of rewardPerSecond. Duo to that the function will return zero to the stability pool, even tho there was funded amount of tokens to the contract by the owner. 

Recommended - The function `updateDistributionPeriod` should not be allowed to set the distributionPeriod to zero, as this leads to issues occurring and a problem regarding issuing amount of Oath tokens to the stability pool. Consider adding a simple check in the function `require(_newDistributionPeriod != 0, "Zero distribution not allowed")`.

```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

120:  function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121:        distributionPeriod = _newDistributionPeriod;
122:    }
```
```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

101:  function fund(uint amount) external onlyOwner {
102:        require(amount != 0, "cannot fund 0");
103:        OathToken.transferFrom(msg.sender, address(this), amount);
104:
105:        // roll any unissued OATH into new distribution
106:        if (lastIssuanceTimestamp < lastDistributionTime) {
107:            uint timeLeft = lastDistributionTime.sub(lastIssuanceTimestamp);
108:            uint notIssued = timeLeft.mul(rewardPerSecond);
109:            amount = amount.add(notIssued);
110:        }
111:
112:        rewardPerSecond = amount.div(distributionPeriod);
113:        lastDistributionTime = block.timestamp.add(distributionPeriod);
114:        lastIssuanceTimestamp = block.timestamp;
115:
116:        emit LogRewardPerSecond(rewardPerSecond);
117:    }
```
```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

84:  function issueOath() external override returns (uint issuance) {
85:        _requireCallerIsStabilityPool();
86:        if (lastIssuanceTimestamp < lastDistributionTime) {
87:            uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
88:            uint256 timePassed = endTimestamp.sub(lastIssuanceTimestamp);
89:            issuance = timePassed.mul(rewardPerSecond);
90:            totalOATHIssued = totalOATHIssued.add(issuance);
91:        }
92:
93:        lastIssuanceTimestamp = block.timestamp;
94:        emit TotalOATHIssuedUpdated(totalOATHIssued);
95:    }
```

### [L-2] The function setAddresses in CommunityIssuance is one time callable and the given addresses can't be changed after that, as there is no other function to change the address of the stabilityPool on a later plan.
As how it is right now, the function `setAddresses` in CommunityIssuance is one time callable, and can't be called after that in order to change the addresses again. This might be problematic for the stabilityPoolAddress, if on later plan the address need to be changed, as there is no other function for changing the stability pool address. Consider creating a function which is callable only by the owner and can change the address of the stability pool. As how it is right now a mistake isn't allowed with the function setAddresses.

```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61:  function setAddresses
62:    (
63:        address _oathTokenAddress, 
64:        address _stabilityPoolAddress
65:    ) 
66:        external 
67:        onlyOwner 
68:        override 
69:    {
70:        require(!initialized, "issuance has been initialized");
71:        checkContract(_oathTokenAddress);
72:        checkContract(_stabilityPoolAddress);
73:
74:        OathToken = IERC20(_oathTokenAddress);
75:        stabilityPoolAddress = _stabilityPoolAddress;
76:
77:        initialized = true;
78:
79:        emit OathTokenAddressSet(_oathTokenAddress);
80:        emit StabilityPoolAddressSet(_stabilityPoolAddress);
81:    }
```


### [N-1] Both of the functions `transfer` and `transferFrom` does two address(0) checks on the recipient
When the function `transferFrom` is executed it does not one but two unnecessary checks to see if the recipient is address(0).
First the internal function `_requireValidRecipient` is called in both of the functions, apart from doing other things the function checks if the receipt's address is address(0). After that the internal function `_transfer` is called which does the same check for the receipts. As `_requireValidRecipient` is called only by this two functions, the address(0) check in the function isn't needed and can be removed.

```solidity
Ethos-Core/contracts/LUSDToken.sol

235:  function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {
236:        _requireValidRecipient(recipient);
237:        _transfer(sender, recipient, amount);
238:        _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));
239:        return true;
240:    }

346:  function _requireValidRecipient(address _recipient) internal view {
347:        require(
348:            _recipient != address(0) && 
349:            _recipient != address(this),
350:            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
351:        );
352:        require(
353:            !stabilityPools[_recipient] &&
354:            !troveManagers[_recipient] &&
355:            !borrowerOperations[_recipient],
356:            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
357:        );
358:    }

311:  function _transfer(address sender, address recipient, uint256 amount) internal {
312:        assert(sender != address(0));
313:        assert(recipient != address(0));
314:
315:        _balances[sender] = _balances[sender].sub(amount, "ERC20: transfer amount exceeds balance");
316:        _balances[recipient] = _balances[recipient].add(amount);
317:        emit Transfer(sender, recipient, amount);
318:    }
```

### [N-2] Imported console in `LUSDToken.sol`
An imported console dependencies is left imported in the LUSDToken contract.

```solidity
Ethos-Core/contracts/LUSDToken.sol

9: import "./Dependencies/console.sol";
```

### [N-3] State variable `mintingPaused` is applied with its default value
The state variable mintingPaused is applied with its default value in LUSDToken, the false here isn't needed and can be remove as it will still do the same thing.

```solidity
Ethos-Core/contracts/LUSDToken.sol

37: bool public mintingPaused = false;

Ethos-Core/contracts/ActivePool.sol

32: bool public addressesSet = false;
```


### [N-4] A simple check can be applied, instead of creating a whole function doing its purpose
There are two functions created in LQTYStaking, First `_requireUserHasStake` is used, which simply checks if the given amount is bigger than zero, same goes to the function `_requireNonZeroAmount` which does the same thing. Instead of creating whole functions doing simple checks, this checks can be applied in the core functions.

```solidity
Ethos-Core/contracts/LQTY/LQTYStaking.sol

261: function _requireCallerIsBorrowerOperations() internal view {
262:        require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
263:    }

265:  function _requireUserHasStake(uint currentStake) internal pure {  
266:        require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');  
267:    }
```

### [N-5] The function `sendOath` should check, if the contract has the balance of Oath tokens to do the transfer
The function `sendOath` is called by the stability pool in order to transfer a given amount of Oath tokens to an account.
A good feature here would be to add a require statement to check if the contract has the amount of tokens needed for the transfer. This way the user will understand the reason for the revert.

```solidity
Ethos-Core/contracts/LQTY/CommunityIssuance.sol

124:  function sendOath(address _account, uint _OathAmount) external override {
125:        _requireCallerIsStabilityPool();
126:
127:        OathToken.transfer(_account, _OathAmount);
129:    }
```

### [N-6] The executed code in the function `setAddresses` can be moved in the constructor, as once called the function can't be called a second time
One time callable function is made `setAddresses`, which sets all the needed addresses and can't be called a second. 
Instead of doing all of that, the code can moved to the constructor and executed at a deploying time.

```solidity
Ethos-Core/contracts/TroveManager.sol

function setAddresses(
        address _borrowerOperationsAddress,
        address _collateralConfigAddress,
        address _activePoolAddress,
        address _defaultPoolAddress,
        address _stabilityPoolAddress,
        address _gasPoolAddress,
        address _collSurplusPoolAddress,
        address _priceFeedAddress,
        address _lusdTokenAddress,
        address _sortedTrovesAddress,
        address _lqtyTokenAddress,
        address _lqtyStakingAddress,
        address _redemptionHelperAddress
    )
        external
        override
    {
        require(msg.sender == owner);

        checkContract(_borrowerOperationsAddress);
        checkContract(_collateralConfigAddress);
        checkContract(_activePoolAddress);
        checkContract(_defaultPoolAddress);
        checkContract(_stabilityPoolAddress);
        checkContract(_gasPoolAddress);
        checkContract(_collSurplusPoolAddress);
        checkContract(_priceFeedAddress);
        checkContract(_lusdTokenAddress);
        checkContract(_sortedTrovesAddress);
        checkContract(_lqtyTokenAddress);
        checkContract(_lqtyStakingAddress);
        checkContract(_redemptionHelperAddress);

        borrowerOperationsAddress = _borrowerOperationsAddress;
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPool = IStabilityPool(_stabilityPoolAddress);
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lqtyToken = IERC20(_lqtyTokenAddress);
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
        redemptionHelper = IRedemptionHelper(_redemptionHelperAddress);

        emit BorrowerOperationsAddressChanged(_borrowerOperationsAddress);
        emit CollateralConfigAddressChanged(_collateralConfigAddress);
        emit ActivePoolAddressChanged(_activePoolAddress);
        emit DefaultPoolAddressChanged(_defaultPoolAddress);
        emit StabilityPoolAddressChanged(_stabilityPoolAddress);
        emit GasPoolAddressChanged(_gasPoolAddress);
        emit CollSurplusPoolAddressChanged(_collSurplusPoolAddress);
        emit PriceFeedAddressChanged(_priceFeedAddress);
        emit LUSDTokenAddressChanged(_lusdTokenAddress);
        emit SortedTrovesAddressChanged(_sortedTrovesAddress);
        emit LQTYTokenAddressChanged(_lqtyTokenAddress);
        emit LQTYStakingAddressChanged(_lqtyStakingAddress);
        emit RedemptionHelperAddressChanged(_redemptionHelperAddress);

        owner = address(0);
    }
```

### [N-7] TODO applied in the contracts
There are TODO comments in the contracts.

Instances:
```solidity
Ethos-Core/contracts/StabilityPool.sol

335: /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
380: /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
```

### [N-8] Constructor lacks address(0) check
Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

Instances:
```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

120: token = IERC20Metadata(_token);
124: treasury = _treasury;
```

### [R-1] Use require instead of assert
The Solidity assert() function is meant to assert invariants.
Properly functioning code should never reach a failing assert statement.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol

128: assert(MIN_NET_DEBT > 0);
197: assert(vars.compositeDebt > 0);
301: assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331: assert(_collWithdrawal <= vars.coll); 
417: assert(_LUSDInStabPool != 0);
1224: assert(totalStakesSnapshot[_collateral] > 0);
1279: assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348: assert(index <= idxLast);
1414: assert(newBaseRate > 0);
1489: assert(decayedBaseRate <= DECIMAL_PRECISION);

Ethos-Core/contracts/StabilityPool.sol

526: assert(_debtToOffset <= _totalLUSDDeposits);
551: assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591: assert(newP > 0);

Ethos-Core/contracts/LUSDToken.sol

312: assert(sender != address(0));
313: assert(recipient != address(0));
321: assert(account != address(0));
329: assert(account != address(0));
337: assert(owner != address(0));
338: assert(spender != address(0));
```
Recommended: Consider whether the condition checked in the assert() is actually an invariant.
If not, replace the assert() statement with a require() statement.

### [R-2] `_requireMintingNotPaused` can be used in `pauseMinting`, so the function can be called only when the minting isn't already paused
In general the function `pauseMinting` is used by the guardian or governance to temporary pause the minting of the token.
A good feature would be to add a call to the function `_requireMintingNotPaused` in `pauseMinting`. So it can be called only when the minting isn't already paused.

```solidity
Ethos-Core/contracts/LUSDToken.sol

133: function pauseMinting() external {
134:        require(
135:            msg.sender == guardianAddress || msg.sender == governanceAddress,
136:            "LUSD: Caller is not guardian or governance"
137:        );
138:        mintingPaused = true;
139:    }
```

The above instance can be refactored to:

```solidity
function pauseMinting() external {
        _requireMintingNotPaused();
        require(
            msg.sender == guardianAddress || msg.sender == governanceAddress,
            "LUSD: Caller is not guardian or governance"
        );
        mintingPaused = true;
    }
```


### [R-3] Use delete to clear variables instead of zero assignment
You can use the delete keyword instead of setting the variable as zero.

Instances:
```solidity
Ethos-Core/contracts/TroveManager.sol

387: singleLiquidation.debtToOffset = 0;
388: singleLiquidation.collToSendToSP = 0;
468: debtToOffset = 0;
469: collToSendToSP = 0;
1192: Troves[_borrower][_collateral].stake = 0;
1285: Troves[_borrower][_collateral].coll = 0;
1286: Troves[_borrower][_collateral].debt = 0;
1288: rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289: rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;

Ethos-Core/contracts/ActivePool.sol

253:  vars.profit = 0;

Ethos-Core/contracts/StabilityPool.sol

529: lastLUSDLossError_Offset = 0;
578: currentScale = 0;
```

### [R-4] The function `_requireUserHasDeposit` isn't needed as its used only one time, and the check can be applied in the core function 
The function `_requireUserHasDeposit` is internal pure function, which is called only in `withdrawFromSP`. 
The purpose of the function is simple, it just check if the given amount is greater than zero. Instead of doing all of that, we can remove the function `_requireUserHasDeposit` and just apply a simple check in the core function `withdrawFromSP`.

```solidity
Ethos-Core/contracts/StabilityPool.sol

869:  function _requireUserHasDeposit(uint _initialDeposit) internal pure {
870:        require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
871:    }
```


### [R-5] Some brackets on if statements can be removed, as the code will still work as intended
Brackets are applied on if statements executing only one thing, the code will work as intended even if the brackets are removed.

```solidity
Ethos-Core/contracts/TroveManager.sol

1238: if (_debt == 0) { return; }
```

The above instance can be refactored to:

```solidity
1238: if (_debt == 0)  return; 
```

Other instances:
```solidity
Ethos-Core/contracts/StabilityPool.sol

428: if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}
469: if (totalLUSD == 0 || _debtToOffset == 0) { return; }
489: if (totalLUSD == 0) { return; }
720: if (initialDeposit == 0) { return 0; }
742: if (epochSnapshot < currentEpoch) { return 0; }
768: if (compoundedDeposit < initialDeposit.div(1e9)) {return 0;}
784: if (_amount == 0) {return;}
795: if (LUSDWithdrawal == 0) {return;}
```

### [R-6] Numeric values having to do with time should use time units for readability
Suffixes like seconds, minutes, hours, days and weeks after literal numbers can be used to specify units of time where seconds are the base unit and units are considered naively in the following way:

`1 == 1 seconds`
`1 minutes == 60 seconds`
`1 hours == 60 minutes`
`1 days == 24 hours`
`1 weeks == 7 days`

```solidity
Ethos-Core/contracts/TroveManager.sol

53: uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
```

### [R-7] Some number values can be refactored with _ 
Consider using underscores for number values to improve readability.

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol

41: uint256 public constant PERCENT_DIVISOR = 10000;
```

The above instance can be refactored to:

```solidity
41: uint256 public constant PERCENT_DIVISOR = 10_000;
```

### [O-1] Outdated Compiler Version
Using an outdated compiler version can be problematic especially if there are publicly disclosed bugs and issues that affect the current compiler version. It is recommended to use a recent version of the Solidity compiler.

Instances - All contracts

### [O-2] Events is missing indexed fields
Index event fields make the field more quickly accessible to off-chain.
Each event should use three indexed fields if there are three or more fields.

Instances:
```solidity
Ethos-Core/contracts/CollateralConfig.sol

37: event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
38: event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);

Ethos-Core/contracts/TroveManager.sol

200: event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);
206: event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);
207: event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
208: event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
209: event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);

Ethos-Core/contracts/LQTY/LQTYStaking.sol

61: event CollateralSent(address _account, address _collateral, uint _amount);
62: event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);
```

### [O-3] Use a more recent pragma version 
Old version of solidity is used, consider using the new one `0.8.18`.
You can see what new versions offer regarding bug fixed [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

### [O-4] Hardcoded values can't be changed
Ones set hardcoded values can't be changed later, consider this as a reminder.

```solidity
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24: address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25: ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26:     ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27: IAaveProtocolDataProvider public constant DATA_PROVIDER =
28:     IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29: IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
```

### [O-5] Create your own import names instead of using the regular ones
For better readability, you should name the imports instead of using the regular ones.

Instances:
```solidity
Ethos-Core/contracts/CollateralConfig.sol
Ethos-Core/contracts/BorrowerOperations.sol
Ethos-Core/contracts/TroveManager.sol
Ethos-Core/contracts/ActivePool.sol
Ethos-Core/contracts/StabilityPool.sol
Ethos-Core/contracts/LQTY/CommunityIssuance.sol
Ethos-Core/contracts/LQTY/LQTYStaking.sol
Ethos-Core/contracts/LUSDToken.sol
Ethos-Vault/contracts/ReaperVaultV2.sol
Ethos-Vault/contracts/ReaperVaultERC4626.sol
Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
```