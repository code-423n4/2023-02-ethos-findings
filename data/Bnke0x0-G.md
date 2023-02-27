
### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::33 => address public collateralConfigAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::34 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::35 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::36 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::37 => address public defaultPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::38 => address public collSurplusPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::39 => address public treasuryAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::40 => address public lqtyStakingAddress;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::25 => ICollateralConfig public collateralConfig;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::26 => ITroveManager public troveManager;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::34 => ILQTYStaking public lqtyStaking;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::35 => address public lqtyStakingAddress;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::37 => ILUSDToken public lusdToken;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::40 => ISortedTroves public sortedTroves;
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::18 => bool public initialized = false;
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::34 => address[] public collaterals; // for returning entire list of allowed collaterals
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::37 => IERC20 public OathToken;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::39 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::41 => uint public totalOATHIssued;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::42 => uint public lastDistributionTime;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::43 => uint public distributionPeriod;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::44 => uint public rewardPerSecond;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::46 => uint public lastIssuanceTimestamp;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::26 => uint public totalLQTYStaked;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::29 => uint public F_LUSD; // Running sum of LUSD fees per-LQTY-staked
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::39 => IERC20 public lqtyToken;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::40 => ILUSDToken public lusdToken;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::41 => ICollateralConfig public collateralConfig;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::43 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::44 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::45 => address public activePoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::31 => uint256 private _totalSupply;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::67 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::68 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::69 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::70 => address public governanceAddress; // can pause/unpause minting and upgrade addresses
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::71 => address public guardianAddress; // can pause minting during emergency
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::152 => IBorrowerOperations public borrowerOperations;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::154 => ICollateralConfig public collateralConfig;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::156 => ITroveManager public troveManager;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::158 => ILUSDToken public lusdToken;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::160 => address public lqtyTokenAddress;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::163 => ISortedTroves public sortedTroves;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::165 => ICommunityIssuance public communityIssuance;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::200 => uint128 public currentScale;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::203 => uint128 public currentEpoch;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::226 => uint public lastLQTYError;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::229 => uint public lastLUSDLossError_Offset;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::21 => address public owner;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::25 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::27 => ICollateralConfig public collateralConfig;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::29 => IStabilityPool public override stabilityPool;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::35 => ILUSDToken public override lusdToken;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::37 => IERC20 public override lqtyToken;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::39 => ILQTYStaking public override lqtyStaking;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::42 => ISortedTroves public sortedTroves;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::44 => IRedemptionHelper public override redemptionHelper;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::63 => uint public baseRate;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::66 => uint public lastFeeOperationTime;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::32 => IAToken public gWant;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::35 => uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::41 => address[] public rewardClaimingTokens;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::56 => address[2][] public steps;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::42 => uint256 public tvlCap;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::44 => uint256 public totalAllocBPS; // Sum of allocBPS across all strategies (in BPS, <= 10k)
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::45 => uint256 public totalAllocated; // Amount of tokens that have been allocated to all strategies
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::46 => uint256 public lastReport; // block.timestamp of last report from any strategy
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::49 => bool public emergencyShutdown;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::56 => uint256 public lockedProfitDegradation; // rate per block of degradation. DEGRADATION_COEFFICIENT is 100% per block
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::57 => uint256 public lockedProfit; // how much profit is locked and cant be withdrawn
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::76 => bytes32 public constant ADMIN = keccak256("ADMIN");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::78 => address public treasury; // address to whom performance fee is remitted in the form of vault shares
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::28 => address public want;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::30 => bool public emergencyExit;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::31 => uint256 public lastHarvestTimestamp;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::33 => uint256 public upgradeProposalTime;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::58 => address public vault;
```



### [G02] State variables can be packed into fewer storage slots

#### Impact
If variables occupying the same slot are both written the same 
function or by the constructor avoids a separate Gsset (20000 gas). 
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::33 => address public collateralConfigAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::34 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::35 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::36 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::37 => address public defaultPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::38 => address public collSurplusPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::39 => address public treasuryAddress;
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::40 => address public lqtyStakingAddress;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::35 => address public lqtyStakingAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::39 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::43 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::44 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::45 => address public activePoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::67 => address public troveManagerAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::68 => address public stabilityPoolAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::69 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::70 => address public governanceAddress; // can pause/unpause minting and upgrade addresses
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::71 => address public guardianAddress; // can pause minting during emergency
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::160 => address public lqtyTokenAddress;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::21 => address public owner;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::25 => address public borrowerOperationsAddress;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::78 => address public treasury; // address to whom performance fee is remitted in the form of vault shares
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::28 => address public want;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::58 => address public vault;
```





### [G03] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops



#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::108 => for(uint256 i = 0; i < numCollaterals; i++) {
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::56 => for(uint256 i = 0; i < _collaterals.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::206 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::228 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::240 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::351 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::397 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::640 => for (uint i = 0; i < assets.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::810 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::831 => for (uint i = 0; i < collaterals.length; i++) {
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::859 => for (uint i = 0; i < numCollaterals; i++) {
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::608 => for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::690 => for (vars.i = 0; vars.i < _n; vars.i++) {
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::808 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::882 => for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {
```



### [G04] Splitting `require()` statements that use `&&` Cost gas

#### Impact
See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) for an example
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::653 => require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1539 => require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);
```



### [G05] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::699 => uint128 epochSnapshot = snapshots.epoch;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::700 => uint128 scaleSnapshot = snapshots.scale;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::738 => uint128 scaleSnapshot = snapshots.scale;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::739 => uint128 epochSnapshot = snapshots.epoch;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::745 => uint128 scaleDiff = currentScale.sub(scaleSnapshot);
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::817 => uint128 currentScaleCached = currentScale;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::818 => uint128 currentEpochCached = currentEpoch;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::82 => uint128 arrayIndex;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1344 => uint128 index = Troves[_borrower][_collateral].arrayIndex;
```



### [G06] Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`

#### Impact
See [this](https://github.com/ethereum/solidity/issues/9232) issue for a detailed description of the issue
#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::73 => bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::74 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::75 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::76 => bytes32 public constant ADMIN = keccak256("ADMIN");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::49 => bytes32 public constant KEEPER = keccak256("KEEPER");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::50 => bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::51 => bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::52 => bytes32 public constant ADMIN = keccak256("ADMIN");
```


### [G07] Use a more recent version of solidity

#### Impact
Use a solidity version of at least 0.8.10 to have external calls skip
 contract existence checks if the external call has a return value
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::3 => pragma solidity 0.6.11;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultERC4626.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::3 => pragma solidity ^0.8.0;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::3 => pragma solidity ^0.8.0;
```



### [G08] Amounts should be checked for 0 before calling a transfer

#### Impact
Checking non-zero transfer values can avoid an expensive external call and save gas.

While this is done in some places, it’s not consistently done in the solution.
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::103 => OathToken.transferFrom(msg.sender, address(this), amount);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::127 => OathToken.transfer(_account, _OathAmount);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::135 => lusdToken.transfer(msg.sender, LUSDGain);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::163 => lqtyToken.safeTransfer(msg.sender, LQTYToWithdraw);
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::171 => lusdToken.transfer(msg.sender, LUSDGain);
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::410 => token.safeTransfer(_receiver, value);
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultV2.sol::526 => token.safeTransfer(vars.stratAddr, vars.credit - vars.freeWantInStrat);
```



### [G09] Multiple `address` mappings can be combined into a single `mapping` of an `address` to a `struct`, where appropriate



#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::41 => mapping (address => uint256) internal collAmount;  
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::42 => mapping (address => uint256) internal LUSDDebt;  
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::44 => mapping (address => uint256) public yieldingPercentage; 
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::45 => mapping (address => uint256) public yieldingAmount; 
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::46 => mapping (address => address) public yieldGenerator; 
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::47 => mapping (address => uint256) public yieldClaimThreshold;
```

```
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::25 => mapping( address => uint) public stakes;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::28 => mapping (address => uint) public F_Collateral;  
2023-02-ethos-main/Ethos-Core/contracts/LQTY/LQTYStaking.sol::32 => mapping (address => Snapshot) public snapshots;
```


```
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::54 => mapping (address => uint256) private _nonces;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::57 => mapping (address => uint256) private _balances;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::62 => mapping (address => bool) public troveManagers;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::63 => mapping (address => bool) public stabilityPools;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::64 => mapping (address => bool) public borrowerOperations;
```

```
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::167 => mapping (address => uint256) internal collAmounts;  
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::186 => mapping (address => Deposit) public deposits;  
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::187 => mapping (address => Snapshots) public depositSnapshots; 
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::223 => mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::228 => mapping (address => uint) public lastCollateralError_Offset;
```

```
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::88 => mapping (address => uint) public totalStakes;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::91 => mapping (address => uint) public totalStakesSnapshot;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::94 => mapping (address => uint) public totalCollateralSnapshot;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::104 => mapping (address => uint) public L_Collateral;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::105 => mapping (address => uint) public L_LUSDDebt;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::119 => mapping (address => uint) public lastCollateralError_Redistribution;
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::120 => mapping (address => uint) public lastLUSDDebtError_Redistribution;
```



### [G10] Using `bools` for storage incurs overhead



#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/ActivePool.sol::32 => bool public addressesSet = false;
2023-02-ethos-main/Ethos-Core/contracts/CollateralConfig.sol::18 => bool public initialized = false;
2023-02-ethos-main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol::21 => bool public initialized = false;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::37 => bool public mintingPaused = false;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::62 => mapping (address => bool) public troveManagers;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::63 => mapping (address => bool) public stabilityPools;
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::64 => mapping (address => bool) public borrowerOperations;
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::30 => bool public emergencyExit;
```



### [G11] Usage of `assert()` instead of `require()`

#### Impact
Between solc 0.4.10 and 0.8.0, require() used REVERT (0xfd) opcode which refunded remaining gas on failure while assert() used INVALID (0xfe) opcode which consumed all the supplied gas. (see https://docs.soliditylang.org/en/v0.8.1/control-structures.html#error-handling-assert-require-revert-and-exceptions).require() should be used for checking error conditions on inputs and return values while assert() should be used for invariant checking (properly functioning code should never reach a failing assert statement, unless there is a bug in your contract you should fix).
From the current usage of assert, my guess is that they can be replaced with require, unless a Panic really is intended.
#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::128 => assert(MIN_NET_DEBT > 0);
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::197 => assert(vars.compositeDebt > 0);
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::301 => assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
2023-02-ethos-main/Ethos-Core/contracts/BorrowerOperations.sol::331 => assert(_collWithdrawal <= vars.coll);
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::312 => assert(sender != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::313 => assert(recipient != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::321 => assert(account != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::329 => assert(account != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::337 => assert(owner != address(0));
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::338 => assert(spender != address(0));
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::526 => assert(_debtToOffset <= _totalLUSDDeposits);
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::551 => assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
2023-02-ethos-main/Ethos-Core/contracts/StabilityPool.sol::591 => assert(newP > 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::417 => assert(_LUSDInStabPool != 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1224 => assert(totalStakesSnapshot[_collateral] > 0);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1279 => assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1342 => assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1348 => assert(index <= idxLast);
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1414 => assert(newBaseRate > 0); // Base rate is always non-zero after redemption
2023-02-ethos-main/Ethos-Core/contracts/TroveManager.sol::1489 => assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0
```


### [G12] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/ReaperVaultERC4626.sol::24 => ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::61 => constructor() initializer {}
```




### [G13] `abi.encode()` is less efficient than abi.encodePacked()



#### Findings:
```
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::284 => domainSeparator(), keccak256(abi.encode(
2023-02-ethos-main/Ethos-Core/contracts/LUSDToken.sol::305 => return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));
```



### [G14] Optimize names to save gas

#### Impact
public/external function names and public member variable names can be optimized to save gas. See [this
 link](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) for an example of how it works. Below are the interfaces/abstract 
contracts that can be optimized so that the most frequently-called 
functions use the least amount of gas possible during method lookup. 
Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)
#### Findings:
```
2023-02-ethos-main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol::14 => abstract contract ReaperBaseStrategyv4 is
```
