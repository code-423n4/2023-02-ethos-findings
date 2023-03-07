## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:| 
| [L&#x2011;01] | should change decimal type to uint8 | 1 |
| [L&#x2011;02] | Zero address and Zero msg.value(zero amount) checks are missing | 1 |
| [L&#x2011;03] | Internal functions only called once can be inlined, code restructuring.  | 1 |
| [L&#x2011;04] | Missing check to ensure distribution period is not set as zero | 1 |
| [L&#x2011;05] | Critical Addresses Changes should adopt two-step address change procedure | 3 |
| [L&#x2011;06] | Used Hardcoded address causes no future updates and changes | 4 |

### Non-Critical Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:| 
| [N&#x2011;01] | checkContract( ) dependencies can be used instead of require( ) | 1 |
| [N&#x2011;02] | pauseMinting/unPauseMinting functions should emit event to notify users | 1 |
| [N&#x2011;03] | Function writing that does not comply with the Solidity Style Guide | - |
| [N&#x2011;04] | Do not use floting solidity compiler version. Lock the solidity compiler version  | 4 |
| [N&#x2011;05] | Events for critical change is missing | 1 |
| [N&#x2011;06] | Use scientific notation rather than exponentiation | 2 |

### Low Risk Issues
### [L&#x2011;01]  should change decimal type to uint8
The decimals of Config struct is uint256 and the function getCollateralDecimals ( ) also returns the uint256. But the decimals are usually type of uint8, referring IERC20 dependency used in contract.

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

27    struct Config {
28        bool allowed;
29        uint256 decimals;
30        uint256 MCR;
31        uint256 CCR;
32    }

---
110    function getCollateralDecimals(
111        address _collateral
112    ) external override view checkCollateral(_collateral) returns (uint256) {
113        return collateralConfig[_collateral].decimals;
114    }
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L27-L32
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L110-L114

In the below code the collateral is ERC20 token which is using IERC20 decimal function(uint8) where as the struct decimal is uint256. 

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

63            uint256 decimals = IERC20(collateral).decimals();
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63

IERC20 used as dependencies in collateralConfig.sol decimal reference, here the decimal function returns (uint8).

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

71    function decimals() external view returns (uint8);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Dependencies/IERC20.sol#L71

### Proof of Concept:
Step 1-
ICollateralConfig.getCollateralDecimals( ) return should be changed as below,
```solidity
File: Ethos-Core/contracts/Interfaces/ICollateralConfig.sol

8    function getCollateralDecimals(address _collateral) external view returns (uint8);
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/Interfaces/ICollateralConfig.sol#L8

Step 2-
Struct decimal is changed to uint8.
getCollateralDecimals ( ) returns (uint8)

```solidity
     struct Config {
         bool allowed;
-        uint256 decimals;
+        uint8 decimals;
         uint256 MCR;
         uint256 CCR;
     }
---
-            uint256 decimals = IERC20(collateral).decimals();
+            uint8 decimals = IERC20(collateral).decimals();
---
     function getCollateralDecimals(
         address _collateral
-    ) external override view checkCollateral(_collateral) returns (uint256) {
+    ) external override view checkCollateral(_collateral) returns (uint8) {
         return collateralConfig[_collateral].decimals;
     }
```
### [L&#x2011;02]   Zero address and Zero msg.value(zero amount) checks are missing
Zero address and zero amount checks should be added as require error handling. These checks the conditions and saves unnessary gas wastage. 

1) In BorrowerOperations.openTrove(... ), _collAmount is not checked to 0. This can create unnessary gas wastage.

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

172    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {

---
231        IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collAmount);

``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L172
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L231

### Proof of Concept:
Below code shall be added at top line of BorrowerOperations.openTrove( ). 

``` 
+        require(_collAmount != 0 ,"Amount must be greater than 0");

``` 

2) In ActivePool.sendCollateral( ...), _account and _amount is not checked to 0. This can create unnessary gas wastage.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

171        function sendCollateral(address _collateral, address _account, uint _amount) external override {

``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L171

### Proof of Concept:
Add the zero address and zero value checks.

```solidity
     function sendCollateral(address _collateral, address _account, uint _amount) external override {
+        require(_amount != 0, "Amount must be greater than 0");
+        require(_account != address(0), "Account can not be zero address");
         _requireValidCollateralAddress(_collateral);
         _requireCallerIsBOorTroveMorSP();
         _rebalance(_collateral, _amount);
``` 

3) In ActivePool.increaseLUSDDebt(... ), _amount is not checked to 0. This can create unnessary gas wastage.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

190    function increaseLUSDDebt(address _collateral, uint _amount) external override {
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190

### Proof of Concept:
Add zero value checks.

```solidity

    function increaseLUSDDebt(address _collateral, uint _amount) external override {
+        require(_amount != 0, "Amount must be greater than 0");
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
``` 

4) In ActivePool.decreaseLUSDDebt(... ), _amount is not checked to 0. This can create unnessary gas wastage.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

197    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L197

### Proof of Concept:
Add zero value checks.

```solidity

    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
+        require(_amount != 0, "Amount must be greater than 0");
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
    }
``` 

4) In ActivePool.pullCollateralFromBorrowerOperationsOrDefaultPool(... ), _amount is not checked to 0. This can create unnessary gas wastage.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

204    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external 
       override {
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L204

### Proof of Concept:
Add zero value checks.

```solidity

    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {
+        require(_amount != 0, "Amount must be greater than 0");
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBorrowerOperationsOrDefaultPool();
        collAmount[_collateral] = collAmount[_collateral].add(_amount);
        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);

        IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);
        _rebalance(_collateral, 0);
    }
``` 

### [L&#x2011;03]   Internal functions only called once can be inlined, code restructuring. 
The internal functions or modifier which are used only once can be avoided by directly using the require ( ) error handling conditions in functions. This will prevent repeated code and will save some gas. 
In StabilityPool.withdrawFromSP(...) function. The _amount should be zero value checked. It should also be checked that whatever amount to be withdrawn, the account address has that balance, so this functionality was missing. 

_requireUserHasDeposit( ) function is an internal function which is used only once in the contract. This internal function had enough checks, so we have deleted this internal function and proposed additional checks in withdrawFromSP ( ) function. Which can relatively save some cost and the code is more readable now.

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

367    function withdrawFromSP(uint _amount) external override {
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L367

### Proof of Concept:

```diff
diff --git a/StabilityPool.sol.orig b/StabilityPool.sol
index db163b7..b205e94 100644
--- a/StabilityPool.sol.orig
+++ b/StabilityPool.sol
@@ -365,9 +365,14 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
     * If _amount > userDeposit, the user withdraws all of their compounded deposit.
     */
     function withdrawFromSP(uint _amount) external override {
-        if (_amount !=0) {_requireNoUnderCollateralizedTroves();}
+        _requireNonZeroAmount(_amount);
+        require(deposits[msg.sender].initialValue > 0 , "StabilityPool: User must have a non-zero deposit");
+        require(_amount <= deposits[msg.sender].initialValue, "Amount is greater than deposit value");
+
+        if (_amount !=0) 
+        {_requireNoUnderCollateralizedTroves();}
+
         uint initialDeposit = deposits[msg.sender].initialValue;
-        _requireUserHasDeposit(initialDeposit);
 
         ICommunityIssuance communityIssuanceCached = communityIssuance;
 
@@ -866,10 +871,6 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         }
     }
 
-    function _requireUserHasDeposit(uint _initialDeposit) internal pure {
-        require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');
-    }
-
     function _requireNonZeroAmount(uint _amount) internal pure {
         require(_amount > 0, 'StabilityPool: Amount must be non-zero');
     }
```

### [L&#x2011;04]   Missing check to ensure distribution period is not set as zero
The function updateDistributionPeriod() is used to update the distribution period. If we consider the distribution period could be set to 0 seconds, it can lead to undesired behaviour. A simple check can prevent this issue to be happened. It is recommended.

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

120    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121        distributionPeriod = _newDistributionPeriod;
122    }
``` 

### Proof of Concept:
Add the zero value check.

``` solidity
    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
+       require(_newDistributionPeriod != 0, "Inavalid time period");
        distributionPeriod = _newDistributionPeriod;
    }
``` 

### [L&#x2011;05]  Critical Addresses Changes should adopt two-step address change procedure
All governance functions would not be callable if governance address is set incorrectly
Observerve the update Governancce function….....Note the governance is updated directly… if old governance has provided incorrect address for new governance then governance address is updated incorrectly. 

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

146    function updateGovernance(address _newGovernanceAddress) external {
---

153    function updateGuardian(address _newGuardianAddress) external {
---

160    function upgradeProtocol(
161        address _newTroveManagerAddress,
162        address _newStabilityPoolAddress,
163        address _newBorrowerOperationsAddress
164    ) external {
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L153
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L160-L164

### Recommended mitigation steps:
Implement two step address change procedure such that current owner authority(Governance/Guardian, depending on function) can set a pending address(like pending governance address, guardian address, etc) and then the new address (like new governance address, new guardian address, etc) needs to accept in order to move from pending to confirmed state.

| [L&#x2011;06] | Used Hardcoded address causes no future updates and changes | 4 |
In case the addresses which are hardcoded needs to change due to reasons such as updating their versions in the future then in that case addresses coded as constants cannot be updated, so it is recommended to add the update function to update these hardcoded addresses considering future impacts, this update function should have the onlyOwner modifier, so only owner can update it in future.

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24    address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25    ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
26        ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27    IAaveProtocolDataProvider public constant DATA_PROVIDER =
28        IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29    IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29

### Non-Critical Issues
### [N&#x2011;01]  checkContract( ) dependencies can be used instead of require( )
checkContract( ) dependency can also be used for _treasuryAddress. It will work same as explicitely mentioned in require error handling.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

93        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");

``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L93

### Proof of Concept:
checkContract( ) dependency can be used similarly it is used to check other addresses.  it will save some gas as compared to explicit require error handling and the code wont be repeated. 

```diff
diff --git a/ActivePool.sol.orig b/ActivePool.sol
index 753fcd0..119d52e 100644
--- a/ActivePool.sol.orig
+++ b/ActivePool.sol
@@ -90,7 +90,7 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         checkContract(_stabilityPoolAddress);
         checkContract(_defaultPoolAddress);
         checkContract(_collSurplusPoolAddress);
-        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
+        checkContract(_treasuryAddress);
         checkContract(_lqtyStakingAddress);
 
         collateralConfigAddress = _collateralConfigAddress;
```


### [N&#x2011;02]  pauseMinting/unPauseMinting functions should emit event to notify users
The pauseMinting and unPauseMinting function are used to pause and unpause the minting. When these functions will be used, it is recommended to emit the events to notify the users about this sudden change and to avoid panick.

```solidity
File: Ethos-Core/contracts/ActivePool.sol

133    function pauseMinting() external {
134        require(
135            msg.sender == guardianAddress || msg.sender == governanceAddress,
136            "LUSD: Caller is not guardian or governance"
137        );
138        mintingPaused = true;
139    }
140
141    function unpauseMinting() external {
142       _requireCallerIsGovernance();
143        mintingPaused = false;
144    }
``` 

### [N&#x2011;03]  Function writing that does not comply with the Solidity Style Guide
The style and order of functions helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Solidity style guide:
https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N&#x2011;04]  Do not use floting solidity compiler version. Lock the solidity compiler version
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
There are 4 instances of this issue.

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

3     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3

### [N&#x2011;05]   Events for critical change is missing
To record the distribution period parameter for off-chain monitoring and transparency reasons, please consider emitting an event after the updateDistributionPeriod() function:

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

120    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
121       distributionPeriod = _newDistributionPeriod;
122    }
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122

### Proof of Concept:
UpdateDistributionPeriod event is added in contract.

```diff
diff --git a/CommunityIssuance.sol.orig b/CommunityIssuance.sol
index c79adfe..d6d415c 100644
--- a/CommunityIssuance.sol.orig
+++ b/CommunityIssuance.sol
@@ -51,6 +51,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     event LogRewardPerSecond(uint _rewardPerSecond);
     event StabilityPoolAddressSet(address _stabilityPoolAddress);
     event TotalOATHIssuedUpdated(uint _totalOATHIssued);
+    event UpdateDistributionPeriod(uint __newDistributionPeriod);
 
     // --- Functions ---
 
@@ -118,6 +119,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
 
     // Owner-only function to update the distribution period
     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
+        emit UpdateDistributionPeriod(_newDistributionPeriod);
         distributionPeriod = _newDistributionPeriod;
     }
```

### [N&#x2011;06]   Use scientific notation rather than exponentiation
While the compiler knows to optimize away the exponentiation(**), it is good coding practice to use idioms that do not require compiler optimization, if they exist. 
for example, 1e18 = 10**18
There is 2 instance of this issue:

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40    uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
---

125        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
``` 
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L40
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L125