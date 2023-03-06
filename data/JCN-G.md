## Summary
Certian optimizations were benchmarked to use as baselines for issues + illustrate average savings. All benchmarks were done via the protocol's tests, i.e. using the following config: `solc version 0.8.11`, `optimizer on` , `200 runs` for `Ethos-Vault` contracts. Instances are illustrated with diffs.

Functions that are not covered in the protocol's tests and/or not benchmarked: gas savings are explained via opcodes. Instances are also illustrated with diffs.

**Note**: Some code snippets may be truncated to save space. Code snippets may also be accompanied by `@audit` tags in comments to aid in explaining the issue.

### Gas Optimizations
| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G-01](#state-variables-can-be-packed-to-use-fewer-storage-slots) | State variables can be packed to use fewer storage slots | 12 | 24000 |
| [G-02](#structs-can-be-packed-to-use-fewer-storage-slots) | Structs can be packed to use fewer storage slots | 4 | 8000 |
| [G-03](#avoid-contract-existence-checks-by-using-solidity-version-0810-or-later) | Avoid contract existence checks by using solidity version 0.8.10 or later | 87 |  8700 |
| [G-04](#multiple-accesses-of-a-mappingarray-should-use-a-storage-pointer) | Multiple accesses of a mapping/array should use a storage pointer | 41 | 1969 |
| [G-05](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 34 |  3400 |
| [G-06](#avoid-emitting-storage-values) | Avoid emitting storage values | 26 | 2600 |
| [G-07](#refactor-code-to-save-1-sstore-and-2-sloads) | Refactor code to save 1 SSTORE and 2 SLOADs | 1 |  337 |
| [G-08](#internal-functions-are-cheaper-to-call-compared-to-public-functions) | `Internal` functions are cheaper to call compared to `public` functions | 13 |  286 |
| [G-09](#x--yx---y-costs-more-gas-than-x--x--yx--x---y-for-state-variables) | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | 13 |  234 |

## State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Gas savings: `12 * 2000 = 24000`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L30-L58

### We are able to pack `emergencyExit`, `lastHarvestTimestamp`, and `vault` into a single storage slot to save 2 SLOTs (~4000 gas).

`lastHarvestTimestamp` is simply a timestamp and therefore can be packed as a `uint40` value without fear of overflowing. See [reference 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [reference 2](https://twitter.com/RareSkills_io/status/1632033211609137156). `emergencyExit` is a bool which occupies 1 byte and `vault` is an address which takes up 20 bytes (`uint160`).

Functions which would benefit from these values occupying the same storage slot by incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)` for every subsequent storage slot access: [ReaperBaseStrategyv4.harvest()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L109-L138), [ReaperBaseStrategyv4.setEmergencyExit()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L156-L160)

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
30:    bool public emergencyExit;
31:    uint256 public lastHarvestTimestamp;
32:
33:    uint256 public upgradeProposalTime;
34:
35:    /**
36:     * Reaper Roles in increasing order of privilege.
37:     * {KEEPER} - Stricly permissioned trustless access for off-chain programs or third party keepers.
38:     * {STRATEGIST} - Role conferred to authors of the strategy, allows for tweaking non-critical params.
39:     * {GUARDIAN} - Multisig requiring 2 signatures for emergency measures such as pausing and panicking.
40:     * {ADMIN}- Multisig requiring 3 signatures for unpausing.
41:     *
42:     * The DEFAULT_ADMIN_ROLE (in-built access control role) will be granted to a multisig requiring 4
43:     * signatures. This role would have upgrading capability, as well as the ability to grant any other
44:     * roles.
45:     *
46:     * Also note that roles are cascading. So any higher privileged role should be able to perform all the functions
47:     * of any lower privileged role.
48:     */
49:    bytes32 public constant KEEPER = keccak256("KEEPER");
50:    bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:    bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:    bytes32 public constant ADMIN = keccak256("ADMIN");
53:
54:    /**
55:     * @dev Reaper contracts:
56:     * {vault} - Address of the vault that controls the strategy's funds.
57:     */
58:    address public vault;
```
```diff
diff --git a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
index db020f2..aa14104 100644
--- a/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
+++ b/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
@@ -27,8 +27,6 @@ abstract contract ReaperBaseStrategyv4 is
     // The token the strategy wants to operate
     address public want;

-    bool public emergencyExit;
-    uint256 public lastHarvestTimestamp;

     uint256 public upgradeProposalTime;

@@ -55,6 +53,8 @@ abstract contract ReaperBaseStrategyv4 is
      * @dev Reaper contracts:
      * {vault} - Address of the vault that controls the strategy's funds.
      */
+    bool public emergencyExit;
+    uint40 public lastHarvestTimestamp;
     address public vault;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L44-L57

### We are able to pack `totalAllocBPS` & `emergencyShutdown` and `lastReport`, `withdrawMaxLoss`, & `lockedProfitDegradation` into 2 storage slots to save 3 SLOTs (~6000 gas).

`lastReport` is simply a timestamp and therefore can be packed as a `uint40` value without fear of overflowing. See [reference 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [reference 2](https://twitter.com/RareSkills_io/status/1632033211609137156). `totalAllocBPS`, `withdrawMaxLoss`, and `lockedProfitDegradation` are asserted to stay within certain bounds. `uint80` is sufficient for `totalAllocBPS`, `withdrawMaxLoss`, and `lockedProfitDegradation` (larger uints could also work). `emergencyShutdown` is bool and therefore only occupies 1 byte.

Below are instances where `totalAllocBPS` is asserted to be `<= PERCENT_DIVISOR`:

[ReaperVaultV2.sol#L144-L156](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L156)

[ReaperVaultV2.sol#L191-L199](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L199)

Below are instances where `withdrawMaxLoss` is asserted to be `<= PERCENT_DIVISOR`:

[ReaperVaultV2.sol#L566-L571](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L566-L571)

Below are instances where `lockedProfitDegradation` is asserted to be `<= DEGRADATION_COEFFICIENT`:

[ReaperVaultV2.sol#L617-L622](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L617-L622)

Functions which would benefit from these values occupying the same storage slot by incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)` for every subsequent storage slot access: [ReaperVaultV2.addStrategy()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L171), [ReaperVaultV2.availableCapital()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L252), [ReaperVaultV2.__calculateLockedProfit()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L418-L425), [ReaperVaultV2.report() (`which calls _calculateLockedProfit()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L493-L560), [ReaperVaultV2._withdraw() (`which calls _freeFunds() which in turn calls _calculateLockedProfit()`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L359-L412)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
40:    uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.
41:    uint256 public constant PERCENT_DIVISOR = 10000;
42:    uint256 public tvlCap;
43:
44:    uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
45:    uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
46:    uint256 public lastReport; // block.timestamp of last report from any strategy
47:
48:    uint256 public immutable constructionTime;
49:    bool public emergencyShutdown;
50:
51:    // The token the vault accepts and looks to maximize.
52:    IERC20Metadata public immutable token;
53:
54:    // Max slippage(loss) allowed when withdrawing, in BPS (0.01%)
55:    uint256 public withdrawMaxLoss = 1;
56:    uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
57:    uint256 public lockedProfit; // how much profit is locked and cant be withdrawn
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..c0b3c61 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -41,19 +41,19 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     uint256 public constant PERCENT_DIVISOR = 10000;
     uint256 public tvlCap;

-    uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
+    bool public emergencyShutdown;
+    uint80 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
     uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
-    uint256 public lastReport; // block.timestamp of last report from any strategy

     uint256 public immutable constructionTime;
-    bool public emergencyShutdown;

     // The token the vault accepts and looks to maximize.
     IERC20Metadata public immutable token;

+    uint40 public lastReport; // block.timestamp of last report from any strategy
     // Max slippage(loss) allowed when withdrawing, in BPS (0.01%)
-    uint256 public withdrawMaxLoss = 1;
-    uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
+    uint80 public withdrawMaxLoss = 1;
+    uint80 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
     uint256 public lockedProfit; // how much profit is locked and cant be withdrawn
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L37-L46

### We are able to pack `lastDistributionTime`, `lastIssuanceTimestamp`, `distributionPeriod`, and `rewardPerSecond` into 1 storage slot to save 3 SLOTs (~6000 gas) *or* we are able to pack `lastDistributionTime`, `lastIssuanceTimestamp`, and `OathToken` into 1 storage slot to save 2 SLOTs (~4000 gas).

`lastDistributionTime` and `lastIssuanceTimestamp` and are simply timestamps and therefore can be packed as `uint40` values without fear of overflowing. See [reference 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [reference 2](https://twitter.com/RareSkills_io/status/1632033211609137156). `distributionPeriod` and `rewardPerSecond` are only updated via priviledged functions and could possibly be declared as `uint80/uint88`. `OathToken` is an address (`uint160`).

Below is the instance where `distributionPeriod` is updated via a priviledged function:

[CommunityIssuance.sol#L120-L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122)

Below is the instance where `rewardPerSecond` is updated via a priviledged function. **Note that `rewardPerSecond` is updated via values determinded by the owner (`amount` & `distributionPeriod`) and timestamps (`lastDistributionTime` and `lastIssuanceTimestamp`) which we have established could be bounded safetly**:

[CommunityIssuance.sol#L101-L112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L112)

Functions which would benefit from these values occupying the same storage slot by incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)` for every subsequent storage slot access: [CommunityIssuance.fund()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117), [CommunityIssuance.issueOath()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L84-L95)

If `distributionPeriod`, and `rewardPerSecond` are found to not be able to utilize `uint80/uint88`, we can still save 2 SLOTs by packing `lastDistributionTime`, `lastIssuanceTimestamp`, and `OathToken` into 1 storage slot and receive gas savings when the [CommunityIssuance.fund()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101-L117) function is called.

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol
37:    IERC20 public OathToken;
38:
39:    address public stabilityPoolAddress;
40:
41:    uint public totalOATHIssued;
42:    uint public lastDistributionTime;
43:    uint public distributionPeriod;
44:    uint public rewardPerSecond;
45:
46:    uint public lastIssuanceTimestamp;
```
**Packing `lastDistributionTime`, `lastIssuanceTimestamp`, `distributionPeriod`, and `rewardPerSecond` into 1 storage slot**
```diff
diff --git a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
index c79adfe..9365655 100644
--- a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
+++ b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
@@ -39,11 +39,11 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
     address public stabilityPoolAddress;

     uint public totalOATHIssued;
-    uint public lastDistributionTime;
-    uint public distributionPeriod;
-    uint public rewardPerSecond;
+    uint40 public lastDistributionTime;
+    uint40 public lastIssuanceTimestamp;
+    uint80 public distributionPeriod;
+    uint80 public rewardPerSecond;

-    uint public lastIssuanceTimestamp;
```
**Packing `lastDistributionTime`, `lastIssuanceTimestamp`, and `OathToken`into 1 storage slot**
```diff
diff --git a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
index c79adfe..9365655 100644
--- a/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
+++ b/Ethos-Core/contracts/LQTY/CommunityIssuance.sol
@@ -34,16 +34,16 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa
-    IERC20 public OathToken;

     address public stabilityPoolAddress;

     uint public totalOATHIssued;
-    uint public lastDistributionTime;
     uint public distributionPeriod;
     uint public rewardPerSecond;

-    uint public lastIssuanceTimestamp;
+    uint40 public lastDistributionTime;
+    uint40 public lastIssuanceTimestamp;
+    IERC20 public OathToken;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L26-L54

### We are able to pack `yieldingPercentageDrift`, `yieldSplitTreasury`, `yieldSplitStaking`, and `lqtyStakingAddress` into 1 storage slot to save 3 SLOTs (~6000 gas) *or* we can pack `yieldSplitTreasury`, `yieldSplitSP`, and `yieldSplitStaking` into 1 storage slot to save 2 SLOTs (~4000 gas).

`yieldingPercentageDrift`, `yieldSplitTreasury`, `yieldSplitSP`, and `yieldSplitStaking` are asserted to stay within certain bounds and could therefore be of type `> uint8, <= uint32` and stay well within those bounds. `lqtyStakingAddress` is an address `uint160`. 

Below is the instance where `yieldingPercentageDrift` is asserted to be `<= 500`:

[ActivePool.sol#L132-L136](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132-L136)

Below is the instance where the sum of `yieldSplitTreasury`, `yieldSplitSP`, and `yieldSplitStaking` is asserted to be `== 10_000`:

[ActivePool.sol#L144-L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144-L150)

Functions which would benefit from these values occupying the same storage slot by incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)` for every subsequent storage slot access: [ActivePool._rebalance() (`which is called within sendCollateral(), pullCollateralFromBorrowerOperationsOrDefaultPool(), and manualRebalance()`)](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L239-L309)

If instead `yieldSplitTreasury`, `yieldSplitSP`, and `yieldSplitStaking` are chosen to be packed together to save 2 SLOTs, the only function that would benefit from this optimization is the priviledged function [ActivePool.setYieldDistributionParams()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144-L150).

```solidity
File: Ethos-Core/contracts/ActivePool.sol
40:    address public lqtyStakingAddress;
41:    mapping (address => uint256) internal collAmount;  // collateral => amount tracker
42:    mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker
43:
44:    mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
45:    mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
46:    mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
47:    mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute
48:    
49:    uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
50:
51:    // Yield distribution params, must add up to 10k
52:    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
53:    uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
54:    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
```
**Packing `yieldingPercentageDrift`, `yieldSplitTreasury`, `yieldSplitStaking`, and `lqtyStakingAddress` into 1 storage slot**
```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..3e9bfdb 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -37,7 +37,6 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
     address public defaultPoolAddress;
     address public collSurplusPoolAddress;
     address public treasuryAddress;
-    address public lqtyStakingAddress;
     mapping (address => uint256) internal collAmount;  // collateral => amount tracker
     mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

@@ -46,13 +45,14 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
     mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
     mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute

-    uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS

     // Yield distribution params, must add up to 10k
-    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
     uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
-    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
+    uint32 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
+    uint32 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

+    uint32 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS
+    address public lqtyStakingAddress;
```
**Packing `yieldSplitTreasury`, `yieldSplitSP`, and `yieldSplitStaking` into 1 storage slot**
```diff
diff --git a/Ethos-Core/contracts/ActivePool.sol b/Ethos-Core/contracts/ActivePool.sol
index 753fcd0..3e9bfdb 100644
--- a/Ethos-Core/contracts/ActivePool.sol
+++ b/Ethos-Core/contracts/ActivePool.sol
@@ -49,9 +49,9 @@ contract ActivePool is Ownable, CheckContract, IActivePool {
     uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS

     // Yield distribution params, must add up to 10k
-    uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
-    uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
-    uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
+    uint32 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
+    uint32 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
+    uint32 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L63-L66

### We are able to pack `baseRate` and `lastFeeOperationTime` into 1 sotrage slot to save 1 SLOT (~2000 gas).

`lastFeeOperationTime` is simply a timestamp and therefore can be packed as a `uint40` value without fear of overflowing. See [reference 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [reference 2](https://twitter.com/RareSkills_io/status/1632033211609137156). `baseRate` is asserted to stay within certain bounds and therefore can be of type `> uint80, <= uint216`, which ensures the value stays well within the bounds.

Below is the instance where `baseRate` is asserted to be `<= DECIMAL_PRECISION`:

[TroveManager.sol#L1397-L1423](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1397-L1423)

Functions which would benefit from these values occupying the same storage slot by incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)` for every subsequent storage slot access: [TroveManager.updateBaseRateFromRedemption()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1397-L1423), [TroveManager.decayBaseRateFromBorrowing()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1485-L1495), [TroveManager.__calcDecayedBaseRate()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1509-L1514), [TroveManager.getRedemptionRateWithDecay()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1429-L1431), [TroveManager.getRedemptionFeeWithDecay()](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1444-L1446)

```solidity
File: Ethos-Core/contracts/TroveManager.sol
57:   /*
58:    * BETA: 18 digit decimal. Parameter by which to divide the redeemed fraction, in order to calc the new base rate from a redemption.
59:    * Corresponds to (1 / ALPHA) in the white paper.
60:    */
61:    uint constant public BETA = 2;
62:
63:    uint public baseRate;
64:
65:    // The timestamp of the latest fee operation (redemption or new LUSD issuance)
66:    uint public lastFeeOperationTime;
```
```diff
diff --git a/Ethos-Core/contracts/TroveManager.sol b/Ethos-Core/contracts/TroveManager.sol
index 6f587d9..7686d0f 100644
--- a/Ethos-Core/contracts/TroveManager.sol
+++ b/Ethos-Core/contracts/TroveManager.sol
@@ -60,10 +60,10 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager
     */
     uint constant public BETA = 2;

-    uint public baseRate;
+    uint80 public baseRate;

     // The timestamp of the latest fee operation (redemption or new LUSD issuance)
-    uint public lastFeeOperationTime;
+    uint40 public lastFeeOperationTime;
```

## Structs can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs) we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a `Gwarmaccess (100 gas)` versus a `Gcoldsload (2100 gas)`.

Gas savings: `4 * 2000 = 8000`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L25

### We are able to pack `activation`, `lasReport`, `feeBPS`, and `allocBPS` into a single storage slot to save 3 SLOTs (~6000 gas).

`activation` and `lastReport` are simply timestamps and therefore can be packed as `uint40` values without fear of overflowing. See [reference 1](https://twitter.com/PaulRBerg/status/1591832937179250693) and [reference 2](https://twitter.com/RareSkills_io/status/1632033211609137156). `feeBPS` and `allocBPS` are asserted to stay within certain bounds. `uint80` should be sufficient for these values.

Below are instances where `feeBPS` is asserted to be `<= PERCENT_DIVISOR / 5`:

[Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L155](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L155)

[Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L181)

Below are instances where `allocBPS` is asserted to be `<= PERCENT_DIVISOR`:

[Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L156](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L156)

[Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L199](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L199)

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
25:   struct StrategyParams {
26:       uint256 activation; // Activation block.timestamp
27:       uint256 feeBPS; // Performance fee taken from profit, in BPS
28:       uint256 allocBPS; // Allocation in BPS of vault's total assets
29:       uint256 allocated; // Amount of capital allocated to this strategy
30:       uint256 gains; // Total returns that Strategy has realized for Vault
31:       uint256 losses; // Total losses that Strategy has realized for Vault
32:       uint256 lastReport; // block.timestamp of the last time a report occured
33:   }
```

```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..c0b3c61 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -23,13 +23,13 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     struct StrategyParams {
-        uint256 activation; // Activation block.timestamp
-        uint256 feeBPS; // Performance fee taken from profit, in BPS
-        uint256 allocBPS; // Allocation in BPS of vault's total assets
+        uint40 activation; // Activation block.timestamp
+        uint40 lastReport; // block.timestamp of the last time a report occured
+        uint80 feeBPS; // Performance fee taken from profit, in BPS
+        uint80 allocBPS; // Allocation in BPS of vault's total assets
         uint256 allocated; // Amount of capital allocated to this strategy
         uint256 gains; // Total returns that Strategy has realized for Vault
         uint256 losses; // Total losses that Strategy has realized for Vault
-        uint256 lastReport; // block.timestamp of the last time a report occured
     }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L27-L32

### We are able to pack `allowed` and `decimals` into a single storage slot to save 1 SLOT (~2000 gas). I believe in practice `uint256` for `decimals` is unecessary and choosing a smaller uint, such as `uint8` would be sufficient, especially in this case.

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol
27:    struct Config {
28:        bool allowed;
29:        uint256 decimals;
30:        uint256 MCR;
31:        uint256 CCR;
32:    }
```

```diff
diff --git a/Ethos-Core/contracts/CollateralConfig.sol b/Ethos-Core/contracts/CollateralConfig.sol
index d524f23..534cd03 100644
--- a/Ethos-Core/contracts/CollateralConfig.sol
+++ b/Ethos-Core/contracts/CollateralConfig.sol
@@ -26,7 +26,7 @@ contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {

     struct Config {
         bool allowed;
-        uint256 decimals;
+        uint8 decimals;
         uint256 MCR;
         uint256 CCR;
     }
```

## Avoid contract existence checks by using solidity version 0.8.10 or later
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value.

In addition, `Ethos-Core/Dependencies/CheckContract.sol` already implements this check on necessary addresses so the compiler is not adding anything valuable with these additional checks.

Total instances: 87 

Gas savings: `87 * 100 = 8700`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L63

```solidity
File: Ethos-Core/contracts/CollaterolConfig.sol
63:             uint256 decimals = IERC20(collateral).decimals(); // @audit: `decimals()`
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L178

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol
178:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit `getCollateralCCR`

179:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit `getCollateralMCR`

180:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit `getCollateralDecimals`

181:         vars.price = priceFeed.fetchPrice(_collateral); // @audit: `fetchPrice()`

220:         contractsCache.troveManager.increaseTroveColl(msg.sender, _collateral, _collAmount); // @audit `increaseTroveColl()`, returns uint

221:         contractsCache.troveManager.increaseTroveDebt(msg.sender, _collateral, vars.compositeDebt); // @audit: `increaseeTroveDebt`, returns uint

224:         vars.stake = contractsCache.troveManager.updateStakeAndTotalStakes(msg.sender, _collateral); // @audit: `updateStakeAndTotalStakes`

227:         vars.arrayIndex = contractsCache.troveManager.addTroveOwnerToArray(msg.sender, _collateral); // @audit: `addTroveOwnerToArray`

286:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: `getCollateralCCR`

287:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: `getCollateralMCR`

288:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals`

289:         vars.price = priceFeed.fetchPrice(_collateral); // @audit: `fetchPrice`

316:         vars.debt = contractsCache.troveManager.getTroveDebt(_borrower, _collateral); // @audit: `getTroveDebt`

317:         vars.coll = contractsCache.troveManager.getTroveColl(_borrower, _collateral); // @audit: `getTroveColl`

344:         vars.stake = contractsCache.troveManager.updateStakeAndTotalStakes(_borrower, _collateral); // @audit: `updateStakeAndTotalStakes`

381:         uint256 collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: `getCollateralCCR()`

382:         uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audti: //getCollateralDecimals()`

383:         uint price = priceFeed.fetchPrice(_collateral); // @audit: `fetchPrice()`

388:         uint coll = troveManagerCached.getTroveColl(msg.sender, _collateral); // @audit: `getTroveColl()`

389:         uint debt = troveManagerCached.getTroveDebt(msg.sender, _collateral); // @audit: `getTroveDebt()`

421:         uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount); // @audit: `getBorrowingFee()`

468:         uint newColl = (_isCollIncrease) ? _troveManager.increaseTroveColl(_borrower, _collateral, _collChange) // @audit: `increaseTroveColl()`
469:                                        : _troveManager.decreaseTroveColl(_borrower, _collateral, _collChange); // @audit: `decreaseTroveColl()`
470:         uint newDebt = (_isDebtIncrease) ? _troveManager.increaseTroveDebt(_borrower, _collateral, _debtChange) // @audit: `increaseTroveColl()`
471:                                        : _troveManager.decreaseTroveDebt(_borrower, _collateral, _debtChange); // @audit: `decreaseTroveDebt`()

525:         require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address"); // @audit: `isCollateralAllowed()`

529:         require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance"); // @audit `balanceOf()`

530:         require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance"); // @audit: `allowance()`

542:         uint status = _troveManager.getTroveStatus(_borrower, _collateral); // @audit: `getTroveStatus()`

547:         uint status = _troveManager.getTroveStatus(_borrower, _collateral); // @audit: `getTroveStatus`()

645:         require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment"); // @audit `balanceOf()`
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L420

```solidity
File: Ethos-Core/contracts/TroveManager.sol
420:         uint collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

521:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit `getCollateralCCR()`

522:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit `getCollateralDecimals()`

523:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit `getCollateralMCR()`

524:         vars.price = priceFeed.fetchPrice(_collateral); // @audit `fetchPrice()`

525:         vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits(); // @audit `getTotalLUSDDeposits()`

596:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit `getCollateralDecimals`

597:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit `getCollateralCCR`

598:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit `getCollateralMCR`

606:         vars.user = _sortedTroves.getLast(_collateral); // @audit ``getLast()`

607:         address firstUser = _sortedTroves.getFirst(_collateral); // @audit: `getFirst()`

610:         address nextUser = _sortedTroves.getPrev(_collateral, vars.user); // @audit `getPrev()`

691:         vars.user = sortedTrovesCached.getLast(_collateral); // @audit `getLast()`

725:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit `getCollateralDecimals()`

726:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit `getCollateralCCR()`

727:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit `getCollateralMCR()`

728:         vars.price = priceFeed.fetchPrice(_collateral); // @audit: `fetchPrice()`

729:         vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits(); // @audit: `getTotalLUSDDeposits()`

798:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

799:         vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: `getCollateralCCR()`

800:         vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: `getCollateralMCR()`

1048:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

1061:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

1131:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

1146:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

1308:        uint activeColl = _activePool.getCollateral(_collateral); // @audit: `getCollateral()`

1309:        uint liquidatedColl = defaultPool.getCollateral(_collateral); // @audit `getCollateral()`

1362:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit `getCollateralDecimals()`

1367:        uint256 collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit `getCollateralCCR()`

1368:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L105

```solidity
File: Ethos-Core/contracts/ActivePool.sol
105:         address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals(); // @audit: `getAllowedCollaterals()`

111:         require(IERC4626(vault).asset() == collateral, "Vault asset must be collateral"); // @audit: `asset()`

247:         vars.ownedShares = vars.yieldGenerator.balanceOf(address(this)); // @audit: `balanceOf()`

248:         vars.sharesToAssets = vars.yieldGenerator.convertToAssets(vars.ownedShares); // @audit: `convertToAssets()`

280:         IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this)); // @audit `deposit()`

282:         IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this)); // @audit `withdraw()`

288:         vars.profit = IERC20(_collateral).balanceOf(address(this)).add(vars.yieldingAmount).sub(collAmount[_collateral]); // @audit `balanceOf()`

315:         ICollateralConfig(collateralConfigAddress).isCollateralAllowed(_collateral), // @audit: `isCollateralAllowed()`

328:         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper()); // @audit: `redemptionHelper()`
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L417

```solidity
File: Ethos-Core/contracts/StabilityPool.sol
417:         uint LQTYIssuance = _communityIssuance.issueOath(); // @audit: `issueOath()`

638:         assets = collateralConfig.getAllowedCollaterals(); // @audit: `getAllowedCollaterals()`

664:         vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: `getCollateralDecimals()`

806:         address[] memory collaterals = collateralConfig.getAllowedCollaterals(); // @audit: `getAllowedCollaterals()`

857:         address[] memory collaterals = collateralConfig.getAllowedCollaterals(); // @audit `getAllowedCollaterals()`

861:         uint price = priceFeed.fetchPrice(collateral); // @audit: `fetchPrice()`

862:         address lowestTrove = sortedTroves.getLast(collateral); // @audit: `getLast()`

863:         uint256 collMCR = collateralConfig.getCollateralMCR(collateral); // @audit `getCollateralMCR()`

864:         uint ICR = troveManager.getCurrentICR(lowestTrove, collateral, price); // @audit `getCurrentICR()`
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L103

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol
103:         OathToken.transferFrom(msg.sender, address(this), amount); // @audit `transferFrom()`

127:         OathToken.transfer(_account, _OathAmount); // @audit: `transfer()`
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L135

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
135:         lusdToken.transfer(msg.sender, LUSDGain); // @audit: `transfer()`

171:         lusdToken.transfer(msg.sender, LUSDGain); // @audit: `transfer()`

204:         assets = collateralConfig.getAllowedCollaterals(); // @audit `getAllowedCollaterals()`

226:         address[] memory collaterals = collateralConfig.getAllowedCollaterals(); // @audit `getAllowedCollaterals()`

252:         address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper()); // @audit: `redemptionHelper()`
```

## Multiple accesses of a mapping/array should use a storage pointer
Caching a mapping's value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `strategies[_strategy]`, save its reference via a storage pointer: `StrategyParams storage strategy = strategies[_strategy]` and use the pointer instead.

Total instances: 41

Gas savings: `36 (not benchmarked, and thus given an average savings value) * 40 + 529 (from benchmarked instances) = 1969`

### Gas Savings for `addStrategy()`, obtained via protocol's tests: Avg 224 gas | `strategies[_strategy]` can be cached as a storage pointer
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  170982  |  205182  |  182382 |    3     |
| After  |  170738  |  204938  |  182138 |    3     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L171

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
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
157:
158:        strategies[_strategy] = StrategyParams({
159:            activation: block.timestamp,
160:            feeBPS: _feeBPS,
161:            allocBPS: _allocBPS,
162:            allocated: 0,
163:            gains: 0,
164:            losses: 0,
165:            lastReport: block.timestamp
166:        });
167:
168:        totalAllocBPS += _allocBPS;
169:        withdrawalQueue.push(_strategy);
170:        emit StrategyAdded(_strategy, _feeBPS, _allocBPS);
171:    }
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..eecb943 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -149,21 +149,20 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         _atLeastRole(DEFAULT_ADMIN_ROLE);
         require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
         require(_strategy != address(0), "Invalid strategy address");
-        require(strategies[_strategy].activation == 0, "Strategy already added");
+        StrategyParams storage strategy = strategies[_strategy];
+        require(strategy.activation == 0, "Strategy already added");
         require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
         require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
         require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");

-        strategies[_strategy] = StrategyParams({
-            activation: block.timestamp,
-            feeBPS: _feeBPS,
-            allocBPS: _allocBPS,
-            allocated: 0,
-            gains: 0,
-            losses: 0,
-            lastReport: block.timestamp
-        });
+        strategy.activation = block.timestamp;
+        strategy.feeBPS = _feeBPS;
+        strategy.allocBPS = _allocBPS;
+        strategy.allocated = 0;
+        strategy.gains = 0;
+        strategy.losses = 0;
+        strategy.lastReport = block.timestamp;

         totalAllocBPS += _allocBPS;
         withdrawalQueue.push(_strategy);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L178-L184

### `strategies[_strategy]` can be cached as a storage pointer
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
178:    function updateStrategyFeeBPS(address _strategy, uint256 _feeBPS) external {
179:        _atLeastRole(ADMIN);
180:        require(strategies[_strategy].activation != 0, "Invalid strategy address");
181:        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
182:        strategies[_strategy].feeBPS = _feeBPS;
183:        emit StrategyFeeBPSUpdated(_strategy, _feeBPS);
184:    }
```

### Gas Savings for `updateStrategyAllocBPS()`, obtained via protocol's tests: Avg 172 gas | `strategies[_strategy]` can be cached as a storage pointer
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  34567  |  47361  |  41465 |    8     |
| After  |  34500  |  47294  |  41293 |    8     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L191-L199

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
191:    function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {
192:        _atLeastRole(STRATEGIST);
193:       require(strategies[_strategy].activation != 0, "Invalid strategy address");
194:        totalAllocBPS -= strategies[_strategy].allocBPS;
195:        strategies[_strategy].allocBPS = _allocBPS;
196:        totalAllocBPS += _allocBPS;
197:        require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
198:        emit StrategyAllocBPSUpdated(_strategy, _allocBPS);
199:    }
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..eecb943 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -190,9 +190,10 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {
         _atLeastRole(STRATEGIST);
-        require(strategies[_strategy].activation != 0, "Invalid strategy address");
-        totalAllocBPS -= strategies[_strategy].allocBPS;
-        strategies[_strategy].allocBPS = _allocBPS;
+        StrategyParams storage strategy = strategies[_strategy];
+        require(strategy.activation != 0, "Invalid strategy address");
+        totalAllocBPS -= strategy.allocBPS;
+        strategy.allocBPS = _allocBPS;
         totalAllocBPS += _allocBPS;
         require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
         emit StrategyAllocBPSUpdated(_strategy, _allocBPS); 
```

### Gas Savings for `revokeStrategy()`, obtained via protocol's tests: Avg 133 gas | `strategies[_strategy]` can be cached as a storage pointer
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  31639  |  33557  |  32515 |    6     |
| After  |  31516  |  33434  |  32382 |    6     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L205-L217

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
205:    function revokeStrategy(address _strategy) external {
206:        if (msg.sender != _strategy) {
207:            _atLeastRole(GUARDIAN);
208:        }
209:
230:        if (strategies[_strategy].allocBPS == 0) {
231:            return;
232:        }
233:
234:        totalAllocBPS -= strategies[_strategy].allocBPS;
235:        strategies[_strategy].allocBPS = 0;
236:        emit StrategyRevoked(_strategy);
237:    }
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..eecb943 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -206,13 +206,15 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
         if (msg.sender != _strategy) {
             _atLeastRole(GUARDIAN);
         }
+
+        StrategyParams storage strategy = strategies[_strategy];

-        if (strategies[_strategy].allocBPS == 0) {
+        if (strategy.allocBPS == 0) {
             return;
         }

-        totalAllocBPS -= strategies[_strategy].allocBPS;
-        strategies[_strategy].allocBPS = 0;
+        totalAllocBPS -= strategy.allocBPS;
+        strategy.allocBPS = 0;
         emit StrategyRevoked(_strategy);
     }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L232

### `strategies[stratAddr]` can be cached as a storage pointer
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
225:    function availableCapital() public view returns (int256) {
226:        address stratAddr = msg.sender;
227:        if (totalAllocBPS == 0 || emergencyShutdown) {
228:            return -int256(strategies[stratAddr].allocated);
229:        }
230:
231:        uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
232:        uint256 stratCurrentAllocation = strategies[stratAddr].allocated;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L378-L395

### ReaperVaultV2.__withdraw() | `strategies[stratAddr]` can be cached as a storage pointer
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
378:                address stratAddr = withdrawalQueue[i];
379:                uint256 strategyBal = strategies[stratAddr].allocated;
380:                if (strategyBal == 0) {
381:                    continue;
382:                }
383:
384:                uint256 remaining = value - vaultBalance;
385:                uint256 loss = IStrategy(stratAddr).withdraw(Math.min(remaining, strategyBal));
386:                uint256 actualWithdrawn = token.balanceOf(address(this)) - vaultBalance;
387:
388:                // Withdrawer incurs any losses from withdrawing as reported by strat
389:                if (loss != 0) {
390:                    value -= loss;
391:                    totalLoss += loss;
392:                    _reportLoss(stratAddr, loss);
393:                }
394:
395:                strategies[stratAddr].allocated -= actualWithdrawn;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L225-L236

### `snapshots[_user]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
225:    function _updateUserSnapshots(address _user) internal {
226:        address[] memory collaterals = collateralConfig.getAllowedCollaterals();
227:        uint[] memory amounts = new uint[](collaterals.length);
228:        for (uint i = 0; i < collaterals.length; i++) {
229:            address collateral = collaterals[i];
230:            snapshots[_user].F_Collateral_Snapshot[collateral] = F_Collateral[collateral];
231:            amounts[i] = F_Collateral[collateral];
232:        }
233:        uint fLUSD = F_LUSD;
234:        snapshots[_user].F_LUSD_Snapshot = fLUSD;
235:        emit StakerSnapshotsUpdated(_user, collaterals, amounts, fLUSD);
236:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L824-L838

### StabilityPool._updateDepositAndSnapshots() | `depositSnapshots[_depositor]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/StabilityPool.sol
824:        // Record new snapshots of the latest running product P, and sum G, for the depositor
825:        depositSnapshots[_depositor].P = currentP;
826:        depositSnapshots[_depositor].G = currentG;
827:        depositSnapshots[_depositor].scale = currentScaleCached;
828:        depositSnapshots[_depositor].epoch = currentEpochCached;
829:
830:        // Record new snapshots of the latest running sum S for all collaterals for the depositor
831:        for (uint i = 0; i < collaterals.length; i++) {
832:            address collateral = collaterals[i];
833:            uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];
834:            depositSnapshots[_depositor].S[collateral] = currentS;
835:            amounts[i] = currentS;
836:        }
837:        emit DepositSnapshotUpdated(_depositor, currentP, collaterals, amounts, currentG);
838:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L962-L980

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
962:    function updateDebtAndCollAndStakesPostRedemption(
963:        address _borrower,
964:        address _collateral,
965:        uint256 _newDebt,
967:        uint256 _newColl
968:    ) external override {
969:        _requireCallerIsRedemptionHelper();
970:        Troves[_borrower][_collateral].debt = _newDebt;
971:        Troves[_borrower][_collateral].coll = _newColl;
972:        _updateStakeAndTotalStakes(_borrower, _collateral);
973:
974:        emit TroveUpdated(
975:            _borrower,
976:            _collateral,
977:            _newDebt, _newColl,
978:            Troves[_borrower][_collateral].stake,
979:            TroveManagerOperation.redeemCollateral
980:        );
981:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1066-L1074

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1066:    function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {
1067:        uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);
1068:        uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);
1069:
1070:        uint currentCollateral = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);
1071:        uint currentLUSDDebt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);
1072:
1073:        return (currentCollateral, currentLUSDDebt);
1074:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1082-L1108

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1082:    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
1083:        if (hasPendingRewards(_borrower, _collateral)) {
1084:            _requireTroveIsActive(_borrower, _collateral);
1085:
1086:            // Compute pending rewards
1087:            uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);
1088:            uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);
1089:
1090:            // Apply pending rewards to trove's state
1091:            Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward);
1092:            Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward);
1093:
1094:            _updateTroveRewardSnapshots(_borrower, _collateral);
1095:
1096:            // Transfer from DefaultPool to ActivePool
1097:            _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, pendingLUSDDebtReward, pendingCollateralReward);
1098:
1099:            emit TroveUpdated(
1100:                _borrower,
1101:                _collateral,
1102:                Troves[_borrower][_collateral].debt,
1103:                Troves[_borrower][_collateral].coll,
1104:                Troves[_borrower][_collateral].stake,
1105:                TroveManagerOperation.applyPendingRewards
1106:            );
1107:        }
1108:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1116-L1120

### `rewardSnapshots[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1116:    function _updateTroveRewardSnapshots(address _borrower, address _collateral) internal {
1117:        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral];
1118:        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral];
1119:        emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]);
1120:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1123-L1135

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1123:    function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {
1124:        uint snapshotCollateral = rewardSnapshots[_borrower][_collateral].collAmount;
1125:        uint rewardPerUnitStaked = L_Collateral[_collateral].sub(snapshotCollateral);
1126:
1127:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
1128:
1129:        uint stake = Troves[_borrower][_collateral].stake;
1130:
1131:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
1132:        uint pendingCollateralReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);
1133:
1134:        return pendingCollateralReward;
1135:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1138-L1150

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1138:    function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {
1139:        uint snapshotLUSDDebt = rewardSnapshots[_borrower][_collateral].LUSDDebt;
1140:        uint rewardPerUnitStaked = L_LUSDDebt[_collateral].sub(snapshotLUSDDebt);
1141:
1142:        if ( rewardPerUnitStaked == 0 || Troves[_borrower][_collateral].status != Status.active) { return 0; }
1143:
1144:        uint stake = Troves[_borrower][_collateral].stake;
1145:
1146:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral);
1147:        uint pendingLUSDDebtReward = stake.mul(rewardPerUnitStaked).div(10**collDecimals);
1148:
1149:        return pendingLUSDDebtReward;
1150:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1164-L1181

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1164:    function getEntireDebtAndColl(
1165:        address _borrower,
1166:        address _collateral
1167:    )
1168:        public
1169:        view
1170:        override
1171:        returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)
1172:    {
1173:        debt = Troves[_borrower][_collateral].debt;
1174:        coll = Troves[_borrower][_collateral].coll;
1175:
1176:        pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);
1177:        pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);
1178:
1179:        debt = debt.add(pendingLUSDDebtReward);
1180:        coll = coll.add(pendingCollateralReward);
1181:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1189-L1193

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1189:    function _removeStake(address _borrower, address _collateral) internal {
1190:        uint stake = Troves[_borrower][_collateral].stake;
1191:        totalStakes[_collateral] = totalStakes[_collateral].sub(stake);
1192:        Troves[_borrower][_collateral].stake = 0;
1193:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1201-L1210

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
11201:   function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {
11202:        uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);
11203:        uint oldStake = Troves[_borrower][_collateral].stake;
11204:        Troves[_borrower][_collateral].stake = newStake;
11205:
11206:        totalStakes[_collateral] = totalStakes[_collateral].sub(oldStake).add(newStake);
11207:        emit TotalStakesUpdated(_collateral, totalStakes[_collateral]);
11208:
11209:        return newStake;
11210:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1278-L1293

### `Troves[_borrower][_collateral]` and `rewardSnapshots[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1278:    function _closeTrove(address _borrower, address _collateral, Status closedStatus) internal {
1279:        assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1280:
1281:        uint TroveOwnersArrayLength = TroveOwners[_collateral].length;
1282:        _requireMoreThanOneTroveInSystem(TroveOwnersArrayLength, _collateral);
1283:
1284:        Troves[_borrower][_collateral].status = closedStatus;
1285:        Troves[_borrower][_collateral].coll = 0;
1286:        Troves[_borrower][_collateral].debt = 0;
1287:
1288:        rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289:        rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
1290:
1291:        _removeTroveOwner(_borrower, _collateral, TroveOwnersArrayLength);
1292:        sortedTroves.remove(_collateral, _borrower);
1293:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321-L1333

### `TroveOwners[_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1321:    function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {
1322:        /* Max array size is 2**128 - 1, i.e. ~3e30 troves. No risk of overflow, since troves have minimum LUSD
1323:        debt of liquidation reserve plus MIN_NET_DEBT. 3e30 LUSD dwarfs the value of all wealth in the world ( which is < 1e15 USD). */
1324:
1325:        // Push the Troveowner to the array
1326:        TroveOwners[_collateral].push(_borrower);
1327:
1328:        // Record the index of the new Troveowner on their Trove struct
1329:        index = uint128(TroveOwners[_collateral].length.sub(1));
1330:        Troves[_borrower][_collateral].arrayIndex = index;
1331:
1332:        return index;
1333:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1339-L1357

### `Troves[_borrower][_collateral]` and `TroveOwners[_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1339:    function _removeTroveOwner(address _borrower, address _collateral, uint TroveOwnersArrayLength) internal {
1340:        Status troveStatus = Troves[_borrower][_collateral].status;
1341:        // It’s set in caller function `_closeTrove`
1342:        assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1343:
1344:        uint128 index = Troves[_borrower][_collateral].arrayIndex;
1345:        uint length = TroveOwnersArrayLength;
1346:        uint idxLast = length.sub(1);
1347:
1348:        assert(index <= idxLast);
1349:
1350:        address addressToMove = TroveOwners[_collateral][idxLast];
1351:
1352:        TroveOwners[_collateral][index] = addressToMove;
1353:        Troves[addressToMove][_collateral].arrayIndex = index;
1354:        emit TroveIndexUpdated(addressToMove, _collateral, index);
1355:
1356:        TroveOwners[_collateral].pop();
1357:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1567-L1572

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1567:    function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {
1568:        _requireCallerIsBorrowerOperations();
1569:        uint newColl = Troves[_borrower][_collateral].coll.add(_collIncrease);
1570:        Troves[_borrower][_collateral].coll = newColl;
1571:        return newColl;
1572:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1574-L1579

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1574:    function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {
1575:        _requireCallerIsBorrowerOperations();
1576:        uint newColl = Troves[_borrower][_collateral].coll.sub(_collDecrease);
1577:        Troves[_borrower][_collateral].coll = newColl;
1578:        return newColl;
1579:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1581-L1586

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1581:    function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {
1582:        _requireCallerIsBorrowerOperations();
1583:        uint newDebt = Troves[_borrower][_collateral].debt.add(_debtIncrease);
1584:        Troves[_borrower][_collateral].debt = newDebt;
1585:        return newDebt;
1586:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1588-L1593

### `Troves[_borrower][_collateral]` can be cached as a storage pointer
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1588:   function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {
1589:        _requireCallerIsBorrowerOperations();
1590:        uint newDebt = Troves[_borrower][_collateral].debt.sub(_debtDecrease);
1591:        Troves[_borrower][_collateral].debt = newDebt;
1592:        return newDebt;
1593:    }
```

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. **Note that these are instances that the c4udit tool missed**.

Total Instances: 34

Gas savings: `34 * 100 = 3400`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176-L182

### Cache `want` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
176:    function _deposit(uint256 toReinvest) internal {
177:        if (toReinvest != 0) {
178:            address lendingPoolAddress = ADDRESSES_PROVIDER.getLendingPool();
179:            IERC20Upgradeable(want).safeIncreaseAllowance(lendingPoolAddress, toReinvest); // @audit: 1st SLOAD
180:            ILendingPool(lendingPoolAddress).deposit(want, toReinvest, address(this), LENDER_REFERRAL_CODE_NONE); // @audit: 2nd SLOAD
181:        }
182:    }
```

### Gas Savings for `withdraw()`, obtained via protocol's tests: Avg 113 gas | `msg.sender` can be used instead of `vault` to save 1 SLOAD
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  -  |  -  |  318466 |    1     |
| After  |  -  |  -  |  318353 |    1     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L95-L103
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
95:    function withdraw(uint256 _amount) external override returns (uint256 loss) {
96:        require(msg.sender == vault, "Only vault can withdraw"); // @audit: 1st SLOAD
97:        require(_amount != 0, "Amount cannot be zero");
98:        require(_amount <= balanceOf(), "Ammount must be less than balance");
99:
100:        uint256 amountFreed = 0;
101:        (amountFreed, loss) = _liquidatePosition(_amount);
102:        IERC20Upgradeable(want).safeTransfer(vault, amountFreed); // @audit 2nd SLOAD
103:    }
```

### Gas Savings for `harvest()`, obtained via protocol's tests: Avg 83 gas | `vault` can be cached as a stack variable to save 1 SLOAD
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  452564  |  922717  |  536706 |    33     |
| After  |  452481  |  922634  |  536623 |    33     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L109-L138

### Cache `vault` as a stack variable to save 1 SLOAD
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
109:    function harvest() external override returns (int256 roi) {
110:        _atLeastRole(KEEPER);
111:        int256 availableCapital = IVault(vault).availableCapital(); // @audit: 1st SLOAD
112:        uint256 debt = 0;
113:        if (availableCapital < 0) {
114:            debt = uint256(-availableCapital);
115:        }
116:
117:        uint256 repayment = 0;
118:        if (emergencyExit) {
119:            uint256 amountFreed = _liquidateAllPositions();
120:            if (amountFreed < debt) {
121:                roi = -int256(debt - amountFreed);
122:            } else if (amountFreed > debt) {
123:                roi = int256(amountFreed - debt);
124:            }
125:
126:            repayment = debt;
127:            if (roi < 0) {
128:                repayment -= uint256(-roi);
129:            }
130:        } else {
131:            (roi, repayment) = _harvestCore(debt);
132:        }
133:
134:        debt = IVault(vault).report(roi, repayment); // @audit: 2nd SLOAD
135:        _adjustPosition(debt);
136:
137:        lastHarvestTimestamp = block.timestamp;
138:    }
```
### Gas Savings for `addStrategy()`, obtained via protocol's tests: Avg 123 gas | `totalAllocBPS` can be cached as a stack variable to save 1 SLOAD.
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  170982  |  205182  |  182382 |    3     |
| After  |  170859  |  205059  |  182259 |    3     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L144-L171
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
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
156:        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value"); // @audit: 1st SLOAD
157:
158:        strategies[_strategy] = StrategyParams({
159:            activation: block.timestamp,
160:            feeBPS: _feeBPS,
161:            allocBPS: _allocBPS,
162:            allocated: 0,
163:            gains: 0,
164:            losses: 0,
165:            lastReport: block.timestamp
166:        });
167:
168:        totalAllocBPS += _allocBPS; // @audit: 2nd SLOAD
169:        withdrawalQueue.push(_strategy);
170:        emit StrategyAdded(_strategy, _feeBPS, _allocBPS);
171:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L418-L425

### Cache `lockedProfit` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
418:    function _calculateLockedProfit() internal view returns (uint256) {
419:        uint256 lockedFundsRatio = (block.timestamp - lastReport) * lockedProfitDegradation;
420:        if (lockedFundsRatio < DEGRADATION_COEFFICIENT) {
421:            return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT); // @audit: 2 sloads
422:        }
423:
424:        return 0;
452:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L432-L453

### Cache `totalAllocBPS` as a stack variable + re-use cached `allocated` variable to save 2 SLOADS.

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
432:    function _reportLoss(address strategy, uint256 loss) internal {
433:        StrategyParams storage stratParams = strategies[strategy];
434:        // Loss can only be up the amount of capital allocated to the strategy
435:        uint256 allocation = stratParams.allocated; // @audit: 1 sload total
436:        require(loss <= allocation, "Strategy loss cannot be greater than allocation");
437:
438:        if (totalAllocBPS != 0) { // @audit 2 sloads total
439:            // reduce strat's allocBPS proportional to loss
440:            uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS); // @audit: 3 sloads total
441:
442:            // If the loss is too small, bpsChange will be 0
443:            if (bpsChange != 0) {
444:                stratParams.allocBPS -= bpsChange; // 
445:                totalAllocBPS -= bpsChange; // 
446:            }
447:        }
448:
449:        // Finally, adjust our strategy's parameters by the loss
450:        stratParams.losses += loss;
451:        stratParams.allocated -= loss; // @audit 4 sloads total
452:        totalAllocated -= loss; 
453:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L543-L560

### Cache `strategy.allocBPS` as a stack variable to save 1 SLOAD.

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
543:        emit StrategyReported(
544:            vars.stratAddr,
545:            vars.gain,
546:            vars.loss,
547:            vars.debtPayment,
548:            strategy.gains,
549:            strategy.losses,
550:            strategy.allocated,
551:            vars.credit,
552:            strategy.allocBPS // @audit: 1st sload
553:        );
554:
555:        if (strategy.allocBPS == 0 || emergencyShutdown) { // @audit: 2nd sload
556:            return IStrategy(vars.stratAddr).balanceOf();
557:        }
558:
559:        return vars.debt;
560:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251-L259

### Cache `troveManagerAddress` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
251:    function _requireCallerIsTroveManagerOrActivePool() internal view {
252:        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper()); // @audit: 1st sload
253:        require(
254:            msg.sender == troveManagerAddress || // @audit 2nd sload
255:            msg.sender == redemptionHelper ||
256:            msg.sender == activePoolAddress,
257:            "LQTYStaking: caller is not TroveM or ActivePool"
258:        );
259:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L239-L309

### Cache `yieldGenerator[_collateral]` as a stack variable and use already cached `yieldingAmount[_collateral]` to save at most 2 SLOADs.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
239:    function _rebalance(address _collateral, uint256 _amountLeavingPool) internal {
240:        LocalVariables_rebalance memory vars;
241:
242:        // how much has been allocated as per our internal records?
243:        vars.currentAllocated = yieldingAmount[_collateral]; // @audit: 1st sload for `yieldingAmount[_collateral]`
244:        
245:        // what is the present value of our shares?
246:        vars.yieldGenerator = IERC4626(yieldGenerator[_collateral]); // @audit 1st load for `yieldGenerator[_collateral]`
247:        vars.ownedShares = vars.yieldGenerator.balanceOf(address(this));
248:        vars.sharesToAssets = vars.yieldGenerator.convertToAssets(vars.ownedShares);
249:
250:        // if we have profit that's more than the threshold, record it for withdrawal and redistribution
251:        vars.profit = vars.sharesToAssets.sub(vars.currentAllocated);
252:        if (vars.profit < yieldClaimThreshold[_collateral]) {
253:            vars.profit = 0;
254:        }
255:        
256:        // what % of the final pool balance would the current allocation be?
257:        vars.finalBalance = collAmount[_collateral].sub(_amountLeavingPool);
258:        vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);
259:
260:        // if abs(percentOfFinalBal - yieldingPercentage) > drift, we will need to deposit more or withdraw some
261:        vars.yieldingPercentage = yieldingPercentage[_collateral];
262:        vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
263:        vars.yieldingAmount = yieldingAmount[_collateral]; // @audit 2nd sload for `yieldingAmount[_collateral]`, use `vars.currentAllocated`
264:        if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {
265:            // we will end up overallocated, withdraw some
266:            vars.toWithdraw = vars.currentAllocated.sub(vars.finalYieldingAmount);
267:            vars.yieldingAmount = vars.yieldingAmount.sub(vars.toWithdraw);
268:            yieldingAmount[_collateral] = vars.yieldingAmount;
269:        } else if(vars.percentOfFinalBal < vars.yieldingPercentage && vars.yieldingPercentage.sub(vars.percentOfFinalBal) > yieldingPercentageDrift) {
270:            // we will end up underallocated, deposit more
271:            vars.toDeposit = vars.finalYieldingAmount.sub(vars.currentAllocated);
272:            vars.yieldingAmount = vars.yieldingAmount.add(vars.toDeposit);
273:            yieldingAmount[_collateral] = vars.yieldingAmount;
274:        }
275:
276:        // + means deposit, - means withdraw
277:        vars.netAssetMovement = int256(vars.toDeposit) - int256(vars.toWithdraw) - int256(vars.profit);
278:        if (vars.netAssetMovement > 0) {
279:            IERC20(_collateral).safeIncreaseAllowance(yieldGenerator[_collateral], uint256(vars.netAssetMovement)); // @audit 2nd load for `yieldGenerator[_collateral]`
280:            IERC4626(yieldGenerator[_collateral]).deposit(uint256(vars.netAssetMovement), address(this)); // @audit 3rd load for `yieldGenerator[_collateral]`
281:        } else if (vars.netAssetMovement < 0) {
282:            IERC4626(yieldGenerator[_collateral]).withdraw(uint256(-vars.netAssetMovement), address(this), address(this)); // @audit 2nd load for `yieldGenerator[_collateral]`
283:        }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L327-L335

### Consider rearranging require statements and Cache `troveManagerAddress` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
327:    function _requireCallerIsBOorTroveMorSP() internal view {
328:        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper()); // @audit: 1st sload
329:        require(
330:            msg.sender == borrowerOperationsAddress ||
331:            msg.sender == troveManagerAddress || // @audit: 2nd sload. Ensure savings by placing this statement 1st
332:            msg.sender == redemptionHelper ||
333:            msg.sender == stabilityPoolAddress,
334:            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
335:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L513-L526

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
513:    function liquidateTroves(address _collateral, uint _n) external override {
514:        IActivePool activePoolCached = activePool;
515:        IDefaultPool defaultPoolCached = defaultPool;
516:        IStabilityPool stabilityPoolCached = stabilityPool;
517:
518:        LocalVariables_OuterLiquidationFunction memory vars;
519:        LiquidationTotals memory totals;
520:
521:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 1st sload
522:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: 2nd sload
523:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: 3rd sload
524:        vars.price = priceFeed.fetchPrice(_collateral);
525:        vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();
526:        vars.recoveryModeAtStart = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L582-L599

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
582:    function _getTotalsFromLiquidateTrovesSequence_RecoveryMode
583:    (
584:        IActivePool _activePool,
585:        IDefaultPool _defaultPool,
586:        ISortedTroves _sortedTroves,
587:        address _collateral,
588:        uint _price,
589:        uint _LUSDInStabPool,
590:        uint _n
591:    )
592:        internal
593:        returns(LiquidationTotals memory totals)
594:    {
595:        LocalVariables_LiquidationSequence memory vars;
596:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: 1st sload
597:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit 2nd sload
598:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit 3rd sload
599:        LiquidationValues memory singleLiquidation;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L715-L730

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
715:    function batchLiquidateTroves(address _collateral, address[] memory _troveArray) public override {
716:        require(_troveArray.length != 0);
717:
718:        IActivePool activePoolCached = activePool;
719:        IDefaultPool defaultPoolCached = defaultPool;
720:        IStabilityPool stabilityPoolCached = stabilityPool;
721:
722:        LocalVariables_OuterLiquidationFunction memory vars;
723:        LiquidationTotals memory totals;
724:
725:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: 1st sload
726:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 2nd sload
727:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: 3rd sload
728:        vars.price = priceFeed.fetchPrice(_collateral);
729:        vars.LUSDInStabPool = stabilityPoolCached.getTotalLUSDDeposits();
730:        vars.recoveryModeAtStart = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L785-L801

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
785:    function _getTotalFromBatchLiquidate_RecoveryMode
786:    (
787:        IActivePool _activePool,
788:        IDefaultPool _defaultPool,
789:        address _collateral,
790:        uint _price,
791:        uint _LUSDInStabPool,
792:        address[] memory _troveArray
793:    )
794:        internal
795:        returns(LiquidationTotals memory totals)
796:    {
797:        LocalVariables_LiquidationSequence memory vars;
798:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: 1st sload
799:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 2nd sload
800:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: 3rd sload
801:        LiquidationValues memory singleLiquidation;
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L939-L955

### Cache `activePool` and `collSurplusPool` as stack variables to save 2 SLOADS.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
939:    function redeemCloseTrove(
940:        address _borrower,
941:        address _collateral,
942:        uint256 _LUSD,
943:        uint256 _collAmount
944:    ) external override {
945:        _requireCallerIsRedemptionHelper();
946:        lusdToken.burn(gasPoolAddress, _LUSD);
947:        // Update Active Pool LUSD, and send ETH to account
948:        activePool.decreaseLUSDDebt(_collateral, _LUSD); // @audit: 1st sload (`activePool`)
949:
950:        // send ETH from Active Pool to CollSurplus Pool
951:        collSurplusPool.accountSurplus(_borrower, _collateral, _collAmount); // @audit: 1st sload (`collSurplusPool`)
952:        activePool.sendCollateral(_collateral, address(collSurplusPool), _collAmount); // @audit: 2nd sload for `activePool` and `collSurplusPool`
953:
954:        emit TroveUpdated(_borrower, _collateral, 0, 0, 0, TroveManagerOperation.redeemCollateral);
955:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1213-L1228

### Cache `totalStakesSnapshot[_collateral]` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1213:    function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {
1214:        uint stake;
1215:        if (totalCollateralSnapshot[_collateral] == 0) {
1216:            stake = _coll;
1217:        } else {
1218:            /*
1219:            * The following assert() holds true because:
1220:            * - The system always contains >= 1 trove
1221:            * - When we close or liquidate a trove, we redistribute the pending rewards, so if all troves were closed/liquidated,
1222:            * rewards would’ve been emptied and totalCollateralSnapshot would be zero too.
1223:            */
1224:            assert(totalStakesSnapshot[_collateral] > 0); // @audit: 1st sload
1225:            stake = _coll.mul(totalStakesSnapshot[_collateral]).div(totalCollateralSnapshot[_collateral]); // @audit: 2nd sload
1226:        }
1227:        return stake;
1228:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1230-L1259

### Cache `totalStakes[_collateral]` as a stack variable to save 3 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1230:    function _redistributeDebtAndColl(
1231:        IActivePool _activePool,
1232:        IDefaultPool _defaultPool,
1233:        address _collateral,
1234:        uint _debt,
1235:        uint _coll,
1236:        uint256 _collDecimals
1237:    ) internal {
1238:        if (_debt == 0) { return; }
1239:
...
1251:        uint collateralNumerator = _coll.mul(10**_collDecimals).add(lastCollateralError_Redistribution[_collateral]);
1252:        uint LUSDDebtNumerator = _debt.mul(10**_collDecimals).add(lastLUSDDebtError_Redistribution[_collateral]);
1253:
1254:        // Get the per-unit-staked terms
1255:        uint collateralRewardPerUnitStaked = collateralNumerator.div(totalStakes[_collateral]); // @audit: 1st sload
1256:        uint LUSDDebtRewardPerUnitStaked = LUSDDebtNumerator.div(totalStakes[_collateral]); // @audit: 2nd sload
1257:
1258:        lastCollateralError_Redistribution[_collateral] = collateralNumerator.sub(collateralRewardPerUnitStaked.mul(totalStakes[_collateral])); // @audit: 3rd sload
1259:        lastLUSDDebtError_Redistribution[_collateral] = LUSDDebtNumerator.sub(LUSDDebtRewardPerUnitStaked.mul(totalStakes[_collateral])); // @audit: 4th sload
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1366-L1370

### Cache `collateralConfig` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1336:    function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {
1337:        uint256 collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 1st sload
1338:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit 2nd sload
1339:        return _checkRecoveryMode(_collateral, _price, collCCR, collDecimals);
1340:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172-L182

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol
172:    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {
173:        _requireValidCollateralAddress(_collateral);
174:        _requireSufficientCollateralBalanceAndAllowance(msg.sender, _collateral, _collAmount);
175:        ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);
176:        LocalVariables_openTrove memory vars;
177:
178:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 1st sload
179:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit 2nd sload
180:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit 3rd sload
181:        vars.price = priceFeed.fetchPrice(_collateral);
182:        bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282-L290

### Cache `collateralConfig` as a stack variable to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol
282:    function _adjustTrove(address _borrower, address _collateral, uint _collTopUp, uint _collWithdrawal, uint _LUSDChange, bool _isDebtIncrease, address _upperHint, address _lowerHint, uint _maxFeePercentage) internal {
283:        ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);
284:        LocalVariables_adjustTrove memory vars;
285:
286:        vars.collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 1st sload
287:        vars.collMCR = collateralConfig.getCollateralMCR(_collateral); // @audit: 2nd sload
288:        vars.collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit: 3rd sload
289:        vars.price = priceFeed.fetchPrice(_collateral);
290:        bool isRecoveryMode = _checkRecoveryMode(_collateral, vars.price, vars.collCCR, vars.collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L375-L384

### Cache `collateralConfig` as a stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol
375:    function closeTrove(address _collateral) external override {
376:        ITroveManager troveManagerCached = troveManager;
377:        IActivePool activePoolCached = activePool;
378:        ILUSDToken lusdTokenCached = lusdToken;
379:
380:        _requireTroveisActive(troveManagerCached, msg.sender, _collateral);
381:        uint256 collCCR = collateralConfig.getCollateralCCR(_collateral); // @audit: 1st sload
382:        uint256 collDecimals = collateralConfig.getCollateralDecimals(_collateral); // @audit 2nd sload
383:        uint price = priceFeed.fetchPrice(_collateral);
384:        _requireNotInRecoveryMode(_collateral, price, collCCR, collDecimals);
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419-L430

### Cache `lqtyStakingAddress` as a stack variable and use the `ILQTYStaking()` interface to call address in stack variable instead of loading `lqtyStaking`. Saves 1 SLOAD.
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol
419:    function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {
420:        _troveManager.decayBaseRateFromBorrowing(); // decay the baseRate state variable
421:        uint LUSDFee = _troveManager.getBorrowingFee(_LUSDAmount);
422:
423:        _requireUserAcceptsFee(LUSDFee, _LUSDAmount, _maxFeePercentage);
424:        
425:        // Send fee to LQTY staking contract
426:        lqtyStaking.increaseF_LUSD(LUSDFee); // @audit unecessary sload, cache `lqtyStakingAddress` and use interface
427:        _lusdToken.mint(lqtyStakingAddress, LUSDFee); // @audit use cached var
428:
429:        return LUSDFee;
430:    }
```

## Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

Total Instances: 26

Gas savings: `26 * 100 = 2600`

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581

### Avoid 1 SLOAD by emitting `_newTvlCap` instead of `tvlCap`.
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
578:    function updateTvlCap(uint256 _newTvlCap) public {
579:        _atLeastRole(ADMIN);
580:        tvlCap = _newTvlCap;
581:        emit TvlCapUpdated(tvlCap); // @audit: unecessary sload. Use `_newTvlCap`
582:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L122-L138

### Cache `totalLQTYStaked.add(_LQTYamount)` as a stack variable to save 1 SLOAD and avoid emitting a storage variable. Ex: `_totalStaked = totalLQTYStaked.add(_LQTYamount)`. Then set `totalLQTYStaked` equal to `_totalStaked` and emit `_totalStaked`.

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
122:        // Increase user’s stake and total LQTY staked
123:        stakes[msg.sender] = newStake;
124:        totalLQTYStaked = totalLQTYStaked.add(_LQTYamount); // @audit 1st sload
125:        emit TotalLQTYStakedUpdated(totalLQTYStaked); // @audit 2nd sload
125:
127:        // Transfer LQTY from caller to this contract
128:        lqtyToken.safeTransferFrom(msg.sender, address(this), _LQTYamount);
129:
130:        emit StakeChanged(msg.sender, newStake);
131:        emit StakingGainsWithdrawn(msg.sender, LUSDGain, collGainAssets, collGainAmounts);
132:
133:         // Send accumulated LUSD and collateral gains to the caller
134:        if (currentStake != 0) {
135:            lusdToken.transfer(msg.sender, LUSDGain);
136:            _sendCollGainToUser(collGainAssets, collGainAmounts);
137:        }
138:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L152-L166

### Cache `totalLQTYStaked.sub(LQTYToWithdraw)` similarly to the instance above to save 1 SLOAD and avoid emitting a storage variable.
```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
152:        if (_LQTYamount > 0) {
153:            uint LQTYToWithdraw = LiquityMath._min(_LQTYamount, currentStake);
154:
155:            uint newStake = currentStake.sub(LQTYToWithdraw);
156:
157:            // Decrease user's stake and total LQTY staked
158:            stakes[msg.sender] = newStake;
159:            totalLQTYStaked = totalLQTYStaked.sub(LQTYToWithdraw); // @audit: 1st sload
160:            emit TotalLQTYStakedUpdated(totalLQTYStaked); // @audit 2nd sload
161:
162:            // Transfer unstaked LQTY to user
163:            lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);
164:
165:            emit StakeChanged(msg.sender, newStake);
166:        }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L177-L185

### Cache `F_Collateral[_collateral].add(collFeePerLQTYStaked)` similarly to the instance above to save 1 SLOAD and avoid emitting a storage variable.
```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
177:    function increaseF_Collateral(address _collateral, uint _collFee) external override {
178:        _requireCallerIsTroveManagerOrActivePool();
179:        uint collFeePerLQTYStaked;
180:     
181:        if (totalLQTYStaked > 0) {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
182:
183:        F_Collateral[_collateral] = F_Collateral[_collateral].add(collFeePerLQTYStaked); // @audit: 1st sload
184:        emit F_CollateralUpdated(_collateral, F_Collateral[_collateral]); // @audit 2nd sload
185:    }
```

### Cache `F_LUSD.add(LUSDFeePerLQTYStaked)` similarly to the instance above to save 1 SLOAD and avoid emitting a storage variable.
```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
186:
187:    function increaseF_LUSD(uint _LUSDFee) external override {
188:        _requireCallerIsBorrowerOperations();
189:        uint LUSDFeePerLQTYStaked;
190:        
191:        if (totalLQTYStaked > 0) {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
192:        
193:        F_LUSD = F_LUSD.add(LUSDFeePerLQTYStaked); // @audit: 1st slaod
194:        emit F_LUSDUpdated(F_LUSD); // @audit: 2nd sload
195:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L421-L437

### Cache `currentEpoch`, `currentScale`, and `epochToScaleToG[currentEpoch][currentScale]` (using cached values) to save 7 SLOADS and avoid emitting storage variables.
```solidity
File: Ethos-Core/contracts/StabilityPool.sol
421:    function _updateG(uint _LQTYIssuance) internal {
422:        uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD
423:        /*
424:        * When total deposits is 0, G is not updated. In this case, the LQTY issued can not be obtained by later
425:        * depositors - it is missed out on, and remains in the balanceof the CommunityIssuance contract.
426:        *
427:        */
428:        if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}
429:
430:        uint LQTYPerUnitStaked;
431:        LQTYPerUnitStaked =_computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSD);
432:
433:        uint marginalLQTYGain = LQTYPerUnitStaked.mul(P);
434:        epochToScaleToG[currentEpoch][currentScale] = epochToScaleToG[currentEpoch][currentScale].add(marginalLQTYGain); // @audit 6 sloads total
435:
436:        emit G_Updated(epochToScaleToG[currentEpoch][currentScale], currentEpoch, currentScale); // @audit: 11 sloads total
437:    }
```
```diff
diff --git a/Ethos-Core/contracts/StabilityPool.sol b/Ethos-Core/contracts/StabilityPool.sol
index db163b7..3dbcc17 100644
--- a/Ethos-Core/contracts/StabilityPool.sol
+++ b/Ethos-Core/contracts/StabilityPool.sol
@@ -431,9 +431,12 @@ contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {
         LQTYPerUnitStaked =_computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSD);

         uint marginalLQTYGain = LQTYPerUnitStaked.mul(P);
-        epochToScaleToG[currentEpoch][currentScale] = epochToScaleToG[currentEpoch][currentScale].add(marginalLQTYGain);
+        uint128 _currentEpoch = currentEpoch;
+        uint128 _currentScale = currentScale;
+        uint256 updatedG = epochToScaleToG[_currentEpoch][_currentScale].add(marginalLQTYGain) ;
+        epochToScaleToG[_currentEpoch][_currentScale] = updatedG;

-        emit G_Updated(epochToScaleToG[currentEpoch][currentScale], currentEpoch, currentScale);
+        emit G_Updated(updatedG, _currentEpoch, _currentScale);
     }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L583-L589

### Cache `currentScaleCached.add(1)` as a stack variable and emit cached stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/StabilityPool.sol
...
583:        } else if (currentP.mul(newProductFactor).div(DECIMAL_PRECISION) < SCALE_FACTOR) {
584:            newP = currentP.mul(newProductFactor).mul(SCALE_FACTOR).div(DECIMAL_PRECISION); 
585:            currentScale = currentScaleCached.add(1);
586:            emit ScaleUpdated(currentScale); // @audit: unecessary sload. cache `currentScaleCached.add(1)` and emit
587:        } else {
588:            newP = currentP.mul(newProductFactor).div(DECIMAL_PRECISION);
589:        }
...
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L171-L176

### Cache `collAmount[_collateral].sub(_amount)` as a stack variable and emit cached stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
171:    function sendCollateral(address _collateral, address _account, uint _amount) external override {
172:        _requireValidCollateralAddress(_collateral);
173:        _requireCallerIsBOorTroveMorSP();
174:        _rebalance(_collateral, _amount);
175:        collAmount[_collateral] = collAmount[_collateral].sub(_amount); // @audit: 1st sload
176:        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]); // @audit 2nd sload
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L190-L195

### Cache `LUSDDebt[_collateral].add(_amount)` as a stack variable and emit cached stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
190:    function increaseLUSDDebt(address _collateral, uint _amount) external override {
191:        _requireValidCollateralAddress(_collateral);
192:        _requireCallerIsBOorTroveM();
193:        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount); // @audit 1st sload
194:        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); // @audit 2nd sload
195:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L197-L202

### Cache `LUSDDebt[_collateral].sub(_amount)` as a stack variable and emit cached stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
197:    function decreaseLUSDDebt(address _collateral, uint _amount) external override {
198:        _requireValidCollateralAddress(_collateral);
199:        _requireCallerIsBOorTroveMorSP();
200:        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount); // @audit: 1st sload
201:        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); // @audit: 2nd sload
202:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L204-L212

### Cache `collAmount[_collateral].add(_amount)` as a stack variable and emit cached stack variable to save 1 SLOAD.
```solidity
File: Ethos-Core/contracts/ActivePool.sol
204:    function pullCollateralFromBorrowerOperationsOrDefaultPool(address _collateral, uint _amount) external override {
205:        _requireValidCollateralAddress(_collateral);
206:        _requireCallerIsBorrowerOperationsOrDefaultPool();
207:        collAmount[_collateral] = collAmount[_collateral].add(_amount); // @audit 1st sload
208:        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]); // @audit 2nd sload
209:
210:        IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _amount);
211:        _rebalance(_collateral, 0);
212:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1082-L1108

### Cache `Troves[_borrower][_collateral].coll.add(pendingCollateralReward)` and `Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward)` as a stack variables and emit cached stack variables to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1082:    function _applyPendingRewards(IActivePool _activePool, IDefaultPool _defaultPool, address _borrower, address _collateral) internal {
1083:        if (hasPendingRewards(_borrower, _collateral)) {
1084:            _requireTroveIsActive(_borrower, _collateral);
1085:
1086:            // Compute pending rewards
1087:            uint pendingCollateralReward = getPendingCollateralReward(_borrower, _collateral);
1088:            uint pendingLUSDDebtReward = getPendingLUSDDebtReward(_borrower, _collateral);
1089:
1090:            // Apply pending rewards to trove's state
1091:            Troves[_borrower][_collateral].coll = Troves[_borrower][_collateral].coll.add(pendingCollateralReward); // @audit: 1st sload
1092:            Troves[_borrower][_collateral].debt = Troves[_borrower][_collateral].debt.add(pendingLUSDDebtReward); // @audit: 1st sload
1093:
1094:            _updateTroveRewardSnapshots(_borrower, _collateral);
1095:
1096:            // Transfer from DefaultPool to ActivePool
1097:            _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, pendingLUSDDebtReward, pendingCollateralReward);
1098:
1099:            emit TroveUpdated(
1100:                _borrower,
1101:                _collateral,
1102:                Troves[_borrower][_collateral].debt, // @audit: 2nd sload, emit cached var
1103:                Troves[_borrower][_collateral].coll, // @audit: 2nd sload, emit cached var
1104:                Troves[_borrower][_collateral].stake,
1105:                TroveManagerOperation.applyPendingRewards
1106:            );
1107:        }
1108:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1116-L1120

### Cache `L_Collateral[_collateral]` and `L_LUSDDebt[_collateral]` as a stack variables and emit cached stack variables to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1116:    function _updateTroveRewardSnapshots(address _borrower, address _collateral) internal {
1117:        rewardSnapshots[_borrower][_collateral].collAmount = L_Collateral[_collateral]; // @audit: 1st sload
1118:        rewardSnapshots[_borrower][_collateral].LUSDDebt = L_LUSDDebt[_collateral]; // @audit: 1st sload
1119:        emit TroveSnapshotsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]); // @audit 2nd sloads, emit cached vars
1120:    }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1201-L1210

### Cache `totalStakes[_collateral].sub(oldStake).add(newStake)` as a stack variable and emit cached stack variable to save 1 SLOAD. 
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1201:    function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {
1202:        uint newStake = _computeNewStake(_collateral, Troves[_borrower][_collateral].coll);
1203:        uint oldStake = Troves[_borrower][_collateral].stake;
1204:        Troves[_borrower][_collateral].stake = newStake;
1205:
1206:        totalStakes[_collateral] = totalStakes[_collateral].sub(oldStake).add(newStake); // @audit: 1st sload
1207:        emit TotalStakesUpdated(_collateral, totalStakes[_collateral]); // @audit: 2nd sload, use cached var
1208:
1209:        return newStake;
1210:    }
``` 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1261-L1265

### Cache `L_Collateral[_collateral].add(collateralRewardPerUnitStaked)` and `L_LUSDDebt[_collateral].add(LUSDDebtRewardPerUnitStaked)` as a stack variables and emit cached stack variables to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
...
1261:        // Add per-unit-staked terms to the running totals
1262:        L_Collateral[_collateral] = L_Collateral[_collateral].add(collateralRewardPerUnitStaked); // @audit: 1st sload
1263:        L_LUSDDebt[_collateral] = L_LUSDDebt[_collateral].add(LUSDDebtRewardPerUnitStaked); // @audit: 1st sload
1264:
1265:        emit LTermsUpdated(_collateral, L_Collateral[_collateral], L_LUSDDebt[_collateral]); // @audit: 2nd sloads, emit cached vars
...
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1305-L1313

### Cache `totalStakes[_collateral]` and `activeColl.sub(_collRemainder).add(liquidatedColl)` as a stack variables and emit cached stack variables to save 2 SLOADs.
```solidity
File: Ethos-Core/contracts/TroveManager.sol
1305:    function _updateSystemSnapshots_excludeCollRemainder(IActivePool _activePool, address _collateral, uint _collRemainder) internal {
1306:        totalStakesSnapshot[_collateral] = totalStakes[_collateral]; // @audit: cache `totalStakes[_collateral]` to use for later emit
1307:
1308:        uint activeColl = _activePool.getCollateral(_collateral);
1309:        uint liquidatedColl = defaultPool.getCollateral(_collateral);
1310:        totalCollateralSnapshot[_collateral] = activeColl.sub(_collRemainder).add(liquidatedColl); // @audit cache `activeColl.sub(_collRemainder).add(liquidatedColl)` to use for later emit
1311:
1312:        emit SystemSnapshotsUpdated(_collateral, totalStakesSnapshot[_collateral], totalCollateralSnapshot[_collateral]); // @audit: 2 unecessary sloads, emit cached vars
1313:    }
```

## Refactor code to save 1 SSTORE and 2 SLOADs

In the function below the `totalAllocBPS` state variable undergoes 1 uncessary STORE and 2 unecessary SLOADs. We can cache the final value computed for `totalAllocBPS` in a stack variable (avoid 1st SSTORE) and later use that value in the expression to update `totalAllocBPS` (avoid 2nd SLOAD) and emit the cached variable (avoid 3rd SLOAD).

### Gas Savings for `updateStrategyAllocBPS`, obtained via protocol's tests: Avg 337 gas
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  34567  |  47361  |  41465 |    8     |
| After  |  34230  |  47024  |  41128 |    8     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194-L196


```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
191:    function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {
192:        _atLeastRole(STRATEGIST);
193:        require(strategies[_strategy].activation != 0, "Invalid strategy address");
194:        totalAllocBPS -= strategies[_strategy].allocBPS; // @audit 1st sstore, 1st sload
195:        strategies[_strategy].allocBPS = _allocBPS;
196:        totalAllocBPS += _allocBPS; // @audit: 2nd sstore, 2nd sload
197:        require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value"); // @audit: 3rd sload
198:        emit StrategyAllocBPSUpdated(_strategy, _allocBPS);
199:    }
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -191,10 +191,10 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
     function updateStrategyAllocBPS(address _strategy, uint256 _allocBPS) external {
         _atLeastRole(STRATEGIST);
         require(strategies[_strategy].activation != 0, "Invalid strategy address");
-        totalAllocBPS -= strategies[_strategy].allocBPS;
+        uint256 _totalAllocBPS = totalAllocBPS - strategies[_strategy].allocBPS + _allocBPS;
         strategies[_strategy].allocBPS = _allocBPS;
-        totalAllocBPS += _allocBPS;
-        require(totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
+        totalAllocBPS = _totalAllocBPS;
+        require(_totalAllocBPS <= PERCENT_DIVISOR, "Invalid BPS value");
         emit StrategyAllocBPSUpdated(_strategy, _allocBPS);
     }
```

## `Internal` functions are cheaper to call compared to `public` functions
[When you call a public function, all the parameters are again copied into memory and passed to that function. By contrast, when you call an internal function, references of those parameters are passed and they are not copied into memory again.](https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6)

Total Instances: 13

Gas savings: `13 * 22 = 286`

**POC:** https://gist.github.com/0xJCN/637f37d15d3250a4aa48f23f94b4c60c


https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L278

### In the instance below, the `balance()` function is called multiple times throughout various contracts (i.e. `ReaperVaultV2.sol`, `ReaperVaultERC4626.sol`). Consider creating an internal `_balance()` function to save ~22 gas per call.

An internal `_balance()` function can be created, which would be called by the public `balance()` function. In this case the contracts that inherit from `ReaperVaultV2` can save around ~22 gas per internal call to `_balance()`, while the contracts that do not inherit from `ReaperVaultV2` can call the `public` `balance()` function.
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
278:    function balance() public view returns (uint256) {
279:        return token.balanceOf(address(this)) + totalAllocated;
280:    }
```
Consider refactoring the code to include an internal `_balance()` function as shown below:
```solidity
    function balance() public view returns (uint256) {
        return _balance();
    }
    
    function _balance() internal view returns (uint256) {
        return token.balanceOf(address(this)) + totalAllocated;
    }
```
Below are instances in which ~22 gas would be saved by calling the new internal `_balance()` function:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L231

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L237

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L288

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L323
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
231:       uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;

237:       uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;

288:       return balance() - _calculateLockedProfit();

323:       uint256 pool = balance();
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L38

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L80

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L82

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L123

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol
38:        return balance();

80:        if (emergencyShutdown || balance() >= tvlCap) return 0;

82:        return tvlCap - balance();

123:       if (emergencyShutdown || balance() >= tvlCap) return 0;

125:       return convertToShares(tvlCap - balance());
```

### The following functions can also be refactored in a similar way to the `balance` function shown above.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol
51:    function convertToShares(uint256 assets) public view override returns (uint256 shares) {
52:        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
53:        return (assets * totalSupply()) / _freeFunds();
54:    }

66:    function convertToAssets(uint256 shares) public view override returns (uint256 assets) {
67:        if (totalSupply() == 0) return shares;
68:        return (shares * _freeFunds()) / totalSupply();
69:    }
```
Below are instances that would benefit from calling the `internal` functions rather than the currrent `public` versions:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L97

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L125

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L166

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L241

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol
97:         return convertToShares(assets);

125:        return convertToShares(tvlCap - balance());

166:        return convertToAssets(balanceOf(owner));

241:        return convertToAssets(shares);
```

Check out [OpenZeppelin's Implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol#L199-L206) for an example.

## `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables

Total Instances: 13

Gas savings: `13 * 18 = 234`

### Gas Savings for `addStrategy()`, obtained via protocol's tests: Avg 16 gas
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  170982  |  205182  |  182382 |    3     |
| After  |  170966  |  205166  |  182366 |    3     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol#L168
168:  totalAllocBPS += _allocBPS;
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -165,7 +165,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
             lastReport: block.timestamp
         });

-        totalAllocBPS += _allocBPS;
+        totalAllocBPS = totalAllocBPS + _allocBPS;
         withdrawalQueue.push(_strategy);
         emit StrategyAdded(_strategy, _feeBPS, _allocBPS);
     }
```

### Gas Savings for `revokeStrategy()`, obtained via protocol's tests: Avg 21 gas
|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  31639  |  33557  |  32515 |    6     |
| After  |  31619  |  33537  |  32494 |    6     |

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol#L214
214:  totalAllocBPS -= strategies[_strategy].allocBPS;
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -211,7 +211,7 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
             return;
         }

-        totalAllocBPS -= strategies[_strategy].allocBPS;
+        totalAllocBPS = totalAllocBPS - strategies[_strategy].allocBPS;
         strategies[_strategy].allocBPS = 0;
         emit StrategyRevoked(_strategy);
     }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395-L396

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
395:            strategies[stratAddr].allocated -= actualWithdrawn;
396:            totalAllocated -= actualWithdrawn;
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -392,8 +392,8 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
                     _reportLoss(stratAddr, loss);
                 }

-                strategies[stratAddr].allocated -= actualWithdrawn;
-                totalAllocated -= actualWithdrawn;
+                strategies[stratAddr].allocated = strategies[stratAddr].allocated - actualWithdrawn;
+                totalAllocated = totalAllocated - actualWithdrawn;
             }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L452

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
442:           // If the loss is too small, bpsChange will be 0
443:           if (bpsChange != 0) {
444:               stratParams.allocBPS -= bpsChange;
445:               totalAllocBPS -= bpsChange;
446:           }
447:       }
448:
449:      // Finally, adjust our strategy's parameters by the loss
450:      stratParams.losses += loss;
451:      stratParams.allocated -= loss;
452:      totalAllocated -= loss;
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -441,15 +441,15 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont

             // If the loss is too small, bpsChange will be 0
             if (bpsChange != 0) {
-                stratParams.allocBPS -= bpsChange;
-                totalAllocBPS -= bpsChange;
+                stratParams.allocBPS = stratParams.allocBPS - bpsChange;
+                totalAllocBPS = totalAllocBPS - bpsChange;
             }
         }

         // Finally, adjust our strategy's parameters by the loss
-        stratParams.losses += loss;
-        stratParams.allocated -= loss;
-        totalAllocated -= loss;
+        stratParams.losses = statParams.losses + loss;
+        stratParams.allocated = stratParams.allocated - loss;
+        totalAllocated = totalAllocated - loss;
     }
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L514-L521

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol
513:            if (vars.debtPayment != 0) {
514:                strategy.allocated -= vars.debtPayment;
515:                totalAllocated -= vars.debtPayment;
516:                vars.debt -= vars.debtPayment; // tracked for return value
517:            }
518:        } else if (vars.available > 0) {
519:            vars.credit = uint256(vars.available);
520:            strategy.allocated += vars.credit;
521:            totalAllocated += vars.credit; 
522:        }
```
```diff
diff --git a/Ethos-Vault/contracts/ReaperVaultV2.sol b/Ethos-Vault/contracts/ReaperVaultV2.sol
index b5f2a58..8eb2f8b 100644
--- a/Ethos-Vault/contracts/ReaperVaultV2.sol
+++ b/Ethos-Vault/contracts/ReaperVaultV2.sol
@@ -511,14 +511,14 @@ contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessCont
             if (vars.debtPayment != 0) {
-                strategy.allocated -= vars.debtPayment;
-                totalAllocated -= vars.debtPayment;
+                strategy.allocated = strategy.allocated - vars.debtPayment;
+                totalAllocated = totalAllocated - vars.debtPayment;
                 vars.debt -= vars.debtPayment; // tracked for return value
             }
         } else if (vars.available > 0) {
             vars.credit = uint256(vars.available);
-            strategy.allocated += vars.credit;
-            totalAllocated += vars.credit;
+            strategy.allocated = strategy.allocated + vars.credit;
+            totalAllocated = totalAllocated + vars.credit;
         }
```