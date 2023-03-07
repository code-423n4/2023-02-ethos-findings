
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 26 |
| [NC-2](#NC-2) |  `require()` / `revert()` statements should have descriptive reason strings | 10 |
| [NC-3](#NC-3) | Return values of `approve()` not checked | 5 |
| [NC-4](#NC-4) | TODO Left in the code | 2 |
| [NC-5](#NC-5) | Event is missing `indexed` fields | 91 |
| [NC-6](#NC-6) | Functions not used internally could be marked external | 2 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (26)*:
```solidity
File: example/ActivePool.sol

96:         collateralConfigAddress = _collateralConfigAddress;

97:         borrowerOperationsAddress = _borrowerOperationsAddress;

98:         troveManagerAddress = _troveManagerAddress;

99:         stabilityPoolAddress = _stabilityPoolAddress;

100:         defaultPoolAddress = _defaultPoolAddress;

101:         collSurplusPoolAddress = _collSurplusPoolAddress;

103:         lqtyStakingAddress = _lqtyStakingAddress;

```

```solidity
File: example/BorrowerOperations.sol

146:         stabilityPoolAddress = _stabilityPoolAddress;

147:         gasPoolAddress = _gasPoolAddress;

152:         lqtyStakingAddress = _lqtyStakingAddress;

```

```solidity
File: example/LQTY/CommunityIssuance.sol

75:         stabilityPoolAddress = _stabilityPoolAddress;

```

```solidity
File: example/LQTY/LQTYStaking.sol

88:         troveManagerAddress = _troveManagerAddress;

89:         borrowerOperationsAddress = _borrowerOperationsAddress;

90:         activePoolAddress = _activePoolAddress;

```

```solidity
File: example/LUSDToken.sol

102:         troveManagerAddress = _troveManagerAddress;

106:         stabilityPoolAddress = _stabilityPoolAddress;

110:         borrowerOperationsAddress = _borrowerOperationsAddress;

114:         governanceAddress = _governanceAddress;

117:         guardianAddress = _guardianAddress;

149:         governanceAddress = _newGovernanceAddress;

156:         guardianAddress = _newGuardianAddress;

170:         troveManagerAddress = _newTroveManagerAddress;

174:         stabilityPoolAddress = _newStabilityPoolAddress;

178:         borrowerOperationsAddress = _newBorrowerOperationsAddress;

```

```solidity
File: example/TroveManager.sol

266:         borrowerOperationsAddress = _borrowerOperationsAddress;

271:         gasPoolAddress = _gasPoolAddress;

```

### <a name="NC-2"></a>[NC-2]  `require()` / `revert()` statements should have descriptive reason strings

*Instances (10)*:
```solidity
File: example/TroveManager.sol

250:         require(msg.sender == owner);

551:         require(totals.totalDebtInSequence > 0);

716:         require(_troveArray.length != 0);

754:         require(totals.totalDebtInSequence > 0);

1450:         require(redemptionFee < _collateralDrawn);

1523:         require(msg.sender == borrowerOperationsAddress);

1527:         require(msg.sender == address(redemptionHelper));

1531:         require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));

1535:         require(Troves[_borrower][_collateral].status == Status.active);

1539:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```

### <a name="NC-3"></a>[NC-3] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (5)*:
```solidity
File: example/LUSDToken.sol

231:         _approve(msg.sender, spender, amount);

238:         _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));

243:         _approve(msg.sender, spender, _allowances[msg.sender][spender].add(addedValue));

248:         _approve(msg.sender, spender, _allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));

289:         _approve(owner, spender, amount);

```

### <a name="NC-4"></a>[NC-4] TODO Left in the code
TODOs may signal that a feature is missing or not ready for audit, consider resolving the issue and removing the TODO comment

*Instances (2)*:
```solidity
File: example/StabilityPool.sol

335:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.

380:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.

```

### <a name="NC-5"></a>[NC-5] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (91)*:
```solidity
File: example/ActivePool.sol

58:     event CollateralConfigAddressChanged(address _newCollateralConfigAddress);

59:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

60:     event TroveManagerAddressChanged(address _newTroveManagerAddress);

61:     event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);

62:     event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);

63:     event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);

64:     event YieldingPercentageUpdated(address _collateral, uint256 _bps);

65:     event YieldingPercentageDriftUpdated(uint256 _driftBps);

66:     event YieldClaimThresholdUpdated(address _collateral, uint256 _threshold);

67:     event YieldDistributionParamsUpdated(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit);

```

```solidity
File: example/BorrowerOperations.sol

92:     event CollateralConfigAddressChanged(address _newCollateralConfigAddress);

93:     event TroveManagerAddressChanged(address _newTroveManagerAddress);

94:     event ActivePoolAddressChanged(address _activePoolAddress);

95:     event DefaultPoolAddressChanged(address _defaultPoolAddress);

96:     event StabilityPoolAddressChanged(address _stabilityPoolAddress);

97:     event GasPoolAddressChanged(address _gasPoolAddress);

98:     event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);

99:     event PriceFeedAddressChanged(address  _newPriceFeedAddress);

100:     event SortedTrovesAddressChanged(address _sortedTrovesAddress);

101:     event LUSDTokenAddressChanged(address _lusdTokenAddress);

102:     event LQTYStakingAddressChanged(address _lqtyStakingAddress);

104:     event TroveCreated(address indexed _borrower, address _collateral, uint arrayIndex);

105:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint stake, BorrowerOperation operation);

106:     event LUSDBorrowingFeePaid(address indexed _borrower, address _collateral, uint _LUSDFee);

```

```solidity
File: example/LQTY/CommunityIssuance.sol

50:     event OathTokenAddressSet(address _oathTokenAddress);

51:     event LogRewardPerSecond(uint _rewardPerSecond);

52:     event StabilityPoolAddressSet(address _stabilityPoolAddress);

53:     event TotalOATHIssuedUpdated(uint _totalOATHIssued);

```

```solidity
File: example/LQTY/LQTYStaking.sol

49:     event LQTYTokenAddressSet(address _lqtyTokenAddress);

50:     event LUSDTokenAddressSet(address _lusdTokenAddress);

51:     event TroveManagerAddressSet(address _troveManager);

52:     event BorrowerOperationsAddressSet(address _borrowerOperationsAddress);

53:     event ActivePoolAddressSet(address _activePoolAddress);

54:     event CollateralConfigAddressSet(address _collateralConfigAddress);

56:     event StakeChanged(address indexed staker, uint newStake);

57:     event StakingGainsWithdrawn(address indexed staker, uint LUSDGain, address[] _assets, uint[] _amounts);

58:     event F_CollateralUpdated(address _collateral, uint _F_Collateral);

59:     event F_LUSDUpdated(uint _F_LUSD);

60:     event TotalLQTYStakedUpdated(uint _totalLQTYStaked);

61:     event CollateralSent(address _account, address _collateral, uint _amount);

62:     event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);

```

```solidity
File: example/LUSDToken.sol

78:     event TroveManagerAddressChanged(address _troveManagerAddress);

79:     event StabilityPoolAddressChanged(address _newStabilityPoolAddress);

80:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

81:     event GovernanceAddressChanged(address _governanceAddress);

82:     event GuardianAddressChanged(address _guardianAddress);

```

```solidity
File: example/StabilityPool.sol

233:     event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);

234:     event StabilityPoolLUSDBalanceUpdated(uint _newBalance);

236:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

237:     event CollateralConfigAddressChanged(address _newCollateralConfigAddress);

238:     event TroveManagerAddressChanged(address _newTroveManagerAddress);

239:     event ActivePoolAddressChanged(address _newActivePoolAddress);

240:     event DefaultPoolAddressChanged(address _newDefaultPoolAddress);

241:     event LUSDTokenAddressChanged(address _newLUSDTokenAddress);

242:     event SortedTrovesAddressChanged(address _newSortedTrovesAddress);

243:     event PriceFeedAddressChanged(address _newPriceFeedAddress);

244:     event CommunityIssuanceAddressChanged(address _newCommunityIssuanceAddress);

246:     event P_Updated(uint _P);

247:     event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);

248:     event G_Updated(uint _G, uint128 _epoch, uint128 _scale);

249:     event EpochUpdated(uint128 _currentEpoch);

250:     event ScaleUpdated(uint128 _currentScale);

252:     event DepositSnapshotUpdated(address indexed _depositor, uint _P, address[] _assets, uint[] _amounts, uint _G);

253:     event UserDepositChanged(address indexed _depositor, uint _newDeposit);

255:     event CollateralGainWithdrawn(address indexed _depositor, address _collateral, uint _collAmount);

256:     event LQTYPaidToDepositor(address indexed _depositor, uint _LQTY);

257:     event CollateralSent(address _collateral, address _to, uint _amount);

```

```solidity
File: example/TroveManager.sol

186:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

187:     event CollateralConfigAddressChanged(address _newCollateralConfigAddress);

188:     event PriceFeedAddressChanged(address _newPriceFeedAddress);

189:     event LUSDTokenAddressChanged(address _newLUSDTokenAddress);

190:     event ActivePoolAddressChanged(address _activePoolAddress);

191:     event DefaultPoolAddressChanged(address _defaultPoolAddress);

192:     event StabilityPoolAddressChanged(address _stabilityPoolAddress);

193:     event GasPoolAddressChanged(address _gasPoolAddress);

194:     event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);

195:     event SortedTrovesAddressChanged(address _sortedTrovesAddress);

196:     event LQTYTokenAddressChanged(address _lqtyTokenAddress);

197:     event LQTYStakingAddressChanged(address _lqtyStakingAddress);

198:     event RedemptionHelperAddressChanged(address _redemptionHelperAddress);

200:     event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);

201:     event TroveUpdated(address indexed _borrower, address _collateral, uint _debt, uint _coll, uint _stake, TroveManagerOperation _operation);

202:     event TroveLiquidated(address indexed _borrower, address _collateral, uint _debt, uint _coll, TroveManagerOperation _operation);

203:     event BaseRateUpdated(uint _baseRate);

204:     event LastFeeOpTimeUpdated(uint _lastFeeOpTime);

205:     event TotalStakesUpdated(address _collateral, uint _newTotalStakes);

206:     event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);

207:     event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

208:     event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);

209:     event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);

210:     event Redemption(

```

### <a name="NC-6"></a>[NC-6] Functions not used internally could be marked external

*Instances (2)*:
```solidity
File: example/TroveManager.sol

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Empty Function Body - Consider commenting why | 1 |
| [L-3](#L-3) | Initializers could be front-run | 3 |
| [L-4](#L-4) | Unsafe ERC20 operation(s) | 4 |
### <a name="L-1"></a>[L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: example/LUSDToken.sol

283:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 

```

### <a name="L-2"></a>[L-2] Empty Function Body - Consider commenting why

*Instances (1)*:
```solidity
File: example/ReaperVaultERC4626.sol

24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

```

### <a name="L-3"></a>[L-3] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (3)*:
```solidity
File: example/ReaperStrategyGranarySupplyOnly.sol

62:     function initialize(

67:     ) public initializer {

70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);

```

### <a name="L-4"></a>[L-4] Unsafe ERC20 operation(s)

*Instances (4)*:
```solidity
File: example/LQTY/CommunityIssuance.sol

103:         OathToken.transferFrom(msg.sender, address(this), amount);

127:         OathToken.transfer(_account, _OathAmount);

```

```solidity
File: example/LQTY/LQTYStaking.sol

135:             lusdToken.transfer(msg.sender, LUSDGain);

171:         lusdToken.transfer(msg.sender, LUSDGain);

```