### QA Report
[Exclusive findings](#exclusive-findings) is below the [automated findings](#automated-findings).

---

## Automated Findings
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Missing checks for `address(0)` when assigning values to address state variables | 26 |
| [NC-2](#NC-2) |  `require()` / `revert()` statements should have descriptive reason strings | 10 |
| [NC-3](#NC-3) | Return values of `approve()` not checked | 5 |
| [NC-4](#NC-4) | TODO Left in the code | 2 |
| [NC-5](#NC-5) | Event is missing `indexed` fields | 93 |
| [NC-6](#NC-6) | Functions not used internally could be marked external | 2 |
| [NC-7](#NC-7) | Typos | 32 |
### <a name="NC-1"></a>[NC-1] Missing checks for `address(0)` when assigning values to address state variables

*Instances (26)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

96:         collateralConfigAddress = _collateralConfigAddress;

97:         borrowerOperationsAddress = _borrowerOperationsAddress;

98:         troveManagerAddress = _troveManagerAddress;

99:         stabilityPoolAddress = _stabilityPoolAddress;

100:         defaultPoolAddress = _defaultPoolAddress;

101:         collSurplusPoolAddress = _collSurplusPoolAddress;

103:         lqtyStakingAddress = _lqtyStakingAddress;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

146:         stabilityPoolAddress = _stabilityPoolAddress;

147:         gasPoolAddress = _gasPoolAddress;

152:         lqtyStakingAddress = _lqtyStakingAddress;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

75:         stabilityPoolAddress = _stabilityPoolAddress;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

88:         troveManagerAddress = _troveManagerAddress;

89:         borrowerOperationsAddress = _borrowerOperationsAddress;

90:         activePoolAddress = _activePoolAddress;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

266:         borrowerOperationsAddress = _borrowerOperationsAddress;

271:         gasPoolAddress = _gasPoolAddress;

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="NC-2"></a>[NC-2]  `require()` / `revert()` statements should have descriptive reason strings

*Instances (10)*:
```solidity
File: Ethos-Core/contracts/TroveManager.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="NC-3"></a>[NC-3] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (5)*:
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

231:         _approve(msg.sender, spender, amount);

238:         _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));

243:         _approve(msg.sender, spender, _allowances[msg.sender][spender].add(addedValue));

248:         _approve(msg.sender, spender, _allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));

289:         _approve(owner, spender, amount);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

### <a name="NC-4"></a>[NC-4] TODO Left in the code
TODOs may signal that a feature is missing or not ready for audit, consider resolving the issue and removing the TODO comment

*Instances (2)*:
```solidity
File: Ethos-Core/contracts/StabilityPool.sol

335:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.

380:         /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

### <a name="NC-5"></a>[NC-5] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (93)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

37:     event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);

38:     event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

50:     event OathTokenAddressSet(address _oathTokenAddress);

51:     event LogRewardPerSecond(uint _rewardPerSecond);

52:     event StabilityPoolAddressSet(address _stabilityPoolAddress);

53:     event TotalOATHIssuedUpdated(uint _totalOATHIssued);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

78:     event TroveManagerAddressChanged(address _troveManagerAddress);

79:     event StabilityPoolAddressChanged(address _newStabilityPoolAddress);

80:     event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);

81:     event GovernanceAddressChanged(address _governanceAddress);

82:     event GuardianAddressChanged(address _guardianAddress);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

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
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="NC-6"></a>[NC-6] Functions not used internally could be marked external

*Instances (2)*:
```solidity
File: Ethos-Core/contracts/TroveManager.sol

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

### <a name="NC-7"></a>[NC-7] Typos

*Instances (32)*:
```diff
File: Ethos-Core/contracts/ActivePool.sol

- 49:     uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
+ 49:     uint256 public yieldingPercentageDrift = 100; // rebalance if % is off by more than 100 BPS

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```diff
File: Ethos-Core/contracts/BorrowerOperations.sol

- 127:         // This makes impossible to open a trove with zero withdrawn LUSD
+ 127:         // This makes it impossible to open a trove with zero withdrawn LUSD

- 230:         // Pull collateral, move it to the Active Pool, and mint the LUSDAmount to the borrower
+ 230:         // Pull collateral, move it to the Active Pool, and mint the LUSD amount to the borrower

- 241:     // Send more collateral to an existing trove
+ 241:     // Send more collateral to an existing Trove

- 253:     // Withdraw collateral from a trove
+ 253:     // Withdraw collateral from a Trove

- 258:     // Withdraw LUSD tokens from a trove: mint new LUSD tokens to the owner, and increase the trove's debt accordingly
+ 258:     // Withdraw LUSD tokens from a Trove: mint new LUSD tokens to the owner, and increase the Trove's debt accordingly

- 305:         // Get the collChange based on whether or not collateral was sent in the transaction
+ 305:         // Get the collChange based on whether or not collateral was sent in the transaction (i.e. if the Stability Pool is sending collateral to a Trove)

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```diff
File: Ethos-Core/contracts/CollateralConfig.sol

- 23:     // Smallest allowed value for Critical system collateral ratio.
+ 23:     // Smallest allowed value for the critical system collateral ratio.

- 34:     address[] public collaterals; // for returning entire list of allowed collaterals
+ 34:     address[] public collaterals; // for returning the entire list of allowed collaterals.

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```diff
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

- 140:     // Unstake the LQTY and send the it back to the caller, along with their accumulated LUSD & collateral gains.
+ 140:     // Unstake the LQTY and send it back to the caller, along with their accumulated LUSD & collateral gains

- 146:         // Grab any accumulated ETH and LUSD gains from the current stake
+ 146:         // Grab any accumulated collateral and LUSD gains from the current stake

- 170:         // Send accumulated LUSD and ETH gains to the caller
+ 170:         // Send accumulated LUSD and collateral gains to the caller

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```diff
File: Ethos-Core/contracts/LUSDToken.sol

- 73:     // Copied from LQTYToken.sol; since we deleted that file, we use LUSDToken's initialization
+ 73:     // Copied from LQTYToken.sol; since we deleted that file, we use LQTYToken's initialization

- 183:     // --- Functions for intra-Liquity calls ---
+ 183:     // --- Functions for intra-Liquity calls --- // TODO: remove this comment

- 308:     // --- Internal operations ---
+ 308:     // Warning: sanity checks (for sender and recipient) should have been done before calling these internal functions

- 309:     // Warning: sanity checks (for sender and recipient) should have been done before calling these internal functions
+ 309:     // --- 'require' functions ---

- 344:     // --- 'require' functions ---
+ 344:         // only latest borrowerOps version can mint

- 361:         // only latest borrowerOps version can mint
+ 361:         // old versions of the protocol may still burn

- 366:         // old versions of the protocol may still burn
+ 366:         // only latest stabilityPool can accept new deposits

- 376:         // only latest stabilityPool can accept new deposits
+ 376:         // old versions of the protocol may still:

- 381:         // old versions of the protocol may still:
+ 381:         // 1. send lusd gas reserve to liquidator

- 382:         // 1. send lusd gas reserve to liquidator
+ 382:         // 2. be able to return users their lusd from the stability pool

- 383:         // 2. be able to return users their lusd from the stability pool
+ 383:     // --- Optional functions ---

397:     // --- Optional functions ---

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

```diff
File: Ethos-Core/contracts/StabilityPool.sol

- 728:     // Internal function, used to calculcate compounded deposits.
+ 728:     // Internal function, used to calculate compounded deposits.

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```diff
File: Ethos-Core/contracts/TroveManager.sol

- 41:     // A doubly linked list of Troves, sorted by their sorted by their collateral ratios
+ 41:     // A doubly linked list of Troves, sorted by their collateral ratios

- 114:     // Array of all active trove addresses - used to to compute an approximate hint off-chain, for the sorted list insertion
+ 114:     // Array of all active trove addresses - used to compute an approximate hint off-chain, for the sorted list insertion

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

```diff
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

- 1: // SPDX-License-Identifier: BUSL1.1
+ 1: // SPDX-License-Identifier: BSL-1.0

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

```diff
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

- 224:     // Allows an on-chain or off-chain user to simulate the effects of their redeemption at the current block,
+ 224:     // Allows an on-chain or off-chain user to simulate the effects of their redemption at the current block,

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

```diff
File: Ethos-Vault/contracts/ReaperVaultV2.sol

- 32:         uint256 lastReport; // block.timestamp of the last time a report occured
+ 32:         uint256 lastReport; // block.timestamp of the last time a report occurred

- 434:         // Loss can only be up the amount of capital allocated to the strategy
+ 434:         // Loss can only be up to the amount of capital allocated to the strategy

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```diff
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

- 24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF
+ 24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RFP

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Do not use deprecated library functions | 1 |
| [L-3](#L-3) | Empty Function Body - Consider commenting why | 2 |
| [L-4](#L-4) | Initializers could be front-run | 8 |
| [L-5](#L-5) | Unsafe ERC20 operation(s) | 4 |
### <a name="L-1"></a>[L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

283:         bytes32 digest = keccak256(abi.encodePacked('\x19\x01', 

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)

### <a name="L-2"></a>[L-2] Do not use deprecated library functions

*Instances (1)*:
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

74:         IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="L-3"></a>[L-3] Empty Function Body - Consider commenting why

*Instances (2)*:
```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

24:     ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

61:     constructor() initializer {}

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="L-4"></a>[L-4] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (8)*:
```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:     function initialize(

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:     function initialize(

67:     ) public initializer {

70:         __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

61:     constructor() initializer {}

63:     function __ReaperBaseStrategy_init(

69:         __UUPSUpgradeable_init();

70:         __AccessControlEnumerable_init();

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)

### <a name="L-5"></a>[L-5] Unsafe ERC20 operation(s)

*Instances (4)*:
```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

103:         OathToken.transferFrom(msg.sender, address(this), amount);

127:         OathToken.transfer(_account, _OathAmount);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

135:             lusdToken.transfer(msg.sender, LUSDGain);

171:         lusdToken.transfer(msg.sender, LUSDGain);

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 21 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (21)*:
```solidity
File: Ethos-Core/contracts/ActivePool.sol

26: contract ActivePool is Ownable, CheckContract, IActivePool {

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

18: contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {

50:     ) external override onlyOwner {

89:     ) external onlyOwner checkCollateral(_collateral) {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

14: contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

67:         onlyOwner 

101:     function fund(uint amount) external onlyOwner {

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

18: contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {

76:         onlyOwner 

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

146: contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)

```solidity
File: Ethos-Core/contracts/TroveManager.sol

18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

15:     ReaperAccessControl,

```
[Link to code](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol)


---
## Exclusive Findings
| Issue | Count |
 | --- | --- |
|[Low Risk Issues](#low-risk-issues) |3|
|[Non Critical Issues](#non-critical-issues-1) |12|
|Total |15|
---
### Low Risk Issues

| No. | Issue |
| --- | --- |
| 1 | [Loss of precision due to rounding](#L-01-loss-of-precision-due-to-rounding)|
| 2 | [Always use `safeTransferFrom` instead of `transferFrom`](#L-02-always-use-safetransferfrom-instead-of-transferfrom)|
| 3 | [Owner can renounce Ownership](#L-03-owner-can-renounce-ownership)|
---
### [L-01] Loss of precision due to rounding

---
**Description:**

Typically occurs while using `/` operator. It is recommended to use `SafeMath` library for all arithmetic operations.

**Recommendation:**

Add scalar so roundings are negligible.

**Lines of Code:** 

- [TroveManager.sol#L54](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L54)
- [TroveManager.sol#L55](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L55)
- [ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)
- [ReaperVaultV2.sol#L155](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)
- [ReaperVaultV2.sol#L181](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181)

---
### [L-02] Always use `safeTransferFrom` instead of `transferFrom`

---
**Lines of Code:** 

- [CommunityIssuance.sol#L103](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103)

---
### [L-03] Owner can renounce Ownership

---
**Description:**

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The non-fungible Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

**Lines of Code:** 

- [ActivePool.sol#L26](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L26)
- [BorrowerOperations.sol#L18](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L18)
- [CollateralConfig.sol#L15](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L15)
- [CommunityIssuance.sol#L14](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14)
- [LQTYStaking.sol#L18](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L18)
- [StabilityPool.sol#L146](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L146)

---
### [L-04] onlyOwner functions

---
**`onlyOwner` functions:** 

- [ActivePool.sol#L83](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L83)
- [ActivePool.sol#L125](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L125)
- [ActivePool.sol#L132](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L132)
- [ActivePool.sol#L138](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L138)
- [ActivePool.sol#L144](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L144)
- [ActivePool.sol#L214](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L214)
- [BorrowerOperations.sol#L125](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L125)
- [CollateralConfig.sol#L50](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L50)
- [CollateralConfig.sol#L89](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L89)
- [CommunityIssuance.sol#L67](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L67)
- [CommunityIssuance.sol#L101](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101)
- [CommunityIssuance.sol#L120](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120)
- [LQTYStaking.sol#L76](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L76)
- [StabilityPool.sol#L273](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L273)



---
### Non Critical Issues

| No. | Issue |
| --- | --- |
| 1 | [For modern and more readable code; update import usages](#NC-01-for-modern-and-more-readable-code-update-import-usages)|
| 2 | [Use `require` instead of `assert`](#NC-02-use-require-instead-of-assert)|
| 3 | [NatSpec comments should be increased in contracts](#NC-03-natspec-comments-should-be-increased-in-contracts)|
| 4 | [Tokens accidentally sent to the contract cannot be recovered](#NC-04-tokens-accidentally-sent-to-the-contract-cannot-be-recovered)|
| 5 | [Use a more recent version of Solidity](#NC-05-use-a-more-recent-version-of-solidity)|
| 6 | [Use underscores for number literals](#NC-06-use-underscores-for-number-literals)|
| 7 | [Long lines are not suitable for the ‘Solidity Style Guide’](#NC-07-long-lines-are-not-suitable-for-the-solidity-style-guide)|
| 8 | [Omissions in Events](#NC-08-omissions-in-events)|
| 9 | [Showing the actual values of numbers in NatSpec comments makes checking and reading code easier](#NC-09-showing-the-actual-values-of-numbers-in-natspec-comments-makes-checking-and-reading-code-easier)|
| 10 | [Constant values such as a call to keccak256(), should used to immutable rather than constant](#nc-10-constant-values-such-as-a-call-to-keccak256-should-used-to-immutable-rather-than-constant) |
| 11 | [Return parameters are declared, hence no need return variable.](#nc-11-return-parameters-are-declared-hence-no-need-return-variable) |
| 12 | [Utilise named return variable](#nc-12-utilise-named-return-variable) |
---
### [NC-01] For modern and more readable code; update import usages

---
**Description:**

Our Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

**Lines of Code:** 

- [ActivePool.sol#L3-L5](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L3-L5)
- [ActivePool.sol#L6-L7](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L6-L7)
- [StabilityPool.sol#L3-L5](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L3-L5)
- [StabilityPool.sol#L6-L7](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L6-L7)
- [StabilityPool.sol#L7-L8](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L7-L8)
- [StabilityPool.sol#L8-L9](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L8-L9)
- [StabilityPool.sol#L9-L10](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L9-L10)
- [StabilityPool.sol#L10-L11](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L10-L11)

---
### [NC-02] Use `require` instead of `assert`

---
**Description:**

Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process's available gas instead of returning it, as request()/revert() did.

Assertion() should be avoided even after solidity version 0.8.0, because its documentation states 'The Assert function generates an error of type Panic(uint256). Code that works properly should never Panic, even on invalid external input. If this happens, you need to fix it in your contract. there's a mistake'.



**Lines of Code:** 

- [BorrowerOperations.sol#L128](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L128)
- [BorrowerOperations.sol#L197](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L197)
- [BorrowerOperations.sol#L301](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L301)
- [BorrowerOperations.sol#L331](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L331)
- [LUSDToken.sol#L312](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L312)
- [LUSDToken.sol#L313](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L313)
- [LUSDToken.sol#L321](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L321)
- [LUSDToken.sol#L329](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L329)
- [LUSDToken.sol#L337](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L337)
- [LUSDToken.sol#L338](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L338)
- [StabilityPool.sol#L526](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L526)
- [StabilityPool.sol#L551](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L551)
- [StabilityPool.sol#L591](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L591)
- [TroveManager.sol#L417](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L417)
- [TroveManager.sol#L1224](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1224)
- [TroveManager.sol#L1279](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1279)
- [TroveManager.sol#L1342](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1342)
- [TroveManager.sol#L1348](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1348)
- [TroveManager.sol#L1414](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1414)
- [TroveManager.sol#L1489](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1489)

---
### [NC-03] NatSpec comments should be increased in contracts

---
**Description:**

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Lines of Code:** 

- [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)
- [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)
- [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)
- [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
- [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
- [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)
- [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)
- [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)
- [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
- [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
- [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

---
### [NC-04] Tokens accidentally sent to the contract cannot be recovered

---
**Description:**

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

**Recommendation:**

Add this code:
```solidity
/**
	* @notice Sends ERC20 tokens trapped in contract to external address
	* @dev Onlyowner is allowed to make this function call
	* @param account is the receiving address
	* @param externalToken is the token being sent
	* @param amount is the quantity being sent
	* @return boolean value indicating whether the operation succeeded.
	*
*/
	function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
		IERC20(externalToken).transfer(account, amount);
		return true;
	}
}
```


---
### [NC-05] Use a more recent version of Solidity

---
**Description:**

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions: https://github.com/ethereum/solidity/blob/develop/Changelog.md

**Recommendation:**

Old version of Solidity is used , newer version can be used (0.8.17)

**Lines of Code:** 

- [ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol)
- [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol)
- [CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol)
- [CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)
- [LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)
- [LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol)
- [StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol)
- [TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol)
- [ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)
- [ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)
- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol)
- [ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

---
### [NC-06] Use underscores for number literals

---
**Description:**

For example, `1000` should be written as `1_000`

**Lines of Code:** 

- [ActivePool.sol#L145](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L145)
- [TroveManager.sol#L53](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L53)
- [TroveManager.sol#L54](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L54)
- [ReaperVaultV2.sol#L41](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41)

---
### [NC-07] Long lines are not suitable for the ‘Solidity Style Guide’

---
**Description:**

It is generally recommended that lines in the source code should not exceed 80-120 characters. Today's screens are much larger, so in some cases it makes sense to expand that. The lines above should be split when they reach that length, as the files will most likely be on GitHub and GitHub always uses a scrollbar when the length is more than 164 characters.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]



**Recommendation:**

Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction



**Lines of Code:** 

- [ActivePool.sol#L47](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L47)
- [ActivePool.sol#L144](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L144)
- [ActivePool.sol#L258](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L258)
- [ActivePool.sol#L264](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L264)
- [ActivePool.sol#L269](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L269)
- [ActivePool.sol#L282](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L282)
- [ActivePool.sol#L288](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L288)
- [BorrowerOperations.sol#L105](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L105)
- [BorrowerOperations.sol#L172](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L172)
- [BorrowerOperations.sol#L190](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L190)
- [BorrowerOperations.sol#L233](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L233)
- [BorrowerOperations.sol#L235](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L235)
- [BorrowerOperations.sol#L237](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L237)
- [BorrowerOperations.sol#L248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L248)
- [BorrowerOperations.sol#L254](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L254)
- [BorrowerOperations.sol#L259](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L259)
- [BorrowerOperations.sol#L264](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L264)
- [BorrowerOperations.sol#L268](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)
- [BorrowerOperations.sol#L272](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L272)
- [BorrowerOperations.sol#L282](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L282)
- [BorrowerOperations.sol#L312](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L312)
- [BorrowerOperations.sol#L343](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)
- [BorrowerOperations.sol#L358](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L358)
- [BorrowerOperations.sol#L419](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L419)
- [BorrowerOperations.sol#L511](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L511)
- [BorrowerOperations.sol#L517](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L517)
- [BorrowerOperations.sol#L528](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L528)
- [BorrowerOperations.sol#L529](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L529)
- [BorrowerOperations.sol#L530](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L530)
- [BorrowerOperations.sol#L538](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L538)
- [BorrowerOperations.sol#L546](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L546)
- [BorrowerOperations.sol#L637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L637)
- [BorrowerOperations.sol#L644](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L644)
- [BorrowerOperations.sol#L645](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L645)
- [BorrowerOperations.sol#L675](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L675)
- [BorrowerOperations.sol#L697](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L697)
- [LQTYStaking.sol#L203](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203)
- [LUSDToken.sol#L238](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L238)
- [LUSDToken.sol#L248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L248)
- [StabilityPool.sol#L474](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L474)
- [StabilityPool.sol#L547](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L547)
- [StabilityPool.sol#L626](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L626)
- [StabilityPool.sol#L637](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L637)
- [StabilityPool.sol#L657](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L657)
- [StabilityPool.sol#L671](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L671)
- [StabilityPool.sol#L673](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L673)
- [TroveManager.sol#L200](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L200)
- [TroveManager.sol#L201](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L201)
- [TroveManager.sol#L202](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L202)
- [TroveManager.sol#L338](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L338)
- [TroveManager.sol#L348](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L348)
- [TroveManager.sol#L351](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L351)
- [TroveManager.sol#L384](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L384)
- [TroveManager.sol#L393](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L393)
- [TroveManager.sol#L398](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L398)
- [TroveManager.sol#L404](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L404)
- [TroveManager.sol#L407](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L407)
- [TroveManager.sol#L416](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L416)
- [TroveManager.sol#L421](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L421)
- [TroveManager.sol#L428](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L428)
- [TroveManager.sol#L571](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L571)
- [TroveManager.sol#L572](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L572)
- [TroveManager.sol#L575](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L575)
- [TroveManager.sol#L618](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L618)
- [TroveManager.sol#L774](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L774)
- [TroveManager.sol#L775](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L775)
- [TroveManager.sol#L778](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L778)
- [TroveManager.sol#L819](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L819)
- [TroveManager.sol#L854](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L854)
- [TroveManager.sol#L887](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L887)
- [TroveManager.sol#L898](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L898)
- [TroveManager.sol#L902](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L902)
- [TroveManager.sol#L903](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L903)
- [TroveManager.sol#L915](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L915)
- [TroveManager.sol#L926](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L926)
- [TroveManager.sol#L957](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L957)
- [TroveManager.sol#L1082](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1082)
- [TroveManager.sol#L1097](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1097)
- [TroveManager.sol#L1258](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1258)
- [TroveManager.sol#L1259](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1259)
- [TroveManager.sol#L1305](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1305)
- [TroveManager.sol#L1312](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1312)
- [TroveManager.sol#L1323](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1323)
- [TroveManager.sol#L1567](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1567)
- [TroveManager.sol#L1574](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1574)
- [TroveManager.sol#L1581](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1581)
- [TroveManager.sol#L1588](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1588)
- [ReaperVaultERC4626.sol#L207](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L207)

---
### [NC-08] Omissions in Events

---
**Description:**

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

**Lines of Code:** 

- [ActivePool.sol#L58](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L58)
- [ActivePool.sol#L59](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L59)
- [ActivePool.sol#L60](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L60)
- [ActivePool.sol#L61](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L61)
- [ActivePool.sol#L62](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L62)
- [ActivePool.sol#L63](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L63)
- [ActivePool.sol#L64](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L64)
- [ActivePool.sol#L65](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L65)
- [ActivePool.sol#L66](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L66)
- [ActivePool.sol#L67](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L67)
- [BorrowerOperations.sol#L92](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L92)
- [BorrowerOperations.sol#L93](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L93)
- [BorrowerOperations.sol#L94](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L94)
- [BorrowerOperations.sol#L95](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L95)
- [BorrowerOperations.sol#L96](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L96)
- [BorrowerOperations.sol#L97](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L97)
- [BorrowerOperations.sol#L98](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L98)
- [BorrowerOperations.sol#L99](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L99)
- [BorrowerOperations.sol#L100](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L100)
- [BorrowerOperations.sol#L101](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L101)
- [BorrowerOperations.sol#L102](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L102)
- [BorrowerOperations.sol#L105](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L105)
- [CollateralConfig.sol#L38](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L38)
- [CommunityIssuance.sol#L53](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L53)
- [LQTYStaking.sol#L56](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L56)
- [LQTYStaking.sol#L58](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L58)
- [LQTYStaking.sol#L59](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L59)
- [LQTYStaking.sol#L60](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L60)
- [LQTYStaking.sol#L62](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L62)
- [LUSDToken.sol#L78](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L78)
- [LUSDToken.sol#L79](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L79)
- [LUSDToken.sol#L80](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L80)
- [LUSDToken.sol#L81](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L81)
- [LUSDToken.sol#L82](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L82)
- [StabilityPool.sol#L233](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L233)
- [StabilityPool.sol#L234](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L234)
- [StabilityPool.sol#L236](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L236)
- [StabilityPool.sol#L237](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L237)
- [StabilityPool.sol#L238](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L238)
- [StabilityPool.sol#L239](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L239)
- [StabilityPool.sol#L240](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L240)
- [StabilityPool.sol#L241](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L241)
- [StabilityPool.sol#L242](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L242)
- [StabilityPool.sol#L243](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L243)
- [StabilityPool.sol#L244](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L244)
- [StabilityPool.sol#L246](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L246)
- [StabilityPool.sol#L247](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L247)
- [StabilityPool.sol#L248](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L248)
- [StabilityPool.sol#L249](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L249)
- [StabilityPool.sol#L250](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L250)
- [StabilityPool.sol#L252](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L252)
- [StabilityPool.sol#L253](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L253)
- [TroveManager.sol#L186](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L186)
- [TroveManager.sol#L187](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L187)
- [TroveManager.sol#L188](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L188)
- [TroveManager.sol#L189](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L189)
- [TroveManager.sol#L190](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L190)
- [TroveManager.sol#L191](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L191)
- [TroveManager.sol#L192](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L192)
- [TroveManager.sol#L193](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L193)
- [TroveManager.sol#L194](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L194)
- [TroveManager.sol#L195](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L195)
- [TroveManager.sol#L196](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L196)
- [TroveManager.sol#L197](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L197)
- [TroveManager.sol#L198](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L198)
- [TroveManager.sol#L201](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L201)
- [TroveManager.sol#L203](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L203)
- [TroveManager.sol#L204](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L204)
- [TroveManager.sol#L205](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L205)
- [TroveManager.sol#L206](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L206)
- [TroveManager.sol#L207](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L207)
- [TroveManager.sol#L208](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L208)
- [TroveManager.sol#L209](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L209)
- [ReaperVaultV2.sol#L81](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L81)
- [ReaperVaultV2.sol#L82](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L82)
- [ReaperVaultV2.sol#L84](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L84)
- [ReaperVaultV2.sol#L85](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L85)
- [ReaperVaultV2.sol#L88](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L88)
- [ReaperVaultV2.sol#L89](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L89)

---
### [NC-09] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

---
**Description:**

For example, `365 days` should be written as `365 days; // 31_536_000 (365*24*60*60)`

**Lines of Code:** 

- [CommunityIssuance.sol#L58-L59](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L58-L59)

---
### [NC-10] Constant values such as a call to keccak256(), should used to immutable rather than constant

---
**Description:**

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

**Lines of Code:**

- [ReaperBaseStrategyv4.sol#L49-#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-#L52)
- [ReaperVaultV2.sol#L73-L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)


---
### [NC-11] Return parameters are declared, hence no need return variable.

---

```diff
// TroveManager.sol

function _liquidateNormalMode(
		IActivePool _activePool,
		IDefaultPool _defaultPool,
		address _collateral,
		address _borrower,
		uint _LUSDInStabPool
)
		internal
		returns (LiquidationValues memory singleLiquidation)
{
		LocalVariables_InnerSingleLiquidateFunction memory vars;

		(singleLiquidation.entireTroveDebt,
		singleLiquidation.entireTroveColl,
		vars.pendingDebtReward,
		vars.pendingCollReward) = getEntireDebtAndColl(_borrower, _collateral);

		_movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);
		_removeStake(_borrower, _collateral);

		singleLiquidation.collGasCompensation = _getCollGasCompensation(singleLiquidation.entireTroveColl);
		singleLiquidation.LUSDGasCompensation = LUSD_GAS_COMPENSATION;
		uint collToLiquidate = singleLiquidation.entireTroveColl.sub(singleLiquidation.collGasCompensation);

		(singleLiquidation.debtToOffset,
		singleLiquidation.collToSendToSP,
		singleLiquidation.debtToRedistribute,
		singleLiquidation.collToRedistribute) = _getOffsetAndRedistributionVals(singleLiquidation.entireTroveDebt, collToLiquidate, _LUSDInStabPool);

		_closeTrove(_borrower, _collateral, Status.closedByLiquidation);
		emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInNormalMode);
		emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInNormalMode);
--		return singleLiquidation;
}
```
**Lines of Code:**

- [TroveManager.sol#L353](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L353)
- [TroveManager.sol#L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L436)
- [TroveManager.sol#L912](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L912)
- [TroveManager.sol#L1332](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1332)

---

### [NC-12] Utilise named return variable

---

```diff
// TroveManager.sol

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
		returns (LiquidationValues memory singleLiquidation)
{
	// Lines of codes

	else if ((_ICR >= _MCR) && (_ICR < _TCR) && (singleLiquidation.entireTroveDebt <= _LUSDInStabPool)) {
				_movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);
				assert(_LUSDInStabPool != 0);

				_removeStake(_borrower, _collateral);
				uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);
				singleLiquidation = _getCappedOffsetVals(singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, _price, _MCR, collDecimals);

				_closeTrove(_borrower, _collateral, Status.closedByLiquidation);
				if (singleLiquidation.collSurplus > 0) {
						collSurplusPool.accountSurplus(_borrower, _collateral, singleLiquidation.collSurplus);
				}

				emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.collToSendToSP, TroveManagerOperation.liquidateInRecoveryMode);
				emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.liquidateInRecoveryMode);

		} else { // if (_ICR >= _MCR && ( _ICR >= _TCR || singleLiquidation.entireTroveDebt > _LUSDInStabPool))
-				LiquidationValues memory zeroVals;
-				return zeroVals;
+				singleLiquidation = LiqudationValues();
		}
}
```

**Lines of Code:**

- [TroveManager.sol#L433](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L433)