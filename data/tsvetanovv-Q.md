
## Low Risk Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[L-01]| Initial value check is missing in Set Functions
|[L-02]| Critical address changes should use two-step procedure 
|[L-03]| Missing event for critical parameter change
|[L-04]| Missing checks for address(0x0)  
|[L-05]| No storage gap for upgradable contracts might lead to storage slot collision
 
***

## Non-Critical Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[NC-01]| Use latest Solidity version
|[NC-02]| Use stable pragma statement
|[NC-03]| Use named imports instead of plain `import FILE.SOL`
|[NC-04]| Update external dependency to latest version
|[NC-05]| Solidity compiler optimizations can be problematic
|[NC-06]| Add NatSpec documentation
|[NC-07]| Commented code
|[NC-08]| Omissions in Events
|[NC-09]| Constants should be defined rather than using magic numbers
|[NC-10]| Constant values such as a call to keccak256(), should used to immutable rather than constant
|[NC-11]| You can use named parameters in mapping types
***


## [L-01] Initial value check is missing in Set Functions

```solidity
Ethos-Core/contracts/BorrowerOperations.sol:

110: function setAddresses(
282: function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {

Ethos-Core/contracts/TroveManager.sol:
1562: function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
1567: function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
1574: function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {
1581: function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
1588: function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

Ethos-Core/contracts/ActivePool.sol
125: function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {
138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {
144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {
486: function updateRewardSum(address _collateral, uint _collToAdd) external override {

Ethos-Vault/contracts/ReaperVaultV2.sol
617: function setLockedProfitDegradation(uint256 degradation) external {
```
Checking whether the current value and the new value are the same should be added.
***
## [L-02] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE

The critical procedures should be a two-step process.
```solidity 
Ethos-Vault/contracts/ReaperVaultV2.sol
627: function updateTreasury(address newTreasury) external {
```

**Recommended Mitigation Steps**
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding a two- step procedure on the critical functions.
***

## [L-03] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

Emitting events allows monitoring activities with off-chain monitoring tools.
```solidity
Ethos-Core/contracts/BorrowerOperations.sol:

412: function claimCollateral(address _collateral) external override {
419: function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {
455: function _updateTroveFromAdjustment
476: function _moveTokensAndCollateralfromAdjustment

Ethos-Core/contracts/TroveManager.sol:
1016: function redeemCollateral(
1562: function setTroveStatus(address _borrower, address _collateral, uint _num) external override {
1567: function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
1574: function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {
1581: function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
1588: function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

Ethos-Core/contracts/ActivePool.sol
190: function increaseLUSDDebt(address _collateral, uint _amount) external override {
197: function decreaseLUSDDebt(address _collateral, uint _amount) external override {

Ethos-Vault/contracts/ReaperVaultV2.sol
627: function updateTreasury(address newTreasury) external {

Ethos-Vault/contracts/ReaperBaseStrategyv4.sol
156: function setEmergencyExit() external {
```
***

## [L-04] MISSING CHECKS FOR ADDRESS(0X0)

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

```solidity
Ethos-Core/contracts/BorrowerOperations.sol:
412: function claimCollateral(address _collateral) external override {
455: function _updateTroveFromAdjustment
476: function _moveTokensAndCollateralfromAdjustment

Ethos-Core/contracts/TroveManager.sol: 
513: function liquidateTroves(address _collateral, uint _n) external override {
939: function redeemCloseTrove(
962: function updateDebtAndCollAndStakesPostRedemption(
982: function burnLUSDAndEmitRedemptionEvent(
1016: function redeemCollateral(
```

#### Recommended Mitigation Steps

Consider adding explicit zero-address validation on input parameters of address type.
***

## [L-05] No storage gap for upgradable contracts might lead to storage slot collision

#### Summary
For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments”. 
Otherwise, it may be very difficult to write new implementation code. Without storage gap, the variable in the contract contract might be overwritten by the upgraded contract if new variables are added. 
This could have unintended and very serious consequences to the child contracts.

This contracts are intended to be upgradeable contract in the code base:
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14

#### Impact

Without storage gap, the variable in the contract contract might be overwritten by the upgraded contract if new variables are added.  This could have unintended and very serious consequences to the child contracts.

#### Recommended Mitigation Steps
Consider defining an appropriate storage gap in each upgradeable parent contract at the end of all the storage variable definitions as follows:

```solidity
uint256[50] __gap; // gap to reserve storage in the contract for future variable additions
```
***


## [NC-01] Use latest Solidity version

Solidity pragma versioning should be upgraded to latest available version. 
***

## [NC-02] Use stable pragma statement

Using a floating pragma statement `^0.8.0` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
***


## [NC-03] USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT FILE.SOL`

**Recommendation:**
`import {contract1 , contract2} from "filename.sol";
***

## [NC-04] Update external dependency to latest version

The latest version is 4.8.2

Ethos-Core/package.json:
```solidity
34: "@openzeppelin/contracts": "^3.3.0",
```

Ethos-Vault/package.json:
```solidity
41: "@openzeppelin/contracts": "^4.7.3",
42: "@openzeppelin/contracts-upgradeable": "^4.7.3",
```

***

## [NC-05] Solidity compiler optimizations can be problematic

hardhat.config.js:
```solidity
compilers: [
      {
        version: "0.8.11",
        settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
 ```
 
Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.
***

## [N-06] Add NatSpec documentation

NatSpec documentation to all public/external functions and variables is essential for better understanding of the code by developers and auditors and is strongly recommended.
***

## [NC-07] COMMENTED CODE

```solidity
Ethos-Core/contracts/TroveManager.sol

14: // import "./Dependencies/Ownable.sol";
18: /*Ownable,*/ 
19: // string constant public NAME = "TroveManager";
```
***

## [NC-08]  Omissions in Events
Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

Events with no old value;
```solidity
Ethos-Core/contracts/LUSDToken.sol
150: emit GovernanceAddressChanged(_newGovernanceAddress);
157: emit GuardianAddressChanged(_newGuardianAddress);
172: emit TroveManagerAddressChanged(_newTroveManagerAddress);
176: emit StabilityPoolAddressChanged(_newStabilityPoolAddress);
180: emit BorrowerOperationsAddressChanged(_newBorrowerOperationsAddress);
```
***

## [NC-09] Constants should be defined rather than using magic numbers

```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol
125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6
```
***

## [NC-10] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.
```solidity
Ethos-Vault/contracts/ReaperVaultV2.sol
73: bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76: bytes32 public constant ADMIN = keccak256("ADMIN");

Ethos-Vault/contracts/ReaperBaseStrategyv4.sol
49: bytes32 public constant KEEPER = keccak256("KEEPER");
50: bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51: bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52: bytes32 public constant ADMIN = keccak256("ADMIN");
```
***

## [NC-11] You can use named parameters in mapping types

From Solidity [0.8.18](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/) you can use named parameters in mapping types. This will make the code much more readable.

For example check code below:

In `TroveManager.sol`:

From:
```solidity
85: // user => (collateral type => trove)
86: mapping (address => mapping (address => Trove)) public Troves;
```

To:
```solidity
86: mapping (address user => mapping (address collateralType=> Trove)) public Troves;
```

This way you can put named parameters in mapping types everywhere in the code.
***