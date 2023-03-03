(1) We can pack structs to smaller size to save storage slot

Structs containing `uint256` should be checked again if it is really necessary to use `uint256` and instead can use a lower value such as `uint128` as it is unlikely there is ever need to pass the max value of `uint128`. By using `uint128` it can save significantly a large margin of gas to all affected structs.
Each storage slot would save about `~2.1k` in gas. Just by changing all `uint256` to `uint128` we can save about `~54` storage slots which total up to `~113,400` in gas savings. The gas savings can increase more if we can allow the use of even smaller `uint` such as `uint96` or `uint64`

```solidity
ActivePool.sol:
  218  
  219      // Due to "stack too deep" error
  220:     struct LocalVariables_rebalance {
  221:         uint256 currentAllocated;
  222:         IERC4626 yieldGenerator;
  223:         uint256 ownedShares;
  224:         uint256 sharesToAssets;
  225:         uint256 profit;
  226:         uint256 finalBalance;
  227:         uint256 percentOfFinalBal;
  228:         uint256 yieldingPercentage;
  229:         uint256 toDeposit;
  230:         uint256 toWithdraw;
  231:         uint256 yieldingAmount;
  232:         uint256 finalYieldingAmount;
  233:         int256 netAssetMovement;
  234:         uint256 treasurySplit;
  235:         uint256 stakingSplit;
  236:         uint256 stabilityPoolSplit;

  //@audit Can save about 8 storage slots

BorrowerOperations.sol:
   47:      struct LocalVariables_adjustTrove {
   48:         uint256 collCCR;
   49:         uint256 collMCR;
   50:         uint256 collDecimals;
   51:         uint price;
   52:         uint collChange;
   53:         uint netDebtChange;
   54:         bool isCollIncrease;
   55:         uint debt;
   56:         uint coll;
   57:         uint oldICR;
   58:         uint newICR;
   59:         uint newTCR;
   60:         uint LUSDFee;
   61:         uint newDebt;
   62:         uint newColl;
   63:         uint stake;
   64:     }

  //@audit Can save about 8 storage slots

   65  
   66:     struct LocalVariables_openTrove {
   67:         uint256 collCCR;
   68:         uint256 collMCR;
   69:         uint256 collDecimals;
   70:         uint price;
   71:         uint LUSDFee;
   72:         uint netDebt;
   73:         uint compositeDebt;
   74:         uint ICR;
   75:         uint NICR;
   76:         uint stake;
   77:         uint arrayIndex;
   78:     }
  //@audit Can save about 5 storage slots

CollateralConfig.sol:
   27:     struct Config {
   28:         bool allowed;
   29:         uint256 decimals;
   30:         uint256 MCR;
   31:         uint256 CCR;
   32:     }
  //@audit Can save about 2 storage slots

StabilityPool.sol:
  178:     struct Snapshots {
  179:         mapping (address => uint) S;
  180:         uint P;
  181:         uint G;
  182:         uint128 scale;
  183:         uint128 epoch;
  184:     }
  185  
    //@audit Can save about 1 storage slot
  ...
  644  
  645      // Due to "stack too deep" error
  646:     struct LocalVariables_getSingularCollateralGain {
  647:         uint256 collDecimals;
  648:         uint128 epochSnapshot;
  649:         uint128 scaleSnapshot;
  650:         uint P_Snapshot;
  651:         uint S_Snapshot;
  652:         uint firstPortion;
  653:         uint secondPortion;
  654:         uint gain;
  655:     }
    //@audit Can save about 3 storage slots

TroveManager.sol:
   75  
   76      // Store the necessary data for a trove
   77:     struct Trove {
   78:         uint debt;
   79:         uint coll;
   80:         uint stake;
   81:         Status status;
   82:         uint128 arrayIndex;
   83:     }
   84  
     //@audit Can save about 1 storage slot

  112:     struct RewardSnapshot { uint collAmount; uint LUSDDebt;}
    //@audit Can save about 1 storage slot

  129:     struct LocalVariables_OuterLiquidationFunction {
  130:         uint256 collDecimals;
  131:         uint256 collCCR;
  132:         uint256 collMCR;
  133:         uint price;
  134:         uint LUSDInStabPool;
  135:         bool recoveryModeAtStart;
  136:         uint liquidatedDebt;
  137:         uint liquidatedColl;
  138:     }
  139  
    //@audit Can save about 4 storage slots

  140:     struct LocalVariables_InnerSingleLiquidateFunction {
  141:         uint collToLiquidate;
  142:         uint pendingDebtReward;
  143:         uint pendingCollReward;
  144:     }
    //@audit Can save about 1 storage slot

  145  
  146:     struct LocalVariables_LiquidationSequence {
  147:         uint remainingLUSDInStabPool;
  148:         uint i;
  149:         uint ICR;
  150:         uint TCR;
  151:         address user;
  152:         bool backToNormalMode;
  153:         uint entireSystemDebt;
  154:         uint entireSystemColl;
  155:         uint256 collDecimals;
  156:         uint256 collCCR;
  157:         uint256 collMCR;
  158:     }
  159  
      //@audit Can save about 4 storage slots

  160:     struct LiquidationValues {
  161:         uint entireTroveDebt;
  162:         uint entireTroveColl;
  163:         uint collGasCompensation;
  164:         uint LUSDGasCompensation;
  165:         uint debtToOffset;
  166:         uint collToSendToSP;
  167:         uint debtToRedistribute;
  168:         uint collToRedistribute;
  169:         uint collSurplus;
  170:     }
  171  
    //@audit Can save about 4 storage slots

  172:     struct LiquidationTotals {
  173:         uint totalCollInSequence;
  174:         uint totalDebtInSequence;
  175:         uint totalCollGasCompensation;
  176:         uint totalLUSDGasCompensation;
  177:         uint totalDebtToOffset;
  178:         uint totalCollToSendToSP;
  179:         uint totalDebtToRedistribute;
  180:         uint totalCollToRedistribute;
  181:         uint totalCollSurplus;
  182:     }    
    //@audit Can save about 4 storage slots

ReaperVaultV2.sol:
   24  
   25:     struct StrategyParams {
   26:         uint256 activation; // Activation block.timestamp
   27:         uint256 feeBPS; // Performance fee taken from profit, in BPS
   28:         uint256 allocBPS; // Allocation in BPS of vault's total assets
   29:         uint256 allocated; // Amount of capital allocated to this strategy
   30:         uint256 gains; // Total returns that Strategy has realized for Vault
   31:         uint256 losses; // Total losses that Strategy has realized for Vault
   32:         uint256 lastReport; // block.timestamp of the last time a report occured
   33:     }
   34  
    //@audit Can save about 3 storage slots
   ..
  471  
  472      // To avoid "stack too deep" errors
  473:     struct LocalVariables_report {
  474:         address stratAddr;
  475:         uint256 loss;
  476:         uint256 gain;
  477:         uint256 fees;
  478:         int256 available;
  479:         uint256 debt;
  480:         uint256 credit;
  481:         uint256 debtPayment;
  482:         uint256 freeWantInStrat;
  483:         uint256 lockedProfitBeforeLoss;
  484:     }
    //@audit Can save about 4 storage slots
```

(2) Significantly optimize gas usage by upgrading solidity version

The solidity version that is used on these contracts are significantly low: `0.6.11`. There has been many optimizations to the latest solidity version of `0.8.19` this can significantly boost up the gas saving of each contract that is used.

```solidity
ActivePool.sol:
    3: pragma solidity 0.6.11;

BorrowerOperations.sol:
    3: pragma solidity 0.6.11;

CollateralConfig.sol:
    3: pragma solidity 0.6.11;

LUSDToken.sol:
    3: pragma solidity 0.6.11;

StabilityPool.sol:
    3: pragma solidity 0.6.11;

TroveManager.sol:
    3: pragma solidity 0.6.11;

CommunityIssuance.sol:
    3: pragma solidity 0.6.11;

LQTYStaking.sol:
    3: pragma solidity 0.6.11;
```