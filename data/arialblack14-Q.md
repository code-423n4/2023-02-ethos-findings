# QA REPORT

### Summary of low risk issues


| Number      | Issue details                                         | Instances |
| ------------- | ------------------------------------------------------- | :---------: |
| [L-1](#L1)  | Use of`block.timestamp`                               |    20    |
| [L-2](#L3)  | `ecrecover()` not checked for signer address of zero. |     2     |
| [L-3](#L6)  | Avoid hardcoded values.                               |     2     |
| [L-4](#L8)  | Lock pragmas to specific compiler version.            |     4     |

*Total: 4 issues.*

### Summary of non-critical issues


| Number         | Issue details                                                                                          | Instances |
| ---------------- | -------------------------------------------------------------------------------------------------------- | :---------: |
| [NC-1](#NC1)   | Consider adding checks for signature malleability.                                                     |     1     |
| [NC-2](#NC3)   | Missing timelock for critical changes.                                                                 |     7     |
| [NC-3](#NC5)   | Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`).                        |     2     |
| [NC-4](#NC6)   | Lines are too long.                                                                                    |    11    |
| [NC-5](#NC7)   | For modern and more readable code, update import usages.                                               |    102    |
| [NC-6](#NC11) | Avoid whitespaces while declaring mapping variables.                                                   |    35    |
| [NC-7](#NC12) | Use of`bytes.concat()` instead of `abi.encodePacked()`.                                                |     1     |

*Total: 7 issues.*

---

### Low Risk Issues

### <a id=L1>[L-1]</a> Use of `block.timestamp`

##### Description

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the *[Entropy Illusion](https://hacken.io/discover/most-common-smart-contract-vulnerabilities/#Entropy_Illusion)* for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

##### Recommendation

Use oracle instead of `block.timestamp`

##### *Instances (20):*

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87 )

```solidity
87: uint256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;
93: lastIssuanceTimestamp = block.timestamp;
113: lastDistributionTime = block.timestamp.add(distributionPeriod);
114: lastIssuanceTimestamp = block.timestamp;
```

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L128 )

```solidity
128: deploymentStartTime = block.timestamp;
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L1501 )

```solidity
1501: uint timePassed = block.timestamp.sub(lastFeeOperationTime);
1504: lastFeeOperationTime = block.timestamp;
1505: emit LastFeeOpTimeUpdated(block.timestamp);
1517: return (block.timestamp.sub(lastFeeOperationTime)).div(SECONDS_IN_ONE_MINUTE);
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L121 )

```solidity
121: constructionTime = block.timestamp;
122: lastReport = block.timestamp;
159: activation: block.timestamp,
165: lastReport: block.timestamp
419: uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;
540: strategy.lastReport = block.timestamp;
541: lastReport = block.timestamp;
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L137 )

```solidity
137: lastHarvestTimestamp = block.timestamp;
169: upgradeProposalTime = block.timestamp;
181: upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;
192: upgradeProposalTime + UPGRADE_TIMELOCK < block.timestamp,
```

### <a id=L3>[L-2]</a> `ecrecover()` not checked for signer address of zero.

##### Description

The `ecrecover()` function returns an address of zero when the signature does not match. This can cause problems if address zero is ever the owner of assets, and someone uses the permit function on address zero.

##### Recommendation

You should check whether the return value is address(0) or not in terms of the security.

##### *Instances (1):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L287 )

```solidity
275: // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
287: address recoveredAddress = ecrecover(digest, v, r, s);
```

### <a id=L6>[L-3]</a> Avoid hardcoded values.

##### Description

It is not good practice to hardcode values, but if you are dealing with addresses much less, these can change between implementations, networks or projects, so it is convenient to remove these values from the source code.

##### Recommendation

Use the selector instead of the raw value.

##### *Instances (2):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L42 )

```solidity
42: bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
44: bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;
```

### <a id=L8>[L-4]</a> Lock pragmas to specific compiler version.

##### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

##### Recommendation

Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version. [solidity-specific/locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

##### *Instances (4):*

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3 )

```solidity
3: pragma solidity ^0.8.0;
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3 )

```solidity
3: pragma solidity ^0.8.0;
```

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3 )

```solidity
3: pragma solidity ^0.8.0;
```

File: [2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3 )

```solidity
3: pragma solidity ^0.8.0;
```

### Non-critical Issues

### <a id=NC1>[NC-1]</a> Consider adding checks for signature malleability.

##### Description

Consider adding checks for signature malleability

##### Recommendation

Use OpenZeppelin's ECDSA contract rather than calling ecrecover() directly

##### *Instances (1):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L275 )

```solidity
275: // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
287: address recoveredAddress = ecrecover(digest, v, r, s);
```

### <a id=NC3>[NC-2]</a> Missing timelock for critical changes.

##### Description

A timelock should be added to functions that perform critical changes.

##### Recommendation

TODO: Add Timelock for the following functions.

##### *Instances (7):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L125 )

```solidity
125: function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
214: function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101 )

```solidity
101: function fund(uint amount) external onlyOwner {
120: function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
```



### <a id=NC5>[NC-3]</a> Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`).

##### Description

Use scientific notation (e.g.`1e18`) rather than exponentiation (e.g.`10**18`) that could lead to errors.

##### Recommendation

Use scientific notation instead of exponentiation.

##### *Instances (2):*

File: [2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40 )

```solidity
40: uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

### <a id=NC6>[NC-4]</a> Lines are too long.

##### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

##### Recommendation

Reduce number of characters per line to improve readability.

##### *Instances (11):*

File: [2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/BorrowerOperations.sol#L268 )

```solidity
268: function adjustTrove(address _collateral, uint _maxFeePercentage, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint) external override {
282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {
343: (vars.newColl, vars.newDebt) = _updateTroveFromAdjustment(contractsCache.troveManager, _borrower, _collateral, vars.collChange, vars.isCollIncrease, vars.netDebtChange, _isDebtIncrease);
511: function _withdrawLUSD(IActivePool _activePool, ILUSDToken _lusdToken, address _collateral, address _account, uint _LUSDAmount, uint _netDebtIncrease) internal {
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L637 )

```solidity
637: function _getCollateralGainFromSnapshots(uint initialDeposit, Snapshots storage snapshots) internal view returns (address[] memory assets, uint[] memory amounts) {
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L351 )

```solidity
351: emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInNormalMode);
393: emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
407: emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.entireTroveColl, TroveManagerOperation.liquidateInRecoveryMode);
428: emit TroveLiquidated(_borrower, _collateral, singleLiquidation.entireTroveDebt, singleLiquidation.collToSendToSP, TroveManagerOperation.liquidateInRecoveryMode);
934: * The redeemer swaps (debt - liquidation reserve) LUSD for (debt - liquidation reserve) worth of collateral, so the LUSD liquidation reserve left corresponds to the remaining debt.
1044: // Return the nominal collateral ratio (ICR) of a given Trove, without the price. Takes a trove's pending coll and debt rewards from redistributions into account.
```

### <a id=NC7>[NC-5]</a> For modern and more readable code, update import usages.

##### Description

Solidity code is cleaner in the following way: On the principle that clearer code is better code, you should import the things you want to use. Specific imports with curly braces allow us to apply this rule better. Check out this [article](https://betterprogramming.pub/solidity-tutorial-all-about-imports-c65110e41f3a)

##### Recommendation

Import like this: `import {contract1 , contract2} from "filename.sol";`

##### *Instances (102):*

```solidity
All imports.
```

### <a id=NC11>[NC-6]</a> Avoid whitespaces while declaring mapping variables.

##### Description

Style Guide helps to maintain code layout consistent and make code more readable.

##### Recommendation

Avoid whitespaces while declaring mapping variables.

##### *Instances (35):*

File: [2023-02-ethos/Ethos-Core/contracts/ActivePool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/ActivePool.sol#L41 )

```solidity
41: mapping (address => uint256) internal collAmount;  // collateral => amount tracker
42: mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker
44: mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
45: mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
46: mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
47: mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
```

File: [2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/CollateralConfig.sol#L35 )

```solidity
35: mapping (address => Config) public collateralConfig; // for O(1) checking of collateral's validity and properties
```

File: [2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L28 )

```solidity
28: mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked
32: mapping (address => Snapshot) public snapshots;
35: mapping (address => uint) F_Collateral_Snapshot;
```

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L54 )

```solidity
54: mapping (address => uint256) private _nonces;
57: mapping (address => uint256) private _balances;
58: mapping (address => mapping (address => uint256)) private _allowances;
62: mapping (address => bool) public troveManagers;
63: mapping (address => bool) public stabilityPools;
64: mapping (address => bool) public borrowerOperations;
```

File: [2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/StabilityPool.sol#L99 )

```solidity
99: * In the current epoch, the latest value of S is stored upon each scale change, and the mapping (scale -> S) is stored for each epoch.
167: mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
179: mapping (address => uint) S;
186: mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct
187: mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct
208: * The 'S' sums are stored in a nested mapping (epoch => scale => collateral => sum):
214: mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
223: mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
228: mapping (address => uint) public lastCollateralError_Offset;
```

File: [2023-02-ethos/Ethos-Core/contracts/TroveManager.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/TroveManager.sol#L86 )

```solidity
86: mapping (address => mapping (address => Trove)) public Troves;
88: mapping (address => uint) public totalStakes;
91: mapping (address => uint) public totalStakesSnapshot;
94: mapping (address => uint) public totalCollateralSnapshot;
104: mapping (address => uint) public L_Collateral;
105: mapping (address => uint) public L_LUSDDebt;
109: mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;
116: mapping (address => address[]) public TroveOwners;
119: mapping (address => uint) public lastCollateralError_Redistribution;
120: mapping (address => uint) public lastLUSDDebtError_Redistribution;
```

### <a id=NC12>[NC-7]</a> Use of `bytes.concat()` instead of `abi.encodePacked()`.

##### Description

Rather than using `abi.encodePacked` for appending bytes, since version `0.8.4`, `bytes.concat()` is enabled.

##### Recommendation

Since version `0.8.4` for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked()`.

##### *Instances (1):*

File: [2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol](https://github.com/code-423n4/2023-02-ethos/tree/main/Ethos-Core/contracts/LUSDToken.sol#L283 )

```solidity
283: bytes32 digest = keccak256(abi.encodePacked('\x19\x01',
```
