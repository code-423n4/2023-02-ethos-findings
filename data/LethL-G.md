## Findings Summary

|  | Issue | Instances |
|---|---|---|
| Gas-1 | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 9 |
| Gas-2 | Use of Named Returns for Local Variables Saves Gas | 35 |
| Gas-3 | Using a ternary operator instead of an "if-else" statement | 5 |
| Gas-4 | Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant | 8 |
| Gas-5 | Pre-calculate the hardcoded hash with `keccak256()` before and only use the result to save gas | 10 |
| Gas-6 | Before some functions, we should check some variables for possible gas save | 1 |

### [Gas-1] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

Using `x = x+y` instead of `x += y` can help to save gas.

Similarly, using `<x> -= <y>` costs more gas than  `<x> = <x> - <y>`.

Instances(9):

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

168:        totalAllocBPS += _allocBPS;

194:        totalAllocBPS -= strategies[_strategy].allocBPS;

196:        totalAllocBPS += _allocBPS;

214:        totalAllocBPS -= strategies[_strategy].allocBPS;

396:                totalAllocated -= actualWithdrawn;

445:                totalAllocBPS -= bpsChange;

452:        totalAllocated -= loss;

515:                totalAllocated -= vars.debtPayment;

521:        totalAllocated += vars.credit;
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

### [Gas-2] Use of Named Returns for Local Variables Saves Gas

Using named return values as temporary local variables can help you conserve gas.

For example, the following code block can be refactored as follows:
```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

230:    function balanceOfPool() public view returns (uint256 supply) {
231:        (supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
232:        address(want),
233:        address(this)
234:        );
235:    }
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L230-L236)

Instances(35)
```
File: Ethos-Core/contracts/BorrowerOperations.sol

421:        uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);

433:        uint usdValue = _price.mul(_coll).div(10**_collDecimals);

468:        uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange)

470:        uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange)

677:        uint newNICR = LiquityMath._computeNominalCR(newColl, newDebt, _collateralDecimals);

699:        uint newICR = LiquityMath._computeCR(newColl, newDebt, _price, _collateralDecimals);

715:        uint newColl = _coll;

716:        uint newDebt = _debt;

744:        uint newTCR = LiquityMath._computeCR(totalColl, totalDebt, _price, _collateralDecimals);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```
File: Ethos-Core/contracts/TroveManager.sol

1049:        uint NICR = LiquityMath._computeNominalCR(currentCollateral, currentLUSDDebt, collDecimals);

1062:        uint ICR = LiquityMath._computeCR(currentCollateral, currentLUSDDebt, _price, collDecimals);

1070:        uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);

1071:        uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);

1132:        uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1147:        uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);

1202:        uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);

1214:        uint stake;

1362:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

1411:        uint newBaseRate = decayedBaseRate.add(redeemedLUSDFraction.div(BETA));

1449:        uint redemptionFee = _redemptionRate.mul(_collateralDrawn).div(DECIMAL_PRECISION);

1569:        uint newColl = Troves[_borrower][_collateral].coll.add(_collIncrease);

1576:        uint newColl = Troves[_borrower][_collateral].coll.sub(_collDecrease);

1583:        uint newDebt = Troves[_borrower][_collateral].debt.add(_debtIncrease);

1590:        uint newDebt = Troves[_borrower][_collateral].debt.sub(_debtDecrease);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol)

```
File: Ethos-Core/contracts/StabilityPool.sol

453:        uint LQTYPerUnitStaked = LQTYNumerator.div(_totalLUSDDeposits);

688:        uint LQTYGain = _getLQTYGainFromSnapshots(initialDeposit, snapshots);

707:        uint LQTYGain = initialDeposit.mul(firstPortion.add(secondPortion)).div(P_Snapshot).div(DECIMAL_PRECISION);

724:        uint compoundedDeposit = _getCompoundedDepositFromSnapshots(initialDeposit, snapshots);

744:        uint compoundedDeposit;
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol)

```
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

219:        uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol)

```
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

272:        uint256 q = x / y;
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol)

```
Ethos-Vault/contracts/ReaperVaultV2.sol

463:        uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;

660:        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```
Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

204:        bytes32[] memory cascadingAccessRoles = new bytes32[](5);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

```
Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

231:        (uint256 supply, , , , , , , , ) = IAaveProtocolDataProvider(DATA_PROVIDER).getUserReserveData(
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol)

### [Gas-3] Using a ternary operator instead of an "if-else" statement

For simple "if-else" statements that are easy to read, replacing a "if-else" statement with a ternary operator can help you save gas.

Instances(5):
```
File: Ethos-Core/contracts/BorrowerOperations.sol

490:        if (_isDebtIncrease) {

649:        if (_isRecoveryMode) {
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

331:        if (totalSupply() == 0) {

534:        if (vars.lockedProfitBeforeLoss > vars.loss) {

604:        if (_active) {
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

### [Gas-4] Expressions for constant values such as a call to `keccak256()`, should use immutable rather than constant

The consequence of this is that the `keccak256()` operation is carried out each time the variable is utilized, leading to a rise in gas costs compared to just storing the calculated hash. By making it immutable, the hashing will only be done during contract deployment, which will reduce gas consumption.

Instances(8):
```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:    bytes32 public constant ADMIN = keccak256("ADMIN");
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:    bytes32 public constant KEEPER = keccak256("KEEPER");
50:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:    bytes32 public constant ADMIN = keccak256("ADMIN");
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

### [Gas-5] Pre-calculate the hardcoded hash with `keccak256()` before and only use the result to save gas

Performing the hardcoded `keccak256()` hash calculation every time the contract is deployed or a function is called is less gas-efficient than pre-calculating the hash beforehand and using only its result.

For example, the following code block can be refactored as follows:
```
File: Ethos-Core/contracts/LUSDToken.sol

123:        _HASHED_NAME = 0xabccaf2943f70764a048255e50e07d10e3c94973a6c6ba8b8ea62b1155209b01; // keccak256(bytes(_NAME));
124:        _HASHED_VERSION = 0xc89efdaa54c0f20c7adf612882df0950f5a951637e0307cdcb4c672f298b8bc6; // keccak256(bytes(_VERSION));
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L123-L124)

Instances(10):
```
File: Ethos-Core/contracts/LUSDToken.sol

123:        _HASHED_NAME = hashedName;
124:        _HASHED_VERSION = hashedVersion;
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

```
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:    bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:    bytes32 public constant ADMIN = keccak256("ADMIN");
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:    bytes32 public constant KEEPER = keccak256("KEEPER");
50:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:    bytes32 public constant ADMIN = keccak256("ADMIN");
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol)

### [Gas-6] Before some functions, we should check some variables for possible gas save

Before making a transfer, it is advisable to check if the amount is 0 to prevent the function from running and incurring gas costs unnecessarily when no transfer will occur.

Instances(1):
```

File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

127:        OathToken.transfer(_account, _OathAmount);
```
[Link to Code](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol)