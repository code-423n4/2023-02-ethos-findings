### Table of contents
- [FINDINGS](#findings)
- [Tighly pack storage variables/optimize the order of variable declaration(Total Gas saved: 16K gas )](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declarationtotal-gas-saved-16k-gas-)
  - [LUSDToken.sol: Pack bool mintingPaused with address troveManagerAddress(Saves 1 SLOT: 2k gas)](#lusdtokensol-pack-bool-mintingpaused-with-address-trovemanageraddresssaves-1-slot-2k-gas)
  - [ReaperVaultV2.sol: pack treasury with emergencyShutdown and lastReport(4k gas)](#reapervaultv2sol-pack-treasury-with-emergencyshutdown-and-lastreport4k-gas)
  - [Save 1 SLOT(2k gas): Pack lastFeeOperationTime with owner](#save-1-slot2k-gas-pack-lastfeeoperationtime-with-owner)
  - [Save 2 SLOTS(4k gas): see description](#save-2-slots4k-gas-see-description)
  - [Save 2 SLOTS(4k gas): Pack lastHarvestTimestamp and upgradeProposalTime](#save-2-slots4k-gas-pack-lastharvesttimestamp-and-upgradeproposaltime)
- [Pack structs by putting data types that fit together next to each other( Saves: 2K gas)](#pack-structs-by-putting-data-types-that-fit-together-next-to-each-other-saves-2k-gas)
  - [ReaperVaultV2.sol: Pack activation together with lastReport(Save 1 SLOT: 2k gas)](#reapervaultv2sol-pack-activation-together-with-lastreportsave-1-slot-2k-gas)
- [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
  - [Emit `_newTvlCap` instead of `tvlCap`](#emit-_newtvlcap-instead-of-tvlcap)
- [IF's/require() statements that check input arguments should be at the top of the function](#ifsrequire-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [reorder the require statement to have cheaper one before the gas consuming one](#reorder-the-require-statement-to-have-cheaper-one-before-the-gas-consuming-one)
  - [cheaper to check function argument against a constant as compared to checking it against a storaga variable](#cheaper-to-check-function-argument-against-a-constant-as-compared-to-checking-it-against-a-storaga-variable)
  - [reorder the require statements to have the cheaper one before the gas consuming one](#reorder-the-require-statements-to-have-the-cheaper-one-before-the-gas-consuming-one)
  - [Reorder the checks to avoid wasting gas checking computing external functionality](#reorder-the-checks-to-avoid-wasting-gas-checking-computing-external-functionality)
  - [reorder the require statement to have the cheaper one come first](#reorder-the-require-statement-to-have-the-cheaper-one-come-first)
  - [Move the cheaper require check on top](#move-the-cheaper-require-check-on-top)
  - [Move the require statement at the beginning of the function](#move-the-require-statement-at-the-beginning-of-the-function)
  - [Reorder the require statement here to have the cheaper one before the gas consuming one](#reorder-the-require-statement-here-to-have-the-cheaper-one-before-the-gas-consuming-one)
  - [The result of a function call should be cached rather than re-calling the function](#the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function)
  - [ReaperVaultV2.sol.\_deposit(): Results of totalSupply() should be cached(saves 1 call on sad path)](#reapervaultv2sol_deposit-results-of-totalsupply-should-be-cachedsaves-1-call-on-sad-path)
  - [ReaperVaultERC4626.sol.convertToShares(): Results of totalSupply() and \_freeFunds() should be cached](#reapervaulterc4626solconverttoshares-results-of-totalsupply-and-_freefunds-should-be-cached)
  - [ReaperVaultERC4626.sol.convertToAssets(): Results of totalSupply() should be cached](#reapervaulterc4626solconverttoassets-results-of-totalsupply-should-be-cached)
  - [ReaperVaultERC4626.sol.maxDeposit(): balance() should be cached](#reapervaulterc4626solmaxdeposit-balance-should-be-cached)
  - [ReaperVaultERC4626.sol.maxMint(): balance() should be cached](#reapervaulterc4626solmaxmint-balance-should-be-cached)
  - [ReaperVaultERC4626.sol.previewMint(): totalSupply() should be cached](#reapervaulterc4626solpreviewmint-totalsupply-should-be-cached)
  - [ReaperVaultERC4626.sol.previewWithdraw(): results of totalSupply() and \_freeFunds() should be cached](#reapervaulterc4626solpreviewwithdraw-results-of-totalsupply-and-_freefunds-should-be-cached)
- [Internal/Private functions only called once can be inlined to save gas](#internalprivate-functions-only-called-once-can-be-inlined-to-save-gas)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [TroveManager.sol.updateDebtAndCollAndStakesPostRedemption(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersolupdatedebtandcollandstakespostredemption-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.\_getCurrentTroveAmounts(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersol_getcurrenttroveamounts-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.\_applyPendingRewards(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersol_applypendingrewards-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.\_updateTroveRewardSnapshots(): rewardSnapshots\[\_borrower\]\[\_collateral\], L\_Collateral\[\_collateral\] and L\_LUSDDebt\[\_collateral\] should be cached](#trovemanagersol_updatetroverewardsnapshots-rewardsnapshots_borrower_collateral-l_collateral_collateral-and-l_lusddebt_collateral-should-be-cached)
  - [TroveManager.sol.getPendingCollateralReward(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersolgetpendingcollateralreward-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.getPendingLUSDDebtReward(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersolgetpendinglusddebtreward-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.getEntireDebtAndColl(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersolgetentiredebtandcoll-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [TroveManager.sol.\_removeStake(): Troves\[\_borrower\]\[\_collateral\] should be cached in local storage](#trovemanagersol_removestake-troves_borrower_collateral-should-be-cached-in-local-storage)
  - [Cache Troves\[\_borrower\]\[\_collateral\] and totalStakes\[\_collateral\] in local storage](#cache-troves_borrower_collateral-and-totalstakes_collateral-in-local-storage)
  - [ActivePool.sol.sendCollateral(): collAmount\[\_collateral\] should be cached in local storage](#activepoolsolsendcollateral-collamount_collateral-should-be-cached-in-local-storage)
  - [ActivePool.sol.increaseLUSDDebt(): LUSDDebt\[\_collateral\] should be cached in local storage](#activepoolsolincreaselusddebt-lusddebt_collateral-should-be-cached-in-local-storage)
  - [ReaperVaultV2.sol.availableCapital(): strategies\[stratAddr\] should be cached in local storage](#reapervaultv2solavailablecapital-strategiesstrataddr-should-be-cached-in-local-storage)
- [Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate](#multiple-address-mappings-can-be-combined-into-a-single-mapping-of-an-address-to-a-struct-where-appropriate)
- [Using storage instead of memory for structs/arrays saves gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas)
- [Avoid contract existence checks by using solidity version 0.8.10 or later](#avoid-contract-existence-checks-by-using-solidity-version-0810-or-later)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [Splitting require() statements that use \&\& saves gas - (saves 8 gas per \&\&)](#splitting-require-statements-that-use--saves-gas---saves-8-gas-per-)
- [Use the cached value in the following to save gas.](#use-the-cached-value-in-the-following-to-save-gas)
- [Caching global variables is expensive than using the variable itself](#caching-global-variables-is-expensive-than-using-the-variable-itself)
  - [Don't cache msg.sender in the following](#dont-cache-msgsender-in-the-following)
  - [Don't cache msg.sender in the following](#dont-cache-msgsender-in-the-following-1)
- [Use a more recent version of solidity](#use-a-more-recent-version-of-solidity)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

## Tighly pack storage variables/optimize the order of variable declaration(Total Gas saved: 16K gas )

Here, the storage variables can be tightly packed by putting data type that can fit together next to each other.
Some packing is only possible as the variables involved are timestamps and thus we can reduce their size from uint256 to uint64 safely

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L37-L71
### LUSDToken.sol: Pack bool mintingPaused with address troveManagerAddress(Saves 1 SLOT: 2k gas)
```solidity
File: /Ethos-Core/contracts/LUSDToken.sol
37:    bool public mintingPaused = false;

51:    bytes32 private immutable _HASHED_NAME;
52:    bytes32 private immutable _HASHED_VERSION;
    
54:    mapping (address => uint256) private _nonces;

67:    address public troveManagerAddress;
```

```diff
diff --git a/Ethos-Core/contracts/LUSDToken.sol b/Ethos-Core/contracts/LUSDToken.sol
index 9c5424d..49f2194 100644
--- a/Ethos-Core/contracts/LUSDToken.sol
+++ b/Ethos-Core/contracts/LUSDToken.sol
@@ -34,7 +34,7 @@ contract LUSDToken is CheckContract, ILUSDToken {
     string constant internal _VERSION = "1";
     uint8 constant internal _DECIMALS = 18;

-    bool public mintingPaused = false;
+

     // --- Data for EIP2612 ---

@@ -62,6 +62,8 @@ contract LUSDToken is CheckContract, ILUSDToken {
     mapping (address => bool) public troveManagers;
     mapping (address => bool) public stabilityPools;
     mapping (address => bool) public borrowerOperations;
+    bool public mintingPaused = false;
     // simple address variables track current version that can mint (in addition to burning)
     // this design makes it so that only the latest version can open troves
     address public troveManagerAddress;
```


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L46-L78
### ReaperVaultV2.sol: pack treasury with emergencyShutdown and lastReport(4k gas)
**lastReport being a timestamp we can get away with using uint64 - should be safe for around 530 years(Following this direction we can save 2 SLOTS : 4k gas)**

**If not willing to to reduce the timestamp variable we can pack `address  treasury and bool  emergencyShutdown` saving 1 SLOT: 2k gas
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
46:    uint256 public lastReport; 
    
48:    uint256 public immutable constructionTime;
49:    bool public emergencyShutdown;

57:    uint256 public lockedProfit; 

78:    address public treasury;
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..e462619 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -43,10 +43,12 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont

     uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
-    uint256 public lastReport; // block.timestamp of last report from any strategy
+    uint64 public lastReport; // block.timestamp of last report from any strategy
+    bool public emergencyShutdown;
+    address public treasury; // address to whom performance fee is remitted in the form of vault shares

     uint256 public immutable constructionTime;
-    bool public emergencyShutdown;
+

     // The token the vault accepts and looks to maximize.
     IERC20Metadata public immutable token;
@@ -75,7 +77,6 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
     bytes32 public constant ADMIN = keccak256("ADMIN");

-    address public treasury; // address to whom performance fee is remitted in the form of vault shares

```


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L21-L66
### Save 1 SLOT(2k gas): Pack lastFeeOperationTime with owner
**lastFeeOperationTime is a  timestamps thus we can use uint64 which would be safe for around 532 years**
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
21:    address public owner;

65:    // The timestamp of the latest fee operation (redemption or new LUSD issuance)
66:    uint public lastFeeOperationTime;
```

```diff
diff --git a/Ethos-Core/contracts/TroveManager.sol b/Ethos-Core/contracts/TroveManager.sol
index 6f587d9..38da844 100644
--- a/Ethos-Core/contracts/TroveManager.sol
+++ b/Ethos-Core/contracts/TroveManager.sol
@@ -20,6 +20,9 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager

     address public owner;

+        // The timestamp of the latest fee operation (redemption or new LUSD issuance)
+    uint64 public lastFeeOperationTime;
+
     // --- Connected contract declarations ---

     address public borrowerOperationsAddress;
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21-L46
### Save 2 SLOTS(4k gas): see description
**lastDistributionTime and lastIssuanceTimestamp are timestamps thus we can use uint64 which would be safe for around 532 years**
```solidity
File: /Ethos-Core/contracts/LQTY/CommunityIssuance.sol
21:    bool public initialized = false;

37:    IERC20 public OathToken;

39:    address public stabilityPoolAddress;

41:    uint public totalOATHIssued;
42:    uint public lastDistributionTime;
43:    uint public distributionPeriod;
44:    uint public rewardPerSecond;

46:    uint public lastIssuanceTimestamp;
```

```diff
diff --git a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
index c79adfe..e7f21ff 100644
--- a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
+++ b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
@@ -19,6 +19,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     string constant public NAME = "CommunityIssuance";

     bool public initialized = false;
+    uint64 public lastDistributionTime;

    /* The issuance factor F determines the curvature of the issuance curve.
     *
@@ -37,13 +38,14 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     IERC20 public OathToken;

     address public stabilityPoolAddress;
+    uint64 public lastIssuanceTimestamp;

     uint public totalOATHIssued;
-    uint public lastDistributionTime;
+
     uint public distributionPeriod;
     uint public rewardPerSecond;

-    uint public lastIssuanceTimestamp;
+

```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L28-L58
### Save 2 SLOTS(4k gas): Pack lastHarvestTimestamp and upgradeProposalTime
**lastHarvestTimestamp and upgradeProposalTime are timestamps thus we can use uint64 which would be safe for around 532 years**
```solidity
File: /Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
28:    address public want;

30:    bool public emergencyExit;
31:    uint256 public lastHarvestTimestamp;

33:    uint256 public upgradeProposalTime;

58:    address public vault;
```

```diff
diff --git a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
index db020f2..a824d97 100644
--- a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
+++ b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
@@ -28,9 +28,9 @@ abstract contract ReaperBaseStrategyv4 is
     address public want;

     bool public emergencyExit;
-    uint256 public lastHarvestTimestamp;
+    uint64 public lastHarvestTimestamp;

-    uint256 public upgradeProposalTime;
+    uint64 public upgradeProposalTime;

```

## Pack structs by putting data types that fit together next to each other( Saves: 2K gas)

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L25-L33
### ReaperVaultV2.sol: Pack activation together with lastReport(Save 1 SLOT: 2k gas)
**activation and lastReport are timestamps thus changing them to uint64 should be safe for around 532 years**
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
25:    struct StrategyParams {
26:        uint256 activation; // Activation block.timestamp
27:        uint256 feeBPS; // Performance fee taken from profit, in BPS
28:        uint256 allocBPS; // Allocation in BPS of vault's total assets
29:        uint256 allocated; // Amount of capital allocated to this strategy
30:        uint256 gains; // Total returns that Strategy has realized for Vault
31:        uint256 losses; // Total losses that Strategy has realized for Vault
32:        uint256 lastReport; // block.timestamp of the last time a report occured
33:    }
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..44f84fb 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -23,13 +23,14 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     using SafeERC20 for IERC20Metadata;

     struct StrategyParams {
-        uint256 activation; // Activation block.timestamp
+        uint64 activation; // Activation block.timestamp
+        uint64 lastReport; // block.timestamp of the last time a report occured
         uint256 feeBPS; // Performance fee taken from profit, in BPS
         uint256 allocBPS; // Allocation in BPS of vault's total assets
         uint256 allocated; // Amount of capital allocated to this strategy
         uint256 gains; // Total returns that Strategy has realized for Vault
         uint256 losses; // Total losses that Strategy has realized for Vault
-        uint256 lastReport; // block.timestamp of the last time a report occured
+
     }
```



## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L578-L582
### Emit `_newTvlCap` instead of `tvlCap`
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
578:    function updateTvlCap(uint256 _newTvlCap) public {
579:        _atLeastRole(ADMIN);
580:        tvlCap = _newTvlCap;
581:        emit TvlCapUpdated(tvlCap);
582:    }
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..c12e679 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -578,7 +578,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     function updateTvlCap(uint256 _newTvlCap) public {
         _atLeastRole(ADMIN);
         tvlCap = _newTvlCap;
-        emit TvlCapUpdated(tvlCap);
+        emit TvlCapUpdated(_newTvlCap);
     }
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L575-L579
```solidity
File: /Ethos-Core/contracts/StabilityPool.sol
575:        if (newProductFactor == 0) {
576:            currentEpoch = currentEpochCached.add(1);
577:            emit EpochUpdated(currentEpoch);
578:            currentScale = 0;
579:            emit ScaleUpdated(currentScale);
```
As currentScale is being assigned value 0, we should just emit the 0 here

```diff
diff --git a/Ethos-Core/contracts/StabilityPool.sol b/Ethos-Core/contracts/StabilityPool.sol
index db163b7..62c0042 100644
--- a/Ethos-Core/contracts/StabilityPool.sol
+++ b/Ethos-Core/contracts/StabilityPool.sol
@@ -576,7 +576,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
             currentEpoch = currentEpochCached.add(1);
             emit EpochUpdated(currentEpoch);
             currentScale = 0;
-            emit ScaleUpdated(currentScale);
+            emit ScaleUpdated(0);
             newP = DECIMAL_PRECISION;

```


##  IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### reorder the require statement to have cheaper one before the gas consuming one
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L46-L54
```solidity
File: /Ethos-Core/contracts/CollateralConfig.sol
46:    function initialize(
47:        address[] calldata _collaterals,
48:        uint256[] calldata _MCRs,
49:        uint256[] calldata _CCRs
50:    ) external override onlyOwner {
51:        require(!initialized, "Can only initialize once");
52:        require(_collaterals.length != 0, "At least one collateral required");
53:        require(_MCRs.length == _collaterals.length, "Array lengths must match");
54:        require(_CCRs.length == _collaterals.length, "Array lenghts must match");
```

The first check ` require(!initialized, "Can only initialize once");` involves reading from the state as `initialized` is a storage variable which is quite expensive. As we would also revert on the other conditions which involves reading function arguments(cheaper) we should reorder the checks as follows

```diff
diff --git a/Ethos-Core/contracts/CollateralConfig.sol b/Ethos-Core/contracts/CollateralConfig.sol
index d524f23..1458836 100644
--- a/Ethos-Core/contracts/CollateralConfig.sol
+++ b/Ethos-Core/contracts/CollateralConfig.sol
@@ -48,10 +48,10 @@ contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
         uint256[] calldata _MCRs,
         uint256[] calldata _CCRs
     ) external override onlyOwner {
-        require(!initialized, "Can only initialize once");
         require(_collaterals.length != 0, "At least one collateral required");
         require(_MCRs.length == _collaterals.length, "Array lengths must match");
         require(_CCRs.length == _collaterals.length, "Array lenghts must match");
+        require(!initialized, "Can only initialize once");

         for(uint256 i = 0; i < _collaterals.length; i++) {
             address collateral = _collaterals[i];
```

### cheaper to check function argument against a constant as compared to checking it against a storaga variable
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L85-L100
```solidity
File: /Ethos-Core/contracts/CollateralConfig.sol
85:    function updateCollateralRatios(
86:        address _collateral,
87:        uint256 _MCR,
88:        uint256 _CCR
89:    ) external onlyOwner checkCollateral(_collateral) {
90:        Config storage config = collateralConfig[_collateral];
91:        require(_MCR <= config.MCR, "Can only walk down the MCR");
92:        require(_CCR <= config.CCR, "Can only walk down the CCR");

94:        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
95:        config.MCR = _MCR;
96:
97:        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
98:        config.CCR = _CCR;
99:        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
100:    }
```
In the above, we have two types of checks, one that involves comparing a function argument to a storage variable `config` and another that compares function argument to a constant. It's obviously cheaper to read a constant as compared to the storage one, thus should have checks for constant come before the one for storage

We can also save alot of gas if we revert before running the line ` Config storage config = collateralConfig[_collateral];` as it involves an SSTORE

```diff
diff --git a/Ethos-Core/contracts/CollateralConfig.sol b/Ethos-Core/contracts/CollateralConfig.sol
index d524f23..f71570d 100644
--- a/Ethos-Core/contracts/CollateralConfig.sol
+++ b/Ethos-Core/contracts/CollateralConfig.sol
@@ -87,15 +87,17 @@ contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
         uint256 _MCR,
         uint256 _CCR
     ) external onlyOwner checkCollateral(_collateral) {
+        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
+        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
+
         Config storage config = collateralConfig[_collateral];
+
         require(_MCR <= config.MCR, "Can only walk down the MCR");
         require(_CCR <= config.CCR, "Can only walk down the CCR");

-        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
         config.MCR = _MCR;
-
-        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
         config.CCR = _CCR;
+
         emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
     }

```

### reorder the require statements to have the cheaper one before the gas consuming one
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L71-L94
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
71:    function setAddresses(

78:        address _treasuryAddress,

81:    )
82:        external
83:        onlyOwner
84:    {
85:        require(!addressesSet, "Can call setAddresses only once");


87:        checkContract(_collateralConfigAddress);
88:        checkContract(_borrowerOperationsAddress);
89:        checkContract(_troveManagerAddress);
90:        checkContract(_stabilityPoolAddress);
91:        checkContract(_defaultPoolAddress);
92:        checkContract(_collSurplusPoolAddress);
93:        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
94:        checkContract(_lqtyStakingAddress);
```

The check on Line 93 would cause our function to revert and since it involves a check for a function parameter, it would be cheaper to have it come before any other operation. In this case we have another check that involves reading from storage which is expensive. Fail early and cheaply

```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..a6903af 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -82,6 +82,8 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         external
         onlyOwner
     {
+        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
+
         require(!addressesSet, "Can call setAddresses only once");

         checkContract(_collateralConfigAddress);
@@ -90,7 +92,6 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         checkContract(_stabilityPoolAddress);
         checkContract(_defaultPoolAddress);
         checkContract(_collSurplusPoolAddress);
-        require(_treasuryAddress != address(0), "Treasury cannot be 0 address");
         checkContract(_lqtyStakingAddress);

         collateralConfigAddress = _collateralConfigAddress;
```

### Reorder the checks to avoid wasting gas checking computing external functionality
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L125-L130
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
125:    function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
126:        _requireValidCollateralAddress(_collateral);
127:        require(_bps <= 10_000, "Invalid BPS value");
128:        yieldingPercentage[_collateral] = _bps;
129:        emit YieldingPercentageUpdated(_collateral, _bps);
130:    }
```

If we end up reverting on the Line `require(_bps <= 10_000, "Invalid BPS value");` we would end up wasting alot of gas on the line `_requireValidCollateralAddress(_collateral);` . The later is a function call and the function has the following implementation
```solidity
    function _requireValidCollateralAddress(address _collateral) internal view {
        require(
            ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),
            "Invalid collateral address"
        );
    }
```
Note in the above function, the require statement involves an external function call which is pretty expensive.
Reccommend to implement the following changes so that incase of an early revert on the `function parameter check` we would not waste gas

```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..d83ee3a 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -123,8 +123,8 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
     }

     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {
+         require(_bps <= 10_000, "Invalid BPS value");
         _requireValidCollateralAddress(_collateral);
-        require(_bps <= 10_000, "Invalid BPS value");
         yieldingPercentage[_collateral] = _bps;
         emit YieldingPercentageUpdated(_collateral, _bps);
     }
```

### reorder the require statement to have the cheaper one come first
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L156
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
144:    function addStrategy(
145:        address _strategy,
146:        uint256 _feeBPS,
147:        uint256 _allocBPS
148:    ) external {
149:        _atLeastRole(DEFAULT_ADMIN_ROLE);
150:        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
151:        require(_strategy != address(0), "Invalid strategy address");
152:        require(strategies[_strategy].activation == 0, "Strategy already added");
153:        require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
154:        require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
155:        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
156:        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
```
Refactor the code as follows to have the cheaper check first

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..48316b9 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -146,13 +146,15 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         uint256 _feeBPS,
         uint256 _allocBPS
     ) external {
+        require(_strategy != address(0), "Invalid strategy address");
+        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
         _atLeastRole(DEFAULT_ADMIN_ROLE);
         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
-        require(_strategy != address(0), "Invalid strategy address");
+
         require(strategies[_strategy].activation == 0, "Strategy already added");
         require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
         require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
-        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
+
         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

```

### Move the cheaper require check on top
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
178:    function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
179:        _atLeastRole(ADMIN);
180:        require(strategies[_strategy].activation != 0, "Invalid strategy address");
181:        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
182:        strategies[_strategy].feeBPS = _feeBPS;
183:        emit StrategyFeeBPSUpdated(_strategy, _feeBPS);
184:    }
```
The require on Line 181 involves checking a function argument which is cheaper compared to it's storage counterpart which is being checked on line 180. Refactor the code as follows to have the cheaper check first
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8532ded 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -176,9 +176,10 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      * @param _feeBPS The new performance fee in basis points.
      */
     function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
+        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
         _atLeastRole(ADMIN);
         require(strategies[_strategy].activation != 0, "Invalid strategy address");
-        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
319:    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
320:        _atLeastRole(DEPOSITOR);
321:        require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
322:        require(_amount != 0, "Invalid amount");
```
The require on Line 322 involves checking a function argument which is cheaper compared to it's storage counterpart which is being checked on line 321. Refactor the code as follows to have the cheaper check first

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..ab8c6c3 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -317,9 +317,10 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     // Internal helper function to deposit {_amount} of assets and mint corresponding
     // shares to {_receiver}. Returns the number of shares that were minted.
     function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {
+        require(_amount != 0, "Invalid amount");
         _atLeastRole(DEPOSITOR);
         require(!emergencyShutdown, "Cannot deposit during emergency shutdown");
-        require(_amount != 0, "Invalid amount");
+
```


https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L88
### Move the require statement at the beginning of the function
```solidity
File: /Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
63:    function __ReaperBaseStrategy_init(
64:        address _vault,
65:        address _want,
66:        address[] memory _strategists,
67:        address[] memory _multisigRoles
68:    ) internal onlyInitializing {
69:        __UUPSUpgradeable_init();
70:        __AccessControlEnumerable_init();

72:        vault = _vault;
73:        want = _want;
74:        IERC20Upgradeable(want).safeApprove(vault, type(uint256).max);

76:        uint256 numStrategists = _strategists.length;
77:        for (uint256 i = 0; i < numStrategists; i = i.uncheckedInc()) {
78:            _grantRole(STRATEGIST, _strategists[i]);
79:        }

81:        require(_multisigRoles.length == 3, "Invalid number of multisig roles");
```
As we might revert if a function argument check does not succeed, it is better to check the argument first before performing any other operations as we might waste gas doing this operations then revert on failure to meet the conditions required for the function argument
```diff
diff --git a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
index db020f2..7c3c5a8 100644
--- a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
+++ b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
@@ -68,6 +68,8 @@ abstract contract ReaperBaseStrategyv4 is
     ) internal onlyInitializing {
         __UUPSUpgradeable_init();
         __AccessControlEnumerable_init();
+        require(_multisigRoles.length == 3, "Invalid number of multisig roles");
+

         vault = _vault;
         want = _want;
@@ -78,7 +80,6 @@ abstract contract ReaperBaseStrategyv4 is
             _grantRole(STRATEGIST, _strategists[i]);
         }

-        require(_multisigRoles.length == 3, "Invalid number of multisig roles")
```

### Reorder the require statement here to have the cheaper one before the gas consuming one
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103
```solidity
File: /Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
95:    function withdraw(uint256 _amount) external override returns (uint256 loss) {
96:        require(msg.sender == vault, "Only vault can withdraw");
97:        require(_amount != 0, "Amount cannot be zero");
98:        require(_amount <= balanceOf(), "Ammount must be less than balance");
```

Cheaper checks should come before the more expensive ones.
```diff
diff --git a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
index db020f2..71504f1 100644
--- a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
+++ b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
@@ -93,8 +93,8 @@ abstract contract ReaperBaseStrategyv4 is
      *      is deducted up-front.
      */
     function withdraw(uint256 _amount) external override returns (uint256 loss) {
-        require(msg.sender == vault, "Only vault can withdraw");
         require(_amount != 0, "Amount cannot be zero");
+        require(msg.sender == vault, "Only vault can withdraw");
         require(_amount <= balanceOf(), "Ammount must be less than balance");

```



### The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L319-L338
### ReaperVaultV2.sol.\_deposit(): Results of totalSupply() should be cached(saves 1 call on sad path)
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
319:    function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {

331:        if (totalSupply() == 0) {
332:            shares = _amount;
333:        } else {
334:            shares = (_amount * totalSupply()) / freeFunds; // use "freeFunds" instead of "pool"
335:        }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51-L54
### ReaperVaultERC4626.sol.convertToShares(): Results of totalSupply() and \_freeFunds() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
51:    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
52:        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
53:        return (assets * totalSupply()) / _freeFunds();
54:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66-L69
### ReaperVaultERC4626.sol.convertToAssets(): Results of totalSupply() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
66:    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
67:        if (totalSupply() == 0) return shares;
68:        return (shares * _freeFunds()) / totalSupply();
69:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79-L83
### ReaperVaultERC4626.sol.maxDeposit(): balance() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
79:    function maxDeposit(address) external view override returns (uint256 maxAssets) {
80:        if (emergencyShutdown || balance() >= tvlCap) return 0;
81:        if (tvlCap == type(uint256).max) return type(uint256).max;
82:        return tvlCap - balance();
83:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122-L126
### ReaperVaultERC4626.sol.maxMint(): balance() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
122:    function maxMint(address) external view override returns (uint256 maxShares) {
123:        if (emergencyShutdown || balance() >= tvlCap) return 0;
124:        if (tvlCap == type(uint256).max) return type(uint256).max;
125:        return convertToShares(tvlCap - balance());
126:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L139-L142
### ReaperVaultERC4626.sol.previewMint(): totalSupply() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
139:    function previewMint(uint256 shares) public view override returns (uint256 assets) {
140:        if (totalSupply() == 0) return shares;
141:        assets = roundUpDiv(shares * _freeFunds(), totalSupply());
142:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L183-L186
### ReaperVaultERC4626.sol.previewWithdraw(): results of totalSupply() and \_freeFunds() should be cached
```solidity
File: /Ethos-Vault/contracts/ReaperVaultERC4626.sol
183:    function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {
184:        if (totalSupply() == 0 || _freeFunds() == 0) return 0;
185:       shares = roundUpDiv(assets * totalSupply(), _freeFunds());
186:    }
```


## Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Affected code:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L438-L445
```solidity
File: /Ethos-Core/contracts/BorrowerOperations.sol
438:    function _getCollChange(
439:        uint _collReceived,
440:        uint _requestedCollWithdrawal
441:    )
442:        internal
443:        pure
444:        returns(uint collChange, bool isCollIncrease)
445:    {


455:    function _updateTroveFromAdjustment
456:    (
457:        ITroveManager _troveManager,
458:        address _borrower,
459:        address _collateral,
460:        uint _collChange,
461:        bool _isCollIncrease,
462:        uint _debtChange,
463:        bool _isDebtIncrease
464:    )
465:        internal
466:        returns (uint, uint)
467:    {

476:    function _moveTokensAndCollateralfromAdjustment
477:    (
478:        IActivePool _activePool,
479:        ILUSDToken _lusdToken,
480:        address _borrower,
481:        address _collateral,
482:        uint _collChange,
483:        bool _isCollIncrease,
484:        uint _LUSDChange,
485:        bool _isDebtIncrease,
486:        uint _netDebtChange
487:    )
488:        internal
489:    {


533:    function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

537:    function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

546:    function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

551:    function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

555:    function _requireNotInRecoveryMode(
556:        address _collateral,
557:        uint _price,
558:        uint256 _CCR,
559:        uint256 _collateralDecimals
560:    ) internal view {

567:    function _requireNoCollWithdrawal(uint _collWithdrawal) internal pure {

571:    function _requireValidAdjustmentInCurrentMode 
572:    (
573:        bool _isRecoveryMode,
574:        address _collateral,
575:        uint _collWithdrawal,
576:        bool _isDebtIncrease, 
577:        LocalVariables_adjustTrove memory _vars
578:    ) 
579:        internal 
580:        view 
581:    {

624:    function _requireNewICRisAboveOldICR(uint _newICR, uint _oldICR) internal pure {

636:    function _requireValidLUSDRepayment(uint _currentDebt, uint _debtRepayment) internal pure {

640:    function _requireCallerIsStabilityPool() internal view {

661:    function _getNewNominalICRFromTroveChange

674:    {
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478-L489
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
478:    function _getCappedOffsetVals

582:    function _getTotalsFromLiquidateTrovesSequence_RecoveryMode

671:    function _getTotalsFromLiquidateTrovesSequence_NormalMode

785:    function _getTotalFromBatchLiquidate_RecoveryMode

864:    function _getTotalsFromBatchLiquidate_NormalMode

1213:    function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

1538:    function _requireMoreThanOneTroveInSystem(uint TroveOwnersArrayLength, address _collateral) internal view {
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L320
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
320:    function _requireCallerIsBorrowerOperationsOrDefaultPool() internal view {
```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L962-L980
### TroveManager.sol.updateDebtAndCollAndStakesPostRedemption(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
962:    function updateDebtAndCollAndStakesPostRedemption(

967:    ) external override {
968:        _requireCallerIsRedemptionHelper();
969:        Troves[_borrower][_collateral].debt = _newDebt;//@audit: Initial access
970:        Troves[_borrower][_collateral].coll = _newColl;//@audit: 2nd access
971:        _updateStakeAndTotalStakes(_borrower, _collateral);

973:        emit TroveUpdated(
        
977:            Troves[_borrower][_collateral].stake,//@audit: 3rd access
978:            TroveManagerOperation.redeemCollateral
979:        );
980:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1066-L1074
### TroveManager.sol.\_getCurrentTroveAmounts(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1066:    function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {

1070:	 uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);//@audit: Initial access
1071:        uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);//@audit: 2nd access
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1082-L1108
### TroveManager.sol.\_applyPendingRewards(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1082:    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {

1091:            Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);//@audit: Initial access
1092:            Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);//@audit: 2nd access


1099:            emit TroveUpdated(
1100:                _borrower,
1101:                _collateral,
1102:                Troves[_borrower][_collateral].debt,//@audit: 3rd access
1103:                Troves[_borrower][_collateral].coll,//@audit: 4th access
1104:                Troves[_borrower][_collateral].stake,//@audit: 5th access
1105:                TroveManagerOperation.applyPendingRewards
1106:            );
1107:        }
1108:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1116-L1120
### TroveManager.sol.\_updateTroveRewardSnapshots(): rewardSnapshots\[\_borrower]\[\_collateral], L_Collateral\[\_collateral] and L_LUSDDebt\[\_collateral] should be cached 
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1116:    function _updateTroveRewardSnapshots(address _borrower, address _collateral) internal {
1117:        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];//@audit: Initial access
1118:        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];//@audit: 2nd access
1119:        emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);
1120:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1123-L1135
### TroveManager.sol.getPendingCollateralReward(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1123:    function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1127:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; } //@audit: Initial access

1129:        uint stake = Troves[_borrower][_collateral].stake;//@audit: 2nd access
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1138-L1150
### TroveManager.sol.getPendingLUSDDebtReward(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1138:    function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1142:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }//@audit: Initial access

1144:        uint stake = Troves[_borrower][_collateral].stake;//@audit: 2nd access
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1164-L1181
### TroveManager.sol.getEntireDebtAndColl(): Troves\[\_borrower]\[\_collateral] should be cached in local storage 
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1164:    function getEntireDebtAndColl(

1173:        debt = Troves[_borrower][_collateral].debt;//@audit: initial access
1174:       coll = Troves[_borrower][_collateral].coll;//@audit: 2nd access
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1189-L1193
### TroveManager.sol.\_removeStake(): Troves\[\_borrower]\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1189:    function _removeStake(address _borrower, address _collateral) internal {
1190:        uint stake = Troves[_borrower][_collateral].stake;
1191:        totalStakes[_collateral] = totalStakes[_collateral].sub(stake);
1192:        Troves[_borrower][_collateral].stake = 0;
1193:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1201-L1210
### Cache Troves\[\_borrower]\[\_collateral] and totalStakes\[\_collateral] in local storage
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1201:    function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {
1202:        uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);
1203:        uint oldStake = Troves[_borrower][_collateral].stake;
1204:        Troves[_borrower][_collateral].stake = newStake;

1206:        totalStakes[_collateral] = totalStakes[_collateral].sub(oldStake).add(newStake);
1207:        emit TotalStakesUpdated(_collateral, totalStakes[_collateral]);
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L171-L188
### ActivePool.sol.sendCollateral(): collAmount\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
171:    function sendCollateral(address _collateral, address _account, uint _amount) external override {
    
175:        collAmount[_collateral] = collAmount[_collateral].sub(_amount);
176:        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190-L195
### ActivePool.sol.increaseLUSDDebt(): LUSDDebt\[\_collateral] should be cached in local storage
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
190:    function increaseLUSDDebt(address _collateral, uint _amount) external override {

193:        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
194:        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]);
195:    }
```
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L197-L202

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L204-L212

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L252
### ReaperVaultV2.sol.availableCapital(): strategies[stratAddr] should be cached in local storage
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
225:    function availableCapital() public view returns (int256) {
226:        address stratAddr = msg.sender;
227:        if (totalAllocBPS == 0 || emergencyShutdown) {
228:            return -int256(strategies[stratAddr].allocated);//@audit: 1st access
229:        }

231:        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR; //@audit: 2nd access
232:        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;//@audit: 3rd access
```

## Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L88-L94
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
88:    mapping (address => uint) public totalStakes;
    
91:    mapping (address => uint) public totalStakesSnapshot;

94:    mapping (address => uint) public totalCollateralSnapshot;
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L104-L105
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
104:    mapping (address => uint) public L_Collateral;
105:    mapping (address => uint) public L_LUSDDebt;
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L119-L120
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
119:    mapping (address => uint) public lastCollateralError_Redistribution;
120:    mapping (address => uint) public lastLUSDDebtError_Redistribution;
```


## Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L687
```solidity
File: /Ethos-Core/contracts/StabilityPool.sol
687:        Snapshots memory snapshots = depositSnapshots[_depositor];
```

```diff
diff --git a/Ethos-Core/contracts/StabilityPool.sol b/Ethos-Core/contracts/StabilityPool.sol
index db163b7..5f2a340 100644
--- a/Ethos-Core/contracts/StabilityPool.sol
+++ b/Ethos-Core/contracts/StabilityPool.sol
@@ -684,7 +684,7 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         uint initialDeposit = deposits[_depositor].initialValue;
         if (initialDeposit == 0) {return 0;}

-        Snapshots memory snapshots = depositSnapshots[_depositor];
+        Snapshots storage snapshots = depositSnapshots[_depositor];
         uint LQTYGain = _getLQTYGainFromSnapshots(initialDeposit, snapshots);

```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L722
```solidity
File: /Ethos-Core/contracts/StabilityPool.sol
722:        Snapshots memory snapshots = depositSnapshots[_depositor];
```

## Avoid contract existence checks by using solidity version 0.8.10 or later

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L178-L181
```solidity
File: /Ethos-Core/contracts/BorrowerOperations.sol

//@audit: getCollateralCCR(_collateral)
178:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralMCR(_collateral)
179:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: getCollateralDecimals(_collateral)
180:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: fetchPrice(_collateral)
181:        vars.price = priceFeed.fetchPrice(_collateral);

//@audit: getCollateralCCR(_collateral);
286:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralMCR(_collateral)
287:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: getCollateralDecimals(_collateral)
288:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: fetchPrice(_collateral)
289:        vars.price = priceFeed.fetchPrice(_collateral);

//@audit: getCollateralCCR(_collateral)
381:        uint256 collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralDecimals(_collateral)
382:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: fetchPrice(_collateral)
383:        uint price = priceFeed.fetchPrice(_collateral);

//@audit: getTroveColl(msg.sender, _collateral)
388:        uint coll = troveManagerCached.getTroveColl(msg.sender, _collateral);

//@audit: getTroveDebt(msg.sender, _collateral) 
389:        uint debt = troveManagerCached.getTroveDebt(msg.sender, _collateral);

//@audit: getBorrowingFee(_LUSDAmount)
421:        uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);

//@audit: increaseTroveColl
468:        uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange): 
469: _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange);
470:        uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange): 471:_troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange);

//@audit: getTroveStatus(_borrower, _collateral)
542:        uint status = _troveManager.getTroveStatus(_borrower, _collateral);

//@audit: getTroveStatus(_borrower, _collateral);
547:        uint status = _troveManager.getTroveStatus(_borrower, _collateral);
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L420
```solidity
File: /Ethos-Core/contracts/TroveManager.sol

//@audit: getCollateralDecimals(_collateral)
420:            uint collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralCCR(_collateral) 
521:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralDecimals(_collateral)
522:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralMCR(_collateral)
523:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: fetchPrice(_collateral)
524:        vars.price = priceFeed.fetchPrice(_collateral);

//@audit: getTotalLUSDDeposits();
525:        vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();

//@audit: getCollateralDecimals(_collateral)
596:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralCCR(_collateral)
597:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralMCR(_collateral)
598:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: getLast(_collateral)
606:        vars.user = _sortedTroves.getLast(_collateral);

//@audit: getFirst(_collateral)
607:        address firstUser = _sortedTroves.getFirst(_collateral);

//@audit: getPrev(_collateral, vars.user)
610:            address nextUser = _sortedTroves.getPrev(_collateral, vars.user);

//@audit: getLast(_collateral)
691:            vars.user = sortedTrovesCached.getLast(_collateral);

//@audit: getCollateralDecimals(_collateral)
725:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralCCR(_collateral)
726:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralMCR(_collateral)
727:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: fetchPrice(_collateral)
728:        vars.price = priceFeed.fetchPrice(_collateral);

//@audit: getTotalLUSDDeposits()
729:        vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();

//@audit: getCollateralDecimals(_collateral)
798:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralCCR(_collateral)
799:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralMCR(_collateral)
800:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral);

//@audit: getCollateralDecimals(_collateral)
1048:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralDecimals(_collateral)
1061:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralDecimals(_collateral)
1131:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralDecimals(_collateral)
1146:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateral(_collateral)
1308:        uint activeColl = _activePool.getCollateral(_collateral);

//@audit: getCollateral(_collateral);
1309:        uint liquidatedColl = defaultPool.getCollateral(_collateral);

//@audit: getCollateralDecimals(_collateral)
1362:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);

//@audit: getCollateralCCR(_collateral);
1367:        uint256 collCCR = collateralConfig.getCollateralCCR(_collateral);

//@audit: getCollateralDecimals(_collateral);
1368:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L105
```solidity
File: /Ethos-Core/contracts/ActivePool.sol

//@audit: getAllowedCollaterals()
105:        address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

//@audit: asset()
111:            require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral");

//@audit: isCollateralAllowed(_collateral)
315:          ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral),

//@audit: redemptionHelper()
328:        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
```

## x += y costs more gas than x = x + y for state variables
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
168:        totalAllocBPS += _allocBPS;

194:        totalAllocBPS -= strategies[_strategy].allocBPS;

196:        totalAllocBPS += _allocBPS;

214:        totalAllocBPS -= strategies[_strategy].allocBPS;

396:        totalAllocated -= actualWithdrawn;

445:        totalAllocBPS -= bpsChange;

452:        totalAllocated -= loss;

515:        totalAllocated -= vars.debtPayment;

521:        totalAllocated += vars.credit;
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L235
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
235:            return -int256(stratCurrentAllocation - stratMaxAllocation);
```
The operation `stratCurrentAllocation - stratMaxAllocation` cannot underflow due to the check on [Line 234](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L234) that ensures that the operation would only be performed if `stratCurrentAllocation` is greater than `stratMaxAllocation`

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L244
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
244:            uint256 available = stratMaxAllocation - stratCurrentAllocation;
```
The operation `stratMaxAllocation - stratCurrentAllocation` cannot underflow due to the check on [Line 236](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L236) that ensures that `stratMaxAllocation` is greater than `stratCurrentAllocation` before performing the arithmetic operation

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L245
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
245:            available = Math.min(available, vaultMaxAllocation - vaultCurrentAllocation);
```
The operation `vaultMaxAllocation - vaultCurrentAllocation` cannot underflow due to the check on [Line 240](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L240) that ensures that `vaultMaxAllocation` is greater than `vaultCurrentAllocation` before performing our arithmetic operation

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L384
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
384:                uint256 remaining = value - vaultBalance;
```
The operation `value - vaultBalance` cannot underflow due to the check on [Line 374](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L374) that ensures that if `value` is less than `vaultBalance`, the loop would break out and our operation would never be performed

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L81
```solidity
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
81:            uint256 toReinvest = wantBalance - _debt;
```
The operation `wantBalance - _debt` cannot underflow due to the check on [Line 80](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L80) that ensures that `wantBalance` is greater than `_debt` before performing the arithmetic operation

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L93
```solidity
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
93:            _withdraw(_amountNeeded - wantBal);
```
The operation `_amountNeeded - wantBal` cannot underflow due to the check on [Line 92](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L92) that ensures that `_amountNeeded` is greater than `wantBal` before performing the arithmetic operation

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L100
```solidity
File: /Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
100:            loss = _amountNeeded - liquidatedAmount;
```
The operation `_amountNeeded - liquidatedAmount` cannot underflow due to the check on [Line 99](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L99) that ensures that `_amountNeeded` is greater than `liquidatedAmount` before performing the arithmetic operation



## Splitting require() statements that use && saves gas - (saves 8 gas per &&)

Instead of using the && operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 8 GAS per &&
The gas difference would only be realized if the revert condition is realized(met).
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654
```solidity
File: /Ethos-Core/contracts/BorrowerOperations.sol
653:            require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
654:                "Max fee percentage must be between 0.5% and 100%");
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1539
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
1539:        require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L347-L351
```solidity
File: /Ethos-Core/contracts/LUSDToken.sol
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
```

##  Use the cached value in the following to save gas.

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L171-L188
```solidity
File: /Ethos-Core/contracts/ActivePool.sol
171:    function sendCollateral(address _collateral, address _account, uint _amount) external override {

179:        if (_account == defaultPoolAddress) {
180:            IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);
181:    IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
182:        } else if (_account == collSurplusPoolAddress) {
183:            IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);         184: ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
185:        } else {
186:            IERC20(_collateral).safeTransfer(_account, _amount);
187:        }
188:    }
```
In the above function, The first if condition checks that `_account == defaultPoolAddress` meaning the operations would only be performed if the two are the same, as `defaultPoolAddress` is a storage variable(1 SLOAD = 100 gas) we could replace it's occurence inside the if blocks with the local `_account` variable (1 MLOAD = 3 gas ) since the two are the same. The storage variable is only read in this block thus we can get away with using the local one.

Similar explanation to the above applies to the `collSurplusPoolAddress` variable

```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..735a0dd 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -177,11 +177,11 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
         emit CollateralSent(_collateral, _account, _amount);

         if (_account == defaultPoolAddress) {
-            IERC20(_collateral).safeIncreaseAllowance(defaultPoolAddress, _amount);
-            IDefaultPool(defaultPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+            IERC20(_collateral).safeIncreaseAllowance(_account, _amount);
+            IDefaultPool(_account).pullCollateralFromActivePool(_collateral, _amount);
         } else if (_account == collSurplusPoolAddress) {
-            IERC20(_collateral).safeIncreaseAllowance(collSurplusPoolAddress, _amount);
-            ICollSurplusPool(collSurplusPoolAddress).pullCollateralFromActivePool(_collateral, _amount);
+            IERC20(_collateral).safeIncreaseAllowance(_account, _amount);
+            ICollSurplusPool(_account).pullCollateralFromActivePool(_collateral, _amount);
         } else {
             IERC20(_collateral).safeTransfer(_account, _amount);
         }
```

## Caching global variables is expensive than using the variable itself
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L252
### Don't cache msg.sender in the following
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
225:    function availableCapital() public view returns (int256) {
226:        address stratAddr = msg.sender;
227:        if (totalAllocBPS == 0 || emergencyShutdown) {
228:            return -int256(strategies[stratAddr].allocated);
229:        }

231:        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
232:        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..d6f5d3e 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -223,13 +223,12 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
      * the vault.
      */
     function availableCapital() public view returns (int256) {
-        address stratAddr = msg.sender;
         if (totalAllocBPS == 0 || emergencyShutdown) {
-            return -int256(strategies[stratAddr].allocated);
+            return -int256(strategies[msg.sender].allocated);
         }

-        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
-        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;
+        uint256 stratMaxAllocation = (strategies[msg.sender].allocBPS * balance()) / PERCENT_DIVISOR;
+        uint256 stratCurrentAllocation = strategies[msg.sender].allocated;

```

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L493-L560
### Don't cache msg.sender in the following
```solidity
File: /Ethos-Vault/contracts/ReaperVaultV2.sol
    function report(int256 _roi, uint256 _repayment) external returns (uint256) {
        LocalVariables_report memory vars;
        vars.stratAddr = msg.sender;
        StrategyParams storage strategy = strategies[vars.stratAddr];
        require(strategy.activation != 0, "Unauthorized strategy");


        if (_roi < 0) {
            vars.loss = uint256(-_roi);
            _reportLoss(vars.stratAddr, vars.loss);
        } else if (_roi > 0) {
            vars.gain = uint256(_roi);
            vars.fees = _chargeFees(vars.stratAddr, vars.gain);
            strategy.gains += vars.gain;
        }


        if (vars.credit > vars.freeWantInStrat) {
            token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);
        } else if (vars.credit < vars.freeWantInStrat) {
            token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);
        }

        emit StrategyReported(
            vars.stratAddr,

        );


        if (strategy.allocBPS == 0 || emergencyShutdown) {
            return IStrategy(vars.stratAddr).balanceOf();
        }
```



## Use a more recent version of solidity

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

Most of the in scope code seems to be running versions lower than 0.8

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3
```solidity
File: /Ethos-Core/contracts/TroveManager.sol
3:pragma solidity 0.6.11;
```