

## [L-01] Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

```
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
```

## [L-02] Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

```
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::87 => uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::93 => lastIssuanceTimestamp = block.timestamp;
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::113 => lastDistributionTime = block.timestamp.add(distributionPeriod);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::114 => lastIssuanceTimestamp = block.timestamp;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::128 => deploymentStartTime = block.timestamp;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::417 => if (_response.timestamp == 0 || _response.timestamp > block.timestamp) {return true;}
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::425 => return block.timestamp.sub(_response.timestamp) > TIMEOUT;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::450 => if (_response.timestamp == 0 || _response.timestamp > block.timestamp) {return true;}
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::458 => return block.timestamp.sub(_tellorResponse.timestamp) > TIMEOUT;
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::309 => require(block.timestamp >= systemDeploymentTime.add(BOOTSTRAP_PERIOD));
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1501 => uint timePassed = block.timestamp.sub(lastFeeOperationTime);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1504 => lastFeeOperationTime = block.timestamp;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1505 => emit LastFeeOpTimeUpdated(block.timestamp);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1517 => return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::26 => uint256 activation; // Activation block.timestamp
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::32 => uint256 lastReport; // block.timestamp of the last time a report occured
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::46 => uint256 public lastReport; // block.timestamp of last report from any strategy
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::121 => constructionTime = block.timestamp;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::122 => lastReport = block.timestamp;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::159 => activation: block.timestamp,
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::165 => lastReport: block.timestamp
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::419 => uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::540 => strategy.lastReport = block.timestamp;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::541 => lastReport = block.timestamp;
```

## [L-03] Open TODOs

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

```
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::335 => /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::380 => /* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
```

## [L-04] decimals() not part of ERC20 standard

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

```
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::63 => uint256 decimals = IERC20(collateral).decimals();
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::580 => try priceAggregator[_collateral].decimals() returns (uint8 decimals) {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::651 => return token.decimals();
```

## [L-05] pragma experimental ABIEncoderV2 is deprecated

Use pragma abicoder v2 instead

```
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::4 => pragma experimental ABIEncoderV2;
```

## [L-06] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). Unless there is a compelling reason, abi.encode should be preferred. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

```
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::178 => latestRandomSeed = uint(keccak256(abi.encodePacked(latestRandomSeed)));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::283 => bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
```

## [L-07] require() should be used instead of assert()

require() should be used for checking error conditions on inputs and return values while assert() should be used for invariant checking

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::128 => assert(MIN_NET_DEBT > 0);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::197 => assert(vars.compositeDebt > 0);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::301 => assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::331 => assert(_collWithdrawal <= vars.coll);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::312 => assert(sender != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::313 => assert(recipient != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::321 => assert(account != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::329 => assert(account != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::337 => assert(owner != address(0));
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::338 => assert(spender != address(0));
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::128 => assert(lusdToken.balanceOf(_redeemer) <= totals.totalLUSDSupplyAtStart);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::526 => assert(_debtToOffset <= _totalLUSDDeposits);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::551 => assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::591 => assert(newP > 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::417 => assert(_LUSDInStabPool != 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1219 => * The following assert() holds true because:
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1224 => assert(totalStakesSnapshot[_collateral] > 0);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1279 => assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1342 => assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1348 => assert(index <= idxLast);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1413 => //assert(newBaseRate <= DECIMAL_PRECISION); // This is already enforced in the line above
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1414 => assert(newBaseRate > 0); // Base rate is always non-zero after redemption
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1489 => assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```

## [L-08] Unsafe use of transfer()/transferFrom() with IERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::103 => OathToken.transferFrom(msg.sender, address(this), amount);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::127 => OathToken.transfer(_account, _OathAmount);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::135 => lusdToken.transfer(msg.sender, LUSDGain);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::171 => lusdToken.transfer(msg.sender, LUSDGain);
```

## [L-09] Events not emitted for important state changes

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

```
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::17 => function setCompleted(uint completed) public restricted {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::54 => function setAddresses(
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1562 => function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::160 => function setHarvestSteps(address[2][] calldata _newSteps) external {
```

## [L-26] _safeMint() should be used rather than _mint() wherever possible

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::188 => _mint(_account, _amount);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::320 => function _mint(address account, uint256 amount) internal {
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::336 => _mint(_receiver, shares);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::467 => _mint(treasury, shares);
```

## [L-26] Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

__gap is empty reserved space in storage that is recommended to be put in place in upgradeable contracts. It allows new state variables to be added in the future without compromising the storage compatibility with existing deployments

```
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::13 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
```

## [L-26] Implementation contract may not be initialized

Implementation contract does not have a constructor with the initializer modifier therefore may be uninitialized. Implementation contracts should be initialized to avoid potential griefs or exploits.

```
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::13 => import "@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
```

## [N-1] Use a more recent version of solidity

Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)
Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)
Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/GasPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::3 => pragma solidity 0.6.11;
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
```

## [N-2] Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 1000000), for better code readability

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::145 => require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::41 => uint256 public constant PERCENT_DIVISOR = 10000;
```

## [N-3] type(uint<n>).max should be used instead of uint<n>(-1)

type(uint<n>).max should be used instead for better code readability

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::258 => vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
```

## [N-4] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

Scientific notation should be used for better code readability

```
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::40 => uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
```

## [N-5] Event is missing indexed fields

Each event should use three indexed fields if there are three or more fields

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::58 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::59 => event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::60 => event TroveManagerAddressChanged(address _newTroveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::61 => event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::62 => event ActivePoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::63 => event ActivePoolCollateralBalanceUpdated(address _collateral, uint _amount);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::64 => event YieldingPercentageUpdated(address _collateral, uint256 _bps);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::65 => event YieldingPercentageDriftUpdated(uint256 _driftBps);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::66 => event YieldClaimThresholdUpdated(address _collateral, uint256 _threshold);
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::67 => event YieldDistributionParamsUpdated(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::92 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::93 => event TroveManagerAddressChanged(address _newTroveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::94 => event ActivePoolAddressChanged(address _activePoolAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::95 => event DefaultPoolAddressChanged(address _defaultPoolAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::96 => event StabilityPoolAddressChanged(address _stabilityPoolAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::97 => event GasPoolAddressChanged(address _gasPoolAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::98 => event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::99 => event PriceFeedAddressChanged(address  _newPriceFeedAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::100 => event SortedTrovesAddressChanged(address _sortedTrovesAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::101 => event LUSDTokenAddressChanged(address _lusdTokenAddress);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::102 => event LQTYStakingAddressChanged(address _lqtyStakingAddress);
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::31 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::32 => event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::33 => event TroveManagerAddressChanged(address _newTroveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::34 => event ActivePoolAddressChanged(address _newActivePoolAddress);
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::37 => event CollateralSent(address _collateral, address _to, uint _amount);
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::37 => event CollateralWhitelisted(address _collateral, uint256 _decimals, uint256 _MCR, uint256 _CCR);
2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol::38 => event CollateralRatiosUpdated(address _collateral, uint256 _MCR, uint256 _CCR);
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::33 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::34 => event TroveManagerAddressChanged(address _newTroveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::35 => event DefaultPoolLUSDDebtUpdated(address _collateral, uint _LUSDDebt);
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::36 => event DefaultPoolCollateralBalanceUpdated(address _collateral, uint _amount);
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::21 => event CollateralConfigAddressChanged(address _collateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::22 => event SortedTrovesAddressChanged(address _sortedTrovesAddress);
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::23 => event TroveManagerAddressChanged(address _troveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::50 => event OathTokenAddressSet(address _oathTokenAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::51 => event LogRewardPerSecond(uint _rewardPerSecond);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::52 => event StabilityPoolAddressSet(address _stabilityPoolAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::53 => event TotalOATHIssuedUpdated(uint _totalOATHIssued);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::49 => event LQTYTokenAddressSet(address _lqtyTokenAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::50 => event LUSDTokenAddressSet(address _lusdTokenAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::51 => event TroveManagerAddressSet(address _troveManager);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::52 => event BorrowerOperationsAddressSet(address _borrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::53 => event ActivePoolAddressSet(address _activePoolAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::54 => event CollateralConfigAddressSet(address _collateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::58 => event F_CollateralUpdated(address _collateral, uint _F_Collateral);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::59 => event F_LUSDUpdated(uint _F_LUSD);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::60 => event TotalLQTYStakedUpdated(uint _totalLQTYStaked);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::61 => event CollateralSent(address _account, address _collateral, uint _amount);
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::62 => event StakerSnapshotsUpdated(address _staker, address[] _assets, uint[] _amounts, uint _F_LUSD);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::78 => event TroveManagerAddressChanged(address _troveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::79 => event StabilityPoolAddressChanged(address _newStabilityPoolAddress);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::80 => event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::81 => event GovernanceAddressChanged(address _governanceAddress);
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::82 => event GuardianAddressChanged(address _guardianAddress);
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::84 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::85 => event LastGoodPriceUpdated(address _collateral, uint _lastGoodPrice);
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::86 => event PriceFeedStatusChanged(address _collateral, Status newStatus);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::51 => event TroveManagerAddressChanged(address _troveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::52 => event BorrowerOperationsAddressChanged(address _borrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::53 => event NodeAdded(address _collateral, address _id, uint _NICR);
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::54 => event NodeRemoved(address _collateral, address _id);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::233 => event StabilityPoolCollateralBalanceUpdated(address _collateral, uint _newBalance);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::234 => event StabilityPoolLUSDBalanceUpdated(uint _newBalance);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::236 => event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::237 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::238 => event TroveManagerAddressChanged(address _newTroveManagerAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::239 => event ActivePoolAddressChanged(address _newActivePoolAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::240 => event DefaultPoolAddressChanged(address _newDefaultPoolAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::241 => event LUSDTokenAddressChanged(address _newLUSDTokenAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::242 => event SortedTrovesAddressChanged(address _newSortedTrovesAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::243 => event PriceFeedAddressChanged(address _newPriceFeedAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::244 => event CommunityIssuanceAddressChanged(address _newCommunityIssuanceAddress);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::246 => event P_Updated(uint _P);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::247 => event S_Updated(address _collateral, uint _S, uint128 _epoch, uint128 _scale);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::248 => event G_Updated(uint _G, uint128 _epoch, uint128 _scale);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::249 => event EpochUpdated(uint128 _currentEpoch);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::250 => event ScaleUpdated(uint128 _currentScale);
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::257 => event CollateralSent(address _collateral, address _to, uint _amount);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::186 => event BorrowerOperationsAddressChanged(address _newBorrowerOperationsAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::187 => event CollateralConfigAddressChanged(address _newCollateralConfigAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::188 => event PriceFeedAddressChanged(address _newPriceFeedAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::189 => event LUSDTokenAddressChanged(address _newLUSDTokenAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::190 => event ActivePoolAddressChanged(address _activePoolAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::191 => event DefaultPoolAddressChanged(address _defaultPoolAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::192 => event StabilityPoolAddressChanged(address _stabilityPoolAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::193 => event GasPoolAddressChanged(address _gasPoolAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::194 => event CollSurplusPoolAddressChanged(address _collSurplusPoolAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::195 => event SortedTrovesAddressChanged(address _sortedTrovesAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::196 => event LQTYTokenAddressChanged(address _lqtyTokenAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::197 => event LQTYStakingAddressChanged(address _lqtyStakingAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::198 => event RedemptionHelperAddressChanged(address _redemptionHelperAddress);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::200 => event Liquidation(address _collateral, uint _liquidatedDebt, uint _liquidatedColl, uint _collGasCompensation, uint _LUSDGasCompensation);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::203 => event BaseRateUpdated(uint _baseRate);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::204 => event LastFeeOpTimeUpdated(uint _lastFeeOpTime);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::205 => event TotalStakesUpdated(address _collateral, uint _newTotalStakes);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::206 => event SystemSnapshotsUpdated(address _collateral, uint _totalStakesSnapshot, uint _totalCollateralSnapshot);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::207 => event LTermsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::208 => event TroveSnapshotsUpdated(address _collateral, uint _L_Collateral, uint _L_LUSDDebt);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::209 => event TroveIndexUpdated(address _borrower, address _collateral, uint _newIndex);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::84 => event UpdateWithdrawalQueue(address[] withdrawalQueue);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::85 => event WithdrawMaxLossUpdated(uint256 withdrawMaxLoss);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::86 => event EmergencyShutdown(bool active);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::87 => event InCaseTokensGetStuckCalled(address token, uint256 amount);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::88 => event TvlCapUpdated(uint256 newTvlCap);
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::89 => event LockedProfitDegradationUpdated(uint256 degradation);
```

## [N-6] The visibility for constructor is ignored

No need to add public to constructor

```
2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::57 => constructor() public {
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::13 => constructor() public {
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::27 => constructor(ICollateralConfig _collateralConfig, TroveManager _troveManager, ISortedTroves _sortedTroves) public {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::225 => constructor() public {
```

## [N-7] Missing NatSpec

Code should include NatSpec

```
2023-02-ethos/Ethos-Core/contracts/ActivePool.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/CollSurplusPool.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/DefaultPool.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/GasPool.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/HintHelpers.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/MultiTroveGetter.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1 => // SPDX-License-Identifier: BUSL-1.1
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol::1 => // SPDX-License-Identifier: BUSL-1.1
```

## [N-8] Missing event for critical parameter change

Emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following changes to the contract.

```
2023-02-ethos/Ethos-Core/contracts/Migrations.sol::17 => function setCompleted(uint completed) public restricted {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::54 => function setAddresses(
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1562 => function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::160 => function setHarvestSteps(address[2][] calldata _newSteps) external {
```

## [N-9] Non-assembly method available

assembly{ id := chainid() } => uint256 id = block.chainid
assembly { size := extcodesize() } => uint256 size = address().code.length

```
2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol::300 => chainID := chainid()
```

## [N-10] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

instances:

```
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol::76 => bytes32 public constant ADMIN = keccak256("ADMIN");
```

## [N-11] Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

```
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::268 => function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::282 => function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::343 => (vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);
2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol::511 => function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {
2023-02-ethos/Ethos-Core/contracts/PriceFeed.sol::610 => function _getPrevChainlinkResponse(address _collateral, uint80 _currentRoundId, uint8 _currentDecimals) internal view returns (ChainlinkResponse memory prevChainlinkResponse) {
2023-02-ethos/Ethos-Core/contracts/RedemptionHelper.sol::171 => if (singleRedemption.cancelledPartial) break; // Partial redemption was cancelled (out-of-date hint, or new net debt < minimum), therefore we could not redeem from the last Trove
2023-02-ethos/Ethos-Core/contracts/SortedTroves.sol::367 => function _findInsertPosition(ITroveManager _troveManager, address _collateral, uint256 _NICR, address _prevId, address _nextId) internal view returns (address, address) {
2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol::637 => function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::351 => emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInNormalMode);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::393 => emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::407 => emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::428 => emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.collToSendToSP, TroveManagerOperation.liquidateInRecoveryMode);
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::934 => * The redeemer swaps (debt - liquidation reserve) LUSD for (debt - liquidation reserve) worth of collateral, so the LUSD liquidation reserve left corresponds to the remaining debt.
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1044 => // Return the nominal collateral ratio (ICR) of a given Trove, without the price. Takes a trove's pending coll and debt rewards from redistributions into account.
```

## [N-12] 2**<n> - 1 should be re-written as type(uint<n>).max

2**<n> - 1 should be re-written as type(uint<n>).max for better code readability

```
2023-02-ethos/Ethos-Core/contracts/TroveManager.sol::1322 => /* Max array size is 2**128 - 1, i.e. ~3e30 troves. No risk of overflow, since troves have minimum LUSD
```

