
# GAS OPTIMIZATIONS

##

### [G-1] State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

> Instances (2) :

> Approximate gas saved : 100000 gas for 5 SLOTS 

if 4 slots 80000 gas saved 

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     35:  mapping(address => StrategyParams) public strategies; // (32)
     42: uint256 public tvlCap; // (32)
     44: uint256 public totalAllocBPS;  // (32)
     45: uint256 public totalAllocated;  // (32)
     46: uint256 public lastReport;  //(32)
     49:  bool public emergencyShutdown;  //(1)
     55:  uint256 public withdrawMaxLoss = 1;  // (32)
     56:  uint256 public lockedProfitDegradation; // (32)
     57:  uint256 public lockedProfit;   // (32)
     78:  address public treasury;  // (20)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L35-L78)

Variables ordering with 10 slots. 

The variable withdrawMaxLoss stores a value of 1 and there is no place in the contract where a higher value is stored for this variable. Therefore, it is safe to change the type of withdrawMaxLoss to uint64 can save one slot 

      /// @Audit Variable ordering with 8 slots instead of the current 10 slots:

      /// withdrawalQueue, Mapping(32):strategies, uint256(32) : tvlCap, uint256(32) : totalAllocBPS, uint256(32) 
          : totalAllocated, uint256(32) : lastReport,uint256(32) : lockedProfitDegradation, uint256(32) : 
          lockedProfit, uint64 (8): withdrawMaxLoss , bool(1) : emergencyShutdown, address(20 ) : treasury
        address[] public withdrawalQueue; 
        mapping(address => StrategyParams) public strategies; // (32)
        uint256 public tvlCap; // (32)
        uint256 public totalAllocBPS;  // (32)
        uint256 public totalAllocated;  // (32)
        uint256 public lastReport;  //(32)
        uint256 public lockedProfitDegradation; // (32)
        uint256 public lockedProfit;   // (32)
        uint64 public withdrawMaxLoss = 1;  // (8)
        bool public emergencyShutdown;  // (1)
        address public treasury;  // (20)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L35-L78)

> So reduce 2 Gsset operations, you can save around 40,000 gas (2 x 20,000 gas). 

#### If the sponsor still feels that withdrawMaxLoss should be declared as uint256, then we can order the variables with 9 slots in the same order . 

        uint256 public lockedProfit;   // (32)
        uint256 public withdrawMaxLoss = 1;  // (32)
        bool public emergencyShutdown;  // (1)
        address public treasury;  // (20)

File : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

    49: uint256 public yieldingPercentageDrift = 100; // rebalance iff % is off by more than 100 BPS

    // Yield distribution params, must add up to 10k
    52:  uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
    53:  uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
    54:  uint256 public yieldSplitStaking = 40_00;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L49-L54)

> Currently variable ordering with 4 slots 

It is correct to declare the variables yieldingPercentageDrift, yieldSplitTreasury, yieldSplitSP, and yieldSplitStaking as uint64 instead of uint256 since their values are not expected to exceed the maximum value of uint64. Doing so would help save some gas costs

uint64 can store values ranging from 0 to 18,446,744,073,709,551,615

The values of the variables yieldingPercentageDrift, yieldSplitTreasury, yieldSplitSP, and yieldSplitStaking are not increased anywhere in the contract. Therefore, uint64 is more than enough to store their values. Based on the contract implementation, there are no possibilities for these variables to overflow

Recommended Mitigations : 

    /// @audit Variable ordering with 1 slot instead of the current 4:
    ///    uint64(8) : yieldingPercentageDrift , uint64(8) : yieldSplitTreasury , uint64(8) : yieldSplitSP , uint64(8) : yieldSplitStaking 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L49-L54)

##

### [G-2]  Structs can be packed into fewer storage slots

> Instances (1) : 

> Total Slots Saved - 3

> Approximate gas saved : 60000 gas

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

DeFi tokens typically use up to 18 decimal places, which is the standard for Ethereum-based tokens such as ERC20 and ERC777. This is because using more decimal places can lead to precision issues and make calculations more complicated.

The variable decimals can be safely declared as uint128 instead of uint256 since uint128 is enough to store up to 18 decimal places. Therefore, using uint128 for decimals would be more efficient and save gas costs . The maximum value stored in uint128 is 340,282,366,920,938,463,463,374,607,431,768,211,455. This value is more than enough for decimals . 

        
        /// @audit Variable ordering with 3 slots instead of the current 4:
        ///  uint256(32): MCR, uint256(32) : CCR, uint128(16) : decimals, bool(1) : allowed
         27:  struct Config {
         28:  bool allowed; 
         29:  uint256 decimals;
         30:  uint256 MCR;
         31:  uint256 CCR;
         32:   }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L27-L32)

FILE : https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol

        struct LocalVariables_adjustTrove {
        uint256 collCCR; //32
        uint256 collMCR; //32
        uint256 collDecimals; //32
        uint price; //32
        uint collChange; //32
        uint netDebtChange; //32
        bool isCollIncrease; //1
        uint debt; //32
        uint coll; //32
        uint oldICR; //32
        uint newICR; //32
        uint newTCR;//32
        uint LUSDFee; //32
        uint newDebt; //32
        uint newColl; //32
        uint stake; //32
    }
 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L47-L64)     

> Currently total 16 slots 

  collCCR,collMCR, LUSDFee we can declare uint128 instead of uint266. The collCCR,collMCR always within the particular range. Once set values not increased as per documentations. Only possible to decrease the collCCR,collMCR values so for those situations uint128 is alone enough. There is no chance to overflow .

LUSDfee also each trove has the debt limit. The fee percent is very less which is always between 1-2%from overall debt. So uint128 is more than enough to store LUSDfee 



        /// @audit Variable ordering with 14 slots instead of the current 16:
        struct LocalVariables_adjustTrove {
       
        uint256 collDecimals; //32
        uint price; //32
        uint collChange; //32
        uint netDebtChange; //32
        uint debt; //32
        uint coll; //32
        uint oldICR; //32
        uint newICR; //32
        uint newTCR;//32
        uint newDebt; //32
        uint newColl; //32
        uint stake; //32
        uint128 collCCR; //16
        uint128 collMCR; //16
        uint128 LUSDFee; //16   
        bool isCollIncrease; //1
        
       }
   

##        

### [G-3] Optimize names to save gas

public/external function names and public member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per [sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

> Instances (7) :

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

      /// @Audit updateCollateralRatios()
      15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L15)

File: 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      
        /// @Audit 
setAddresses(),setYieldingPercentage(),setYieldingPercentageDrift(),setYieldClaimThreshold(),setYieldDistributionParams(),manualRebalance(),
      26 : contract ActivePool is Ownable, CheckContract, IActivePool {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L26)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

      /// @Audit fund(),updateDistributionPeriod()
     14:  contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L14)

File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      /// @Audit pauseMinting(),unpauseMinting(),updateGovernance(),updateGuardian(),upgradeProtocol(),
      28 :  contract LUSDToken is CheckContract, ILUSDToken {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L28)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     /// @Audit addStrategy(),updateStrategyFeeBPS(),updateStrategyAllocBPS(),revokeStrategy(),availableCapital(),setWithdrawalQueue(),balance(),getPricePerFullShare(),depositAll(),deposit(),withdrawAll(),withdraw(),report(),updateWithdrawMaxLoss(),updateTvlCap(),removeTvlCap(),setEmergencyShutdown(),setLockedProfitDegradation(),updateTreasury(),inCaseTokensGetStuck(),
     21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L21)

File : 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

      /// @Audit setEmergencyExit(),initiateUpgradeCooldown(),clearUpgradeCooldown(),
      14: abstract contract ReaperBaseStrategyv4 is

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L14)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

     /// @Audit setHarvestSteps(),authorizedWithdrawUnderlying(),balanceOfWant(),balanceOfPool()
     19:  contract ReaperStrategyGranarySupplyOnly is ReaperBaseStrategyv4, VeloSolidMixin {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L19)

 ##

### [G-4] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations


File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        86 :  mapping (address => mapping (address => Trove)) public Troves;

        88 : mapping (address => uint) public totalStakes;

        91 : mapping (address => uint) public totalStakesSnapshot;

        94:   mapping (address => uint) public totalCollateralSnapshot;

        104:  mapping (address => uint) public L_Collateral;
        105:  mapping (address => uint) public L_LUSDDebt;

        109 :  mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;

        116:  mapping (address => address[]) public TroveOwners;

        119 : mapping (address => uint) public lastCollateralError_Redistribution;
        120:   mapping (address => uint) public lastLUSDDebtError_Redistribution;
    
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L86-L109)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L116-L120)

File :  File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      54 :  mapping (address => uint256) private _nonces;

      57:    mapping (address => uint256) private _balances;
      58:    mapping (address => mapping (address => uint256)) private _allowances;  

      62:    mapping (address => bool) public troveManagers;
      63:    mapping (address => bool) public stabilityPools;
      64:   mapping (address => bool) public borrowerOperations;

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L54-L64)

File : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

       41:   mapping (address => uint256) internal collAmount;  // collateral => amount tracker
       42:   mapping (address => uint256) internal LUSDDebt;  // collateral => corresponding debt tracker

       44:   mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)
       45:   mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming
       46:   mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault
       47:   mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute

  (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L41-L47) 

File :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

       167:    mapping (address => uint256) internal collAmounts;  // deposited collateral tracker
       186:    mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct
       187:    mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct
        214:   mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;
        223:   mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
        228:   mapping (address => uint) public lastCollateralError_Offset;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L186-L187)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

       25:  mapping( address => uint) public stakes;
       28:  mapping (address => uint) public F_Collateral; 
       32:  mapping (address => Snapshot) public snapshots; 
     
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L25-L32)

##

### [G-5]  USE A MORE RECENT VERSION OF SOLIDITY

> Instances (12) :

CONTEXT:
ALL CONTRACTS

Using the latest version of a smart contract can help you reduce gas costs because the latest version is usually optimized for efficiency and can use fewer gas fees to execute the same function compared to an older version. In addition, the latest version may include new features that can further optimize gas usage, such as the use of newer algorithms or data structures.

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

File : CollateralConfig.sol

        3 :  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3)

File: 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

      3:  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3)

File :  2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        3:  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L3)

File: 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      3: pragma solidity 0.6.11;

File :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

      3:  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L3)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

      3:  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)

File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

     3: pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L3)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     3: pragma solidity ^0.8.0;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

      3:  pragma solidity ^0.8.0;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)

File : 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

      3: pragma solidity ^0.8.0;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

     3:  pragma solidity ^0.8.0;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

##

### [G-6] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

In Solidity, declaring a variable inside a loop can result in higher gas costs compared to declaring it outside the loop. This is because every time the loop runs, a new instance of the variable is created, which can lead to unnecessary memory allocation and increased gas costs

On the other hand, if you declare a variable outside the loop, the variable is only initialized once, and you can reuse the same instance of the variable inside the loop. This approach can save gas and optimize the execution of your code

> Instances (10) :

> Approximate Gas Saved : 90 gas

GAS SAMPLE TEST IN remix ide(Without gas optimizations) :

Before Mitigation :

     function testGas() public view {

        for(uint256 i = 0; i < 10; i++) {
          address collateral = msg.sender;
          address  collateral1 = msg.sender;
            
        }

The execution Cost : 2176 

After Mitigation :

     function testGas() public view {
    address collateral; address  collateral1;
        for(uint256 i = 0; i < 10; i++) {
         collateral = msg.sender;
         collateral1 = msg.sender;
            
        }

The execution Cost : 2086 . 

>  Hence clearly after mitigation the gas cost is reduced. Approximately its possible to save 9 gas for every iterations 


File : CollateralConfig.sol

            for(uint256 i = 0; i < _collaterals.length; i++) {
            address collateral = _collaterals[i];  //@Audit Declared inside the loop. This should be avoided . 
            Should declare outside the loop
            checkContract(collateral);
            collaterals.push(collateral);

            Config storage config = collateralConfig[collateral];   //@Audit Declared inside the loop. This should 
            be avoided . Should declare outside the 
           loop
            config.allowed = true;
            uint256 decimals = IERC20(collateral).decimals();
            config.decimals = decimals;

            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
            config.MCR = _MCRs[i];

            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
            config.CCR = _CCRs[i];

            emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
          }
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56-L73)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
            // we need to cache it, because current user is likely going to be deleted
            address nextUser = _sortedTroves.getPrev(_collateral, vars.user);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L608-L612)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L819)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L108-L113)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351-L357)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206-L209)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L228-L231)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L240-L242)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L117-L121)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

           for (uint256 i = 0; i < queueLength; i = i.uncheckedInc()) {
            address strategy = _withdrawalQueue[i];
            StrategyParams storage params = strategies[strategy];

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L264-L266)

##

### [G-7]  USE FUNCTION INSTEAD OF MODIFIERS

Functions can have local variables. When a modifier is used, all of the variables used in the modifier are stored in memory for the entire duration of the function call, even if they are not used after the modifier. This can result in unnecessary gas costs. With a function, you can declare local variables that are only stored in memory for the duration of the function call, which can save on gas costs

 Modifier  can make it harder to optimize the code for gas efficiency. Functions, on the other hand, are more straightforward and easier to optimize for gas

File : CollateralConfig.sol

       modifier checkCollateral(address _collateral) {
        require(collateralConfig[_collateral].allowed, "Invalid collateral address");
        _;
       }

This modifier can be replaced with function like below :

    function checkCollateral(address _collateral) private view returns(bool) {
    return collateralConfig[_collateral].allowed ;
}

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L128-L132)

##

### [G-8]  BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

bytes32 is a fixed-size data type, meaning that its size is known at compile time and cannot change during runtime. This allows Solidity to allocate the exact amount of memory needed for a bytes32 variable, which can be more efficient than allocating memory dynamically for a string variable

Because string variables are dynamically-sized and can grow or shrink during runtime, performing operations on them can be more expensive in terms of gas costs than operations on fixed-size data types like bytes32.

> Instances (8) :

PROOF OF CONCEPT :

IDE USED REMIX (Without Optimization) : 

> USING STRING :

pragma solidity ^0.8.17;

contract StorageTest {

 string constant public NAME = "BorrowerOperations";

}

Remix Reports : 

gas                :  152195 gas
transaction cost   :  132343 gas 
execution cost	   :   73523 gas 


> USING BYTES32 :

pragma solidity ^0.8.17;

contract StorageTest {

bytes32 constant public NAME = bytes32("BorrowerOperations");

}

           gas	               : 113157 gas
           transaction cost    : 98397 gas 
           execution cost      : 41893 gas

GAS SAVED :

       gas                  : 39038
       transaction cost     : 33946
       execution cost	    : 31630

File :  BorrowerOperations.sol


       21 : string constant public NAME = "BorrowerOperations";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

     30 :  string constant public NAME = "ActivePool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30)

FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

     150 :  string constant public NAME = "StabilityPool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150)

FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

      19:    string constant public NAME = "CommunityIssuance";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)

 FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

     23 :  string constant public NAME = "LQTYStaking";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)

FILE : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      32 : string constant internal _NAME = "LUSD Stablecoin";
      33:  string constant internal _SYMBOL = "LUSD";
      34:  string constant internal _VERSION = "1";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L32-L34)

##

### [G-9]  USE REQUIRE INSTEAD OF ASSERT

When a require statement fails, any gas spent after the failing condition is refunded to the caller. This can help prevent unnecessary gas usage and make the contract more efficient. On the other hand, when an assert statement fails, all remaining gas is consumed and cannot be refunded, which can be wasteful and lead to more expensive transactions

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

> Instances (12)

File : BorrowerOperations.sol

      128:  assert(MIN_NET_DEBT > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L128)

      197:  assert(vars.compositeDebt > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L197)

     331:  assert(_collWithdrawal <= vars.coll); 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L331)


FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

     1281 :  assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1279)

     1342 :  assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

     1348 :  assert(index <= idxLast);

File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      312:  assert(sender != address(0));
      313:  assert(recipient != address(0));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L312-L313)

       321:   assert(account != address(0));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L321)

      329:   assert(account != address(0));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L329)

        337:  assert(owner != address(0));
        338:  assert(spender != address(0));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L337-L338)

##

### [G-10]  USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

 When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

> Instances  (11) 

File :  BorrowerOperations.sol

     175 :  ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L175)

     283 :  ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L283)

  (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L301)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

         313 :  address[] memory borrowers = new address[](1);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L313)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

        166:   address[2] memory step = _newSteps[i];

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     660 : bytes32[] memory cascadingAccessRoles = new bytes32[](5);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L660)

 File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

     166:  address[2] memory step = _newSteps[i];

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L166)

File : 2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

     687:  Snapshots memory snapshots = depositSnapshots[_depositor];

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L687) 

    722:  Snapshots memory snapshots = depositSnapshots[_depositor];

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L722)

        806:  address[] memory collaterals = collateralConfig.getAllowedCollaterals();
        807:  uint[] memory amounts = new uint[](collaterals.length);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L806-L807)


##

### [G-11] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

External functions do not require a context switch to the EVM, while public functions do. 

Its possible to save 10-15 gas using external instead public for every function call 


> Instances (1) :

Approximate Saved Gas : 10-15 gas 

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      1045 :  function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1045)

##

### [G-12]  INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

> Instances (35) :

> Approximate Saved Gas : 1050

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

        function _getCollChange(
        uint _collReceived,
        uint _requestedCollWithdrawal
        )
        internal
        pure
        returns(uint collChange, bool isCollIncrease)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L438-L444)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L476-L488)

       524:  function _requireValidCollateralAddress(address _collateral) internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L524)

       533:  function _requireSingularCollChange(uint _collTopUp, uint _collWithdrawal) internal pure {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L533)

      537:  function _requireNonZeroAdjustment(uint _collTopUp, uint _collWithdrawal, uint _LUSDChange) internal pure {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L537)

     546 :  function _requireTroveisNotActive(ITroveManager _troveManager, address _borrower, address _collateral) internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L546)

      551:  function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L551)

   (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L555-L561)

  (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L567)

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L571-L581)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L624)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L636)

       640 : function _requireCallerIsStabilityPool() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L661-L673)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478-L488)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L582-L594)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L671-L682)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L898-L913)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L269)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L265)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L261)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L251)

File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

    320: function _mint(address account, uint256 amount) internal {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L320)

      328:  function _burn(address account, uint256 amount) internal {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L328)
   
      360 : function _requireCallerIsBorrowerOperations() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L360)

     365:  function _requireCallerIsBOorTroveMorSP() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L365)

     375: function _requireCallerIsStabilityPool() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L375)
 
    380:   function _requireCallerIsTroveMorSP() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L380)

  393: function _requireMintingNotPaused() internal view {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L393)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

        function _liquidatePosition(uint256 _amountNeeded)
        internal
        override
        returns (uint256 liquidatedAmount, uint256 loss)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86-L89)

      176:  function _deposit(uint256 toReinvest) internal {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L176)

     206: function _claimRewards() internal {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L206)


### [G-13] SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS

Instead of using the && operator in a single require statement to check multiple conditions, using multiple require statements with 1 condition per require statement will save 30 GAS per &&

> Instances (4)

Approximate Saved Gas : 120 gas

PROOF OF WORK (remix without optimizations) :

(https://github.com/sathishpic22/AllGASTestings/blob/main/Require%26%26splitTest)

pragma solidity ^0.8.7;
contract MappingTest {

int a=10;
    
    // Execution Cost 2398
    function requireTest()public view {

        require(a==10 && a<11,"wrong inputs");
    }
    // Execution Cost 2368
    function requireTest1()public view {
        require(a==10,"wrong inputs");
        require(a<11 ,"wrong inputs");
    }

}

So when splitting require then save 30 gas 

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol


        require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
        "Max fee percentage must be between 0.5% and 100%");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)

File : File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

        require(
            _recipient != address(0) && 
            _recipient != address(this),
            "LUSD: Cannot transfer tokens directly to the LUSD token contract or the zero address"
         );

         require(
            !stabilityPools[_recipient] &&
            !troveManagers[_recipient] &&
            !borrowerOperations[_recipient],
            "LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps"
          );

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L347-L357)

##
     
### [G-14] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

> Instances (13) :

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        function _liquidateNormalMode(
        IActivePool _activePool,
        IDefaultPool _defaultPool,
        address _collateral,
        address _borrower,
        uint _LUSDInStabPool
        )
        internal
        returns (LiquidationValues memory singleLiquidation)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L321-L353)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1321-L1333)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

      function _liquidateAllPositions() internal override returns (uint256 amountFreed) {
        _withdrawUnderlying(balanceOfPool());
        return balanceOfWant();
      }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104-L107)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

      function asset() external view override returns (address assetTokenAddress) {
        return address(token);
      }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29-L31)
  

     function totalAssets() external view override returns (uint256 totalManagedAssets) {
        return balance();
     }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37-L39)

     function convertToShares(uint256 assets) public view override returns (uint256 shares) {
        if (totalSupply() == 0 || _freeFunds() == 0) return assets;
        return (assets * totalSupply()) / _freeFunds();
     }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51-L54)

      66 :  function convertToAssets(uint256 shares) public view override returns (uint256 assets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66-L69)

      79 :  function maxDeposit(address) external view override returns (uint256 maxAssets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)

      function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
        return convertToShares(assets);
     }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96-L98)

      122 :  function maxMint(address) external view override returns (uint256 maxShares) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)

     165 : function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165)

      220 :  function maxRedeem(address owner) external view override returns (uint256 maxShares) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
     
      240 :  function previewRedeem(uint256 shares) external view override returns (uint256 assets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)

##

### [G-15] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 200-210 gas as per remix 

The unchecked keyword was introduced in Solidity version 0.8.0. Prior to that version. 

> Instances (8) :

> Approximate Saved Gas :  1600 gas 

PROOF OF CONCEPT (remix without optimizations):

pragma solidity ^0.8.7;
contract MappingTest {

// Execution Cost 696
function Subtraction() public pure {
int a=20; int b=10; int c=10; int d=30;
    if(a>b){
        a=a-b;
    }
    if(c<d){
        c=d-c;
    }
   
}
//Execution Cost 276
function uncheckedSubtraction() public pure {
int a=20; int b=10; int c=10; int d=30;
    if(a>b){
        unchecked{a=a-b;}
        
    }
    if(c<d){
        unchecked{c=d-c;}
    }
   
}


}

So clearly for each unchecked possible to save 210 gas 

SOLUTION:

 require(a < b); x = b - a => require(a <b); unchecked { x = b - a } 

require(a > b); x = a - b => require(a > b); unchecked { x = a - b } 


File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

       80:   if (wantBalance > _debt) {
       81:   uint256 toReinvest = wantBalance - _debt;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L80-L81)

        92:   if (wantBal < _amountNeeded) {
        93:   _withdraw(_amountNeeded - wantBal);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L92-L93)

       99:    if (_amountNeeded > liquidatedAmount) {
       100:  loss = _amountNeeded - liquidatedAmount;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L99-L100)

      131:  if (totalAssets > allocated) {
      132:  uint256 profit = totalAssets - allocated;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L131-L132)

       135:  } else if (totalAssets < allocated) {
       136:  roi = -int256(allocated - totalAssets);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L135-L136)

File : 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

                if (amountFreed < debt) {
                roi = -int256(debt - amountFreed);
              } else if (amountFreed > debt) {
                roi = int256(amountFreed - debt);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L120-L123)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

             234: if (stratCurrentAllocation > stratMaxAllocation) {
             235:  return -int256(stratCurrentAllocation - stratMaxAllocation);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L234-L235)

             330 : _amount = balAfter - balBefore;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L334)

##

### [G-16]  USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size

(https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Each operation involving a uint128 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint128, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

> Instances (3) :

Approximate Saved Gas : 84 gas 

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

    1329 :  index = uint128(TroveOwners[_collateral].length.sub(1));

    1344 :   uint128 index = Troves[_borrower][_collateral].arrayIndex;

File : File : 2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

    745:  uint128 scaleDiff = currentScale.sub(scaleSnapshot);

##

### [G-17] SUPERFLUOUS EVENT FIELDS

block.timestamp are added to event information by default so adding them manually wastes gas

> Instances (1)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      1505 :  emit LastFeeOpTimeUpdated(block.timestamp);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1505)

##

### [G-18] <x> += <y> OR <x>-=<y> costs more gas than <x> = <x> + <y> OR  <x> = <x> - <y>   for state variables

Using the addition/Subtraction operator instead of plus-equals/minus-equals saves 113 gas

> Instances (14)

> Approximate Saved Gas : 1500 gas 

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     168 : totalAllocBPS += _allocBPS;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
 
    194:  totalAllocBPS -= strategies[_strategy].allocBPS;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)

    195: totalAllocBPS += _allocBPS;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)

    214:  totalAllocBPS -= strategies[_strategy].allocBPS;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)

     395:   strategies[stratAddr].allocated -= actualWithdrawn;
     396:   totalAllocated -= actualWithdrawn;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L395-L396)

    444:  stratParams.allocBPS -= bpsChange;
    445:  totalAllocBPS -= bpsChange;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L445)

        450 : stratParams.losses += loss;
        451:  stratParams.allocated -= loss;
        452:  totalAllocated -= loss;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L450-L452)

       505 : strategy.gains += vars.gain;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L505)

     514:  strategy.allocated -= vars.debtPayment;
     515:  totalAllocated -= vars.debtPayment;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L514-L515)

##

### [G-19]  initialize() function as external can also help to reduce gas costs, since external functions are generally cheaper to call than public or internal functions

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

        function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
        ) public initializer {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L67)

##

### [G-20] MULTIPLE ACCESSES OF A MAPPING/ARRAY SHOULD USE A LOCAL VARIABLE CACHE

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata

Instances (1)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

strategies[_strategy].allocBPS use local variable cache instead calling multiple times 

   210 : if (strategies[_strategy].allocBPS == 0) {

   214:  totalAllocBPS -= strategies[_strategy].allocBPS;

   215:  strategies[_strategy].allocBPS = 0;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L210-L216)

## 

### [G-21] SETTING THE CONSTRUCTOR TO PAYABLE

Saves ~13 gas per instance

> Instances (6) :

> Approximate Saved Gas : 78

File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      225: constructor() public {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L225)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

     57: constructor() public {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57)

File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      84:   constructor(

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L84-L92)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

     111 :     constructor(

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L119)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

     16:  constructor(

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16-L24)

File : 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

     61 :     constructor() initializer {}

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61)

##

### [G-22] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything

> Instances (7)

File :2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

_OathAmount  is not checked for nonzero values before transfer()

     125:  OathToken.transfer(_account, _OathAmount);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L124-L128)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

> LQTYToWithdraw is not checked for nonzero values before safeTransfer()

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L153-L163)

> LUSDGain is not checked for nonzero values before transfer()

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L171)

amounts[i] is not checked for nonzero values before safeTransfer()

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L244)

File : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

_amount  is not checked for nonzero values before calling functions

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L179-L186)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L210)  

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

_collChange is not checked for nonzero values before calling safeTransferFrom()

 497: IERC20(_collateral).safeTransferFrom(msg.sender, address(this), _collChange);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L497)

##

### [G-23] CACHING GLOBAL VARIABLES IS MORE EXPENSIVE THAN USING THE ACTUAL VARIABLE(USE MSG.SENDER INSTEAD OF CACHING IT)

It’s cheaper to use msg.sender as compared to caching

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

> USE msg.sender INSTEAD OF CACHING WITH stratAddr  VARIABLE 

        function availableCapital() public view returns (int256) {
        address stratAddr = msg.sender; // @Audit msg.sender
        if (totalAllocBPS == 0 || emergencyShutdown) {
            return -int256(strategies[stratAddr].allocated);  //@AUDIT USE msg.sender 
         }

         uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;   //@AUDIT USE msg.sender 
         uint256 stratCurrentAllocation = strategies[stratAddr].allocated;   //@AUDIT USE msg.sender 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L225-L232)

> USE msg.sender INSTEAD OF CACHING WITH vars.stratAddr  VARIABLE 


        495 :   vars.stratAddr = msg.sender;

        496:   StrategyParams storage strategy = strategies[vars.stratAddr];
   
        501 :  _reportLoss(vars.stratAddr, vars.loss);
 
        504 :  vars.fees = _chargeFees(vars.stratAddr, vars.gain);

        526 :  token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);

        528 :  token.safeTransferFrom(vars.stratAddr, address(this), vars.freeWantInStrat - vars.credit);

        544:     vars.stratAddr,

        556:  return IStrategy(vars.stratAddr).balanceOf();

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L493-L560)

##

### [G-24]  REORDER THE REQUIRE STATEMENTS TO HAVE THE LESS GAS CONSUMING BEFORE THE EXPENSIVE ONE


File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

          150: require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
          151: require(_strategy != address(0), "Invalid strategy address");
          152: require(strategies[_strategy].activation == 0, "Strategy already added");
          153: require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
          154: require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
          155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
          156: require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");


Its cheaper to check for _strategy != address(0), _feeBPS <= PERCENT_DIVISOR / 5,_allocBPS + totalAllocBPS <= PERCENT_DIVISOR  as compared to  !emergencyShutdown,strategies[_strategy].activation == 0,address(this) == IStrategy(_strategy).vault(),address(token) == IStrategy(_strategy).want() as this involves reading the storage variable. Therefore if the require(_strategy != address(0), "Invalid strategy address");  fails it would be cheaper to fail before evaluating the !emergencyShutdown

First check local variable condition checks then move to state variable condition checks .

          +151: require(_strategy != address(0), "Invalid strategy address");
          +155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
          +156: require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value"); 
          150: require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
          -151: require(_strategy != address(0), "Invalid strategy address");
          152: require(strategies[_strategy].activation == 0, "Strategy already added");
          153: require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
          154: require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
          -155: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
          -156: require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value"); 


(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L150-L156)

> CHECK _feeBPS <= PERCENT_DIVISOR / 5 CONDITION FIRST THEN strategies[_strategy].activation != 0

                 180:  require(strategies[_strategy].activation != 0, "Invalid strategy address");
                 181:  require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L180-L181)

##

### [G-25]  DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

This saves deployment gas


File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

!emergencyShutdown, _strategy].activation != 0, _feeBPS <= PERCENT_DIVISOR / 5, strategies[_strategy].activation != 0 can be REFACTORED TO A MODIFIER OR FUNCTION

      150: require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
      321: require(!emergencyShutdown, "Cannot deposit during emergency shutdown");

  
      180: require(strategies[_strategy].activation != 0, "Invalid strategy address");
      193: require(strategies[_strategy].activation != 0, "Invalid strategy address");
  

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L152)       

        155:  require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        181: require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L155)

##

### [26]  EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

Instances (8) :

Using immutable variables instead of constant variables for expressions that evaluate to constant values, such as a call to keccak256(), can potentially save gas in smart contracts

The reason for this is that immutable variables are evaluated at compile time and the resulting values are hardcoded into the bytecode of the contract. This means that the gas cost of evaluating the expression is incurred only once, during contract compilation, and not at runtime

In contrast, constant variables are evaluated at runtime, which can result in additional gas costs. Specifically, when a constant variable is accessed, the EVM must perform a lookup to retrieve the value of the variable from storage, which can add to the gas cost of executing the contract


File: 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

        49:  bytes32 public constant KEEPER = keccak256("KEEPER");
        50:  bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
        51:  bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
        52:  bytes32 public constant ADMIN = keccak256("ADMIN");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

       73:   bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
       74:   bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
       75:   bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
       76:   bytes32 public constant ADMIN = keccak256("ADMIN");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)

##

### [27] Upgrade Solidity’s optimizer

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.

Set the optimization value higher than 800 in your hardhat.config.ts file

FILE : 2023-02-ethos/Ethos-Core/hardhat.config.js 

                      version: "0.4.23",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
            },
            {
                version: "0.5.17",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
            },
            {
                version: "0.6.11",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 100
                    }
                }
            },
        ]

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/hardhat.config.js#L41-L67)

##

### [28] Use assembly to write address storage values

Instances(49): 

File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol


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

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L266-L278)

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

       collateralConfig = ICollateralConfig(_collateralConfigAddress);
        troveManager = ITroveManager(_troveManagerAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPoolAddress = _stabilityPoolAddress;
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        lqtyStakingAddress = _lqtyStakingAddress;
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L142-L153)

File : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

        collateralConfigAddress = _collateralConfigAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        troveManagerAddress = _troveManagerAddress;
        stabilityPoolAddress = _stabilityPoolAddress;
        defaultPoolAddress = _defaultPoolAddress;
        collSurplusPoolAddress = _collSurplusPoolAddress;
        treasuryAddress = _treasuryAddress;
        lqtyStakingAddress = _lqtyStakingAddress;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L96-L103)

File : 2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

        borrowerOperations = IBorrowerOperations(_borrowerOperationsAddress);
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        troveManager = ITroveManager(_troveManagerAddress);
        activePool = IActivePool(_activePoolAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        communityIssuance = ICommunityIssuance(_communityIssuanceAddress);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L284-L291)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

        OathToken = IERC20(_oathTokenAddress);
        stabilityPoolAddress = _stabilityPoolAddress;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L74-L75)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

        lqtyToken = IERC20(_lqtyTokenAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        troveManagerAddress = _troveManagerAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        activePoolAddress = _activePoolAddress;
        collateralConfig = ICollateralConfig(_collateralConfigAddress);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L86-L91)

##

### [29] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports

Instances (5) : 

File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

   616 : if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { break; }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L616)

   817: if (vars.ICR >= vars.collMCR && vars.remainingLUSDInStabPool == 0) { continue; }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L817)

FILE : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

   311:  if (_isDebtIncrease && !isRecoveryMode) { 

   337:  if (!_isDebtIncrease && _LUSDChange > 0) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L337)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

   264:  if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) > yieldingPercentageDrift) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L264)

Recommendation Code : 

     - if (vars.percentOfFinalBal > vars.yieldingPercentage && vars.percentOfFinalBal.sub(vars.yieldingPercentage) 
       >yieldingPercentageDrift) {

     + if (vars.percentOfFinalBal > vars.yieldingPercentage )
     + if (vars.percentOfFinalBal.sub(vars.yieldingPercentage) 
       > yieldingPercentageDrift)

 


   


    

    

    

   














     
       

      





























