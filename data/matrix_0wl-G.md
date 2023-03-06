# Summary

## Gas Optimizations

|        | Issue                                                                                                                                               |
| ------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                                                          |
| GAS-2  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                                                   |
| GAS-3  | `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES                                               |
| GAS-4  | AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS                                                                                            |
| GAS-6  | SETTING THE CONSTRUCTOR TO PAYABLE                                                                                                                  |
| GAS-7  | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION (INSTANCES)                                                     |
| GAS-8  | DOS WITH BLOCK GAS LIMIT                                                                                                                            |
| GAS-9  | INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT |
| GAS-10 | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT                                                            |
| GAS-11 | KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE                                                                         |
| GAS-12 | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT                                                                                   |
| GAS-13 | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE                                  |
| GAS-14 | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE                                                                    |
| GAS-15 | OPTIMIZE NAMES TO SAVE GAS                                                                                                                          |
| GAS-16 | THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED                                                                                       |
| GAS-17 | USING `PRIVATE` RATHER THAN `PUBLIC` FOR CONSTANTS, SAVES GAS                                                                                       |
| GAS-18 | PROPER DATA TYPES                                                                                                                                   |
| GAS-19 | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD OR FUNCTIONS NOT USED INTERNALLY COULD BE MARKED EXTERNAL           |
| GAS-20 | USING SOLIDITY VERSION 0.8.17 WILL PROVIDE AN OVERALL GAS OPTIMIZATION                                                                              |
| GAS-21 | USE `REQUIRE` INSTEAD OF `ASSERT`                                                                                                                   |
| GAS-22 | `REQUIRE()` OR `REVERT()` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION                                                |
| GAS-23 | ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT               |
| GAS-24 | SPLITTING `REQUIRE()` STATEMENTS THAT USE && SAVES GAS                                                                                              |
| GAS-25 | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS                                                                                        |
| GAS-26 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                                                 |
| GAS-27 | USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD                                                                              |
| GAS-28 | UPGRADE TO AT LEAST 0.8.4                                                                                                                           |
| GAS-29 | USE BYTES32 INSTEAD OF STRING                                                                                                                       |
| GAS-30 | USE NAMED RETURNS WHERE APPROPRIATE                                                                                                                 |
| GAS-31 | NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING                                                                          |
| GAS-32 | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS                                                                 |
| GAS-33 | USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT                                                                     |
| GAS-34 | CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS                                                                                                  |

### [GAS-1] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

305:         return keccak256(abi.encode(typeHash, name, version, _chainID(), address(this)));

```

### [GAS-2] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

23:     uint256 public constant PERCENT_DIVISOR = 10_000;

24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF

25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-3] `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

133:             toFree += profit;

141:         roi -= int256(loss);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

168:         totalAllocBPS += _allocBPS;

194:         totalAllocBPS -= strategies[_strategy].allocBPS;

196:         totalAllocBPS += _allocBPS;

214:         totalAllocBPS -= strategies[_strategy].allocBPS;

390:                     value -= loss;

391:                     totalLoss += loss;

395:                 strategies[stratAddr].allocated -= actualWithdrawn;

396:                 totalAllocated -= actualWithdrawn;

444:                 stratParams.allocBPS -= bpsChange;

445:                 totalAllocBPS -= bpsChange;

450:         stratParams.losses += loss;

451:         stratParams.allocated -= loss;

452:         totalAllocated -= loss;

505:             strategy.gains += vars.gain;

514:                 strategy.allocated -= vars.debtPayment;

515:                 totalAllocated -= vars.debtPayment;

516:                 vars.debt -= vars.debtPayment; // tracked for return value

520:             strategy.allocated += vars.credit;

521:             totalAllocated += vars.credit;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

128:                 repayment -= uint256(-roi);

```

### [GAS-4] AVOID CONTRACT EXISTENCE CHECKS BY USING LOW LEVEL CALLS

#### Description:

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

### [GAS-5] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

127:         OathToken.transfer(_account, _OathAmount);

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

135:             lusdToken.transfer(msg.sender, LUSDGain);

171:         lusdToken.transfer(msg.sender, LUSDGain);

```

### [GAS-6] SETTING THE CONSTRUCTOR TO PAYABLE

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

57:     constructor() public {

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

84:     constructor

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

225:     constructor() public {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

16:     constructor(

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

111:     constructor(

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

61:     constructor() initializer {}

```

### [GAS-7] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION (INSTANCES)

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/TroveManager.sol

551:         require(totals.totalDebtInSequence > 0);

754:         require(totals.totalDebtInSequence > 0);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

155:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

180:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

181:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");

193:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

```

### [GAS-8] DOS WITH BLOCK GAS LIMIT

#### Description:

When smart contracts are deployed or functions inside them are called, the execution of these actions always requires a certain amount of gas, based of how much computation is needed to complete them. The Ethereum network specifies a block gas limit and the sum of all transactions included in a block can not exceed the threshold.

Programming patterns that are harmless in centralized applications can lead to Denial of Service conditions in smart contracts when the cost of executing a function exceeds the block gas limit. Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.

Reference: https://swcregistry.io/docs/SWC-128

This loops could drain all user gas and revert.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

47:         address[] calldata _collaterals,

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

332:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

792:         address[] memory _troveArray

872:         address[] memory _troveArray

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

160:     function setHarvestSteps(address[2][] calldata _newSteps) external {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

117:         address[] memory _strategists,

258:     function setWithdrawalQueue(address[] calldata _withdrawalQueue) external {

659:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

66:         address[] memory _strategists,

```

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [GAS-9] INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-10] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-11] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE

#### Description:

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-12] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

23:     uint256 public constant PERCENT_DIVISOR = 10_000;

24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF

25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-13] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

#### Description:

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

41:     mapping (address => uint256) internal collAmount;  // collateral => amount tracker

44:     mapping (address => uint256) public yieldingPercentage; // collateral => % to use for yield farming (in BPS, <= 10k)

45:     mapping (address => uint256) public yieldingAmount; // collateral => actual wei amount being used for yield farming

46:     mapping (address => address) public yieldGenerator; // collateral => corresponding ERC4626 vault

47:     mapping (address => uint256) public yieldClaimThreshold; // collateral => minimum wei amount of yield to claim and redistribute

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

25:     mapping( address => uint) public stakes;

32:     mapping (address => Snapshot) public snapshots;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

28:     mapping (address => uint) public F_Collateral;  // Running sum of collateral fees per-LQTY-staked

32:     mapping (address => Snapshot) public snapshots;

35:         mapping (address => uint) F_Collateral_Snapshot;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

62:     mapping (address => bool) public troveManagers;

63:     mapping (address => bool) public stabilityPools;

64:     mapping (address => bool) public borrowerOperations;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

186:     mapping (address => Deposit) public deposits;  // depositor address -> Deposit struct

187:     mapping (address => Snapshots) public depositSnapshots;  // depositor address -> snapshots struct

214:     mapping (uint128 => mapping(uint128 => mapping (address => uint))) public epochToScaleToSum;

223:     mapping (uint128 => mapping(uint128 => uint)) public epochToScaleToG;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

86:     mapping (address => mapping (address => Trove)) public Troves;

88:     mapping (address => uint) public totalStakes;



91:     mapping (address => uint) public totalStakesSnapshot;

94:     mapping (address => uint) public totalCollateralSnapshot;



104:     mapping (address => uint) public L_Collateral;

105:     mapping (address => uint) public L_LUSDDebt;


104:     mapping (address => uint) public L_Collateral;

105:     mapping (address => uint) public L_LUSDDebt;

109:     mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;


86:     mapping (address => mapping (address => Trove)) public Troves;

109:     mapping (address => mapping (address => RewardSnapshot)) public rewardSnapshots;

116:     mapping (address => address[]) public TroveOwners;



119:     mapping (address => uint) public lastCollateralError_Redistribution;

120:     mapping (address => uint) public lastLUSDDebtError_Redistribution;

```

#### Recommended Mitigation Steps:

Make mapping a struct instead

### [GAS-14] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

71:    function setAddresses(

125:     function setYieldingPercentage(address _collateral, uint256 _bps) external onlyOwner {

132:     function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138:     function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144:     function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214:     function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

110:   function setAddresses(

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:      function initialize(

85:      function updateCollateralRatios(

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

67:         onlyOwner

101:     function fund(uint amount) external onlyOwner {

120:     function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:       function setAddresses

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:      function setAddresses(

```

### [GAS-15] OPTIMIZE NAMES TO SAVE GAS

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

26: contract ActivePool is Ownable, CheckContract, IActivePool {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

18: contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOperations {

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

15: contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

14: contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMath {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

18: contract LQTYStaking is ILQTYStaking, Ownable, CheckContract, BaseMath {

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

28: contract LUSDToken is CheckContract, ILUSDToken {

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

146: contract StabilityPool is LiquityBase, Ownable, CheckContract, IStabilityPool {

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

18: contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

19: contract ReaperStrategyGranarySupplyOnly is ReaperBaseStrategyv4, VeloSolidMixin {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

12: contract ReaperVaultERC4626 is ReaperVaultV2, IERC4626Functions {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

21: contract ReaperVaultV2 is ReaperAccessControl, ERC20, IERC4626Events, AccessControlEnumerable, ReentrancyGuard {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

14: abstract contract ReaperBaseStrategyv4 is

```

### [GAS-16] THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. One can manually do this.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

108:         for(uint256 i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

56:         for(uint256 i = 0; i < _collaterals.length; i++) {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

206:         for (uint i = 0; i < assets.length; i++) {

228:         for (uint i = 0; i < collaterals.length; i++) {

240:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

351:         for (uint i = 0; i < numCollaterals; i++) {

397:         for (uint i = 0; i < numCollaterals; i++) {

640:         for (uint i = 0; i < assets.length; i++) {

810:             for (uint i = 0; i < collaterals.length; i++) {

831:         for (uint i = 0; i < collaterals.length; i++) {

859:         for (uint i = 0; i < numCollaterals; i++) {

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

608:         for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

690:         for (vars.i = 0; vars.i < _n; vars.i++) {

808:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

882:         for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

```

### [GAS-17] USING `PRIVATE` RATHER THAN `PUBLIC` FOR CONSTANTS, SAVES GAS

#### Description:

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

30:     string constant public NAME = "ActivePool";

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

21:     string constant public NAME = "BorrowerOperations";

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

21:     uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%

25:     uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

19:     string constant public NAME = "CommunityIssuance";

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

23:     string constant public NAME = "LQTYStaking";

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

150:     string constant public NAME = "StabilityPool";

197:     uint public constant SCALE_FACTOR = 1e9;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

48:     uint constant public SECONDS_IN_ONE_MINUTE = 60;

53:     uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

54:     uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%

55:     uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%

61:     uint constant public BETA = 2;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =

27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =

29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:     uint256 public constant DEGRADATION_COEFFICIENT = 10**18; // The unit for calculating profit degradation.

41:     uint256 public constant PERCENT_DIVISOR = 10000;

73:     bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");

74:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

75:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

76:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

23:     uint256 public constant PERCENT_DIVISOR = 10_000;

24:     uint256 public constant UPGRADE_TIMELOCK = 48 hours; // minimum 48 hours for RF

25:     uint256 public constant FUTURE_NEXT_PROPOSAL_TIME = 365 days * 100;

49:     bytes32 public constant KEEPER = keccak256("KEEPER");

50:     bytes32 public constant STRATEGIST = keccak256("STRATEGIST");

51:     bytes32 public constant GUARDIAN = keccak256("GUARDIAN");

52:     bytes32 public constant ADMIN = keccak256("ADMIN");

```

### [GAS-18] PROPER DATA TYPES

#### Description:

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type uint should be used in place of type string whenever possible.

Type uint256 takes less gas to store than uint8

Type bytes should be used over byte[]

If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.

Type bytes32 is cheaper to use than type string and bytes.

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

Fixed size variables are always cheaper than dynamic ones.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

Most of the time it will be better to use a mapping instead of an array because of its cheaper operations.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

80:         address[] calldata _erc4626vaults

105:         address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

34:     address[] public collaterals; // for returning entire list of allowed collaterals

47:         address[] calldata _collaterals,

48:         uint256[] calldata _MCRs,

49:         uint256[] calldata _CCRs

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

109:         address[] memory collGainAssets;

110:         uint[] memory collGainAmounts;

226:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

227:         uint[] memory amounts = new uint[](collaterals.length);

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

182:         uint128 scale;

183:         uint128 epoch;

200:     uint128 public currentScale;

203:     uint128 public currentEpoch;

558:         uint128 currentScaleCached = currentScale;

559:         uint128 currentEpochCached = currentEpoch;

648:         uint128 epochSnapshot;

649:         uint128 scaleSnapshot;

699:         uint128 epochSnapshot = snapshots.epoch;

700:         uint128 scaleSnapshot = snapshots.scale;

738:         uint128 scaleSnapshot = snapshots.scale;

739:         uint128 epochSnapshot = snapshots.epoch;

745:         uint128 scaleDiff = currentScale.sub(scaleSnapshot);

806:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

807:         uint[] memory amounts = new uint[](collaterals.length);

817:         uint128 currentScaleCached = currentScale;

818:         uint128 currentEpochCached = currentEpoch;

857:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

82:         uint128 arrayIndex;

313:         address[] memory borrowers = new address[](1);

1344:         uint128 index = Troves[_borrower][_collateral].arrayIndex;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

41:     address[] public rewardClaimingTokens;

64:         address[] memory _strategists,

65:         address[] memory _multisigRoles,

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

22:         address[] memory _strategists,

23:         address[] memory _multisigRoles

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

38:     address[] public withdrawalQueue;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

66:         address[] memory _strategists,

67:         address[] memory _multisigRoles

```

### [GAS-19] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD OR FUNCTIONS NOT USED INTERNALLY COULD BE MARKED EXTERNAL

#### Description:

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

295:     function getPricePerFullShare() public view returns (uint256) {

650:     function decimals() public view override returns (uint8) {

```

### [GAS-20] USING SOLIDITY VERSION 0.8.17 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

#### Description:

Using at least `0.8.10` will save gas due to skipped extcodesize check if there is a return value. Currently the contracts are compiled using version `^0.8.0` or `0.6.11`.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

3: pragma solidity ^0.8.0;

```

### [GAS-21] USE `REQUIRE` INSTEAD OF `ASSERT`

#### Description:

The `assert()` and `require()` functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the `assert()` function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a `require()` function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

128:         assert(MIN_NET_DEBT > 0);

197:         assert(vars.compositeDebt > 0);

301:         assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

331:         assert(_collWithdrawal <= vars.coll);

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

312:         assert(sender != address(0));

313:         assert(recipient != address(0));

321:         assert(account != address(0));

329:         assert(account != address(0));

337:         assert(owner != address(0));

338:         assert(spender != address(0));

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

526:         assert(_debtToOffset <= _totalLUSDDeposits);

551:         assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);

591:         assert(newP > 0);

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

417:             assert(_LUSDInStabPool != 0);

1224:             assert(totalStakesSnapshot[_collateral] > 0);

1279:         assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

1342:         assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

1348:         assert(index <= idxLast);

1414:         assert(newBaseRate > 0); // Base rate is always non-zero after redemption

1489:         assert(decayedBaseRate <= DECIMAL_PRECISION);  // The baseRate can decay to 0

```

### [GAS-22] `REQUIRE()` OR `REVERT()` STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION

#### Description:

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

107:         require(numCollaterals == _erc4626vaults.length, "Vaults array length must match number of collaterals");

```

### [GAS-23] ADD `UNCHECKED {}` FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS `REQUIRE()`, `REVERT` OR `IF` STATEMENT

#### Description:

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block [see resource](https://github.com/ethereum/solidity/issues/10695)

`require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }`

`if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }`

This will stop the check for overflow and underflow so it will save gas

#### **Proof Of Concept**

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

80:         if (wantBalance > _debt) {

92:         if (wantBal < _amountNeeded) {

99:         if (_amountNeeded > liquidatedAmount) {

131:         if (totalAssets > allocated) {

135:         } else if (totalAssets < allocated) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

234:         if (stratCurrentAllocation > stratMaxAllocation) {

374:                 if (value <= vaultBalance) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

120:             if (amountFreed < debt) {

122:             } else if (amountFreed > debt) {

```

### [GAS-24] SPLITTING `REQUIRE()` STATEMENTS THAT USE && SAVES GAS

#### Description:

Instead of using operator `&&` on a single `require`. Using a two `require` can save more gas.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

653:             require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

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

```solidity
File: Ethos-Core/contracts/TroveManager.sol

1539:         require (TroveOwnersArrayLength > 1 && sortedTroves.getSize(_collateral) > 1);

```

### [GAS-25] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

105:         address[] memory collaterals = ICollateralConfig(collateralConfigAddress).getAllowedCollaterals();

240:         LocalVariables_rebalance memory vars;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

175:         ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

176:         LocalVariables_openTrove memory vars;

283:         ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

284:         LocalVariables_adjustTrove memory vars;

577:         LocalVariables_adjustTrove memory _vars

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

109:         address[] memory collGainAssets;

110:         uint[] memory collGainAmounts;

226:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

227:         uint[] memory amounts = new uint[](collaterals.length);

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

332:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

376:         (address[] memory assets, uint[] memory amounts) = getDepositorCollateralGain(msg.sender);

663:         LocalVariables_getSingularCollateralGain memory vars;

687:         Snapshots memory snapshots = depositSnapshots[_depositor];

722:         Snapshots memory snapshots = depositSnapshots[_depositor];

806:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

807:         uint[] memory amounts = new uint[](collaterals.length);

857:         address[] memory collaterals = collateralConfig.getAllowedCollaterals();

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

313:         address[] memory borrowers = new address[](1);

331:         LocalVariables_InnerSingleLiquidateFunction memory vars;

371:         LocalVariables_InnerSingleLiquidateFunction memory vars;

518:         LocalVariables_OuterLiquidationFunction memory vars;

519:         LiquidationTotals memory totals;

595:         LocalVariables_LiquidationSequence memory vars;

599:         LiquidationValues memory singleLiquidation;

684:         LocalVariables_LiquidationSequence memory vars;

685:         LiquidationValues memory singleLiquidation;

722:         LocalVariables_OuterLiquidationFunction memory vars;

723:         LiquidationTotals memory totals;

797:         LocalVariables_LiquidationSequence memory vars;

801:         LiquidationValues memory singleLiquidation;

877:         LocalVariables_LiquidationSequence memory vars;

878:         LiquidationValues memory singleLiquidation;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

494:         LocalVariables_report memory vars;

```

### [GAS-26] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

255:        if (_chainID() == _CACHED_CHAIN_ID) {
256:            return _CACHED_DOMAIN_SEPARATOR;
257:        } else {
258:            return _buildDomainSeparator(_TYPE_HASH, _HASHED_NAME, _HASHED_VERSION);
259:        }

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

534:        if (vars.lockedProfitBeforeLoss > vars.loss) {
535:            lockedProfit = vars.lockedProfitBeforeLoss - vars.loss;
536:        } else {
537:            lockedProfit = 0;
538:        }

604:        if (_active) {
605:            _atLeastRole(GUARDIAN);
606:        } else {
607:            _atLeastRole(ADMIN);
608:        }

```

### [GAS-27] USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

35:     uint8 constant internal _DECIMALS = 18;

268:         uint8 v,

407:     function decimals() external view override returns (uint8) {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

35:     uint16 private constant LENDER_REFERRAL_CODE_NONE = 0;

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

650:     function decimals() public view override returns (uint8) {

```

### [GAS-28] UPGRADE TO AT LEAST 0.8.4

#### Description:

Use pragma solidity bigger than 0.8.0 - Using newer compiler versions and the optimizer gives gas optimizations and additional safety checks for free!

Safemath by default from 0.8.0 (can be more gas efficient than some library-based safemath.)

[Low-level inliner](https://blog.soliditylang.org/2021/03/02/saving-gas-with-simple-inliner/) from 0.8.2, leads to cheaper runtime gas. Especially relevant when the contract has small functions. For example, OpenZeppelin libraries typically have a lot of small helper functions and if they are not inlined, they cost an additional 20 to 40 gas because of 2 extra jump instructions and additional stack operations needed for function calls.

[Optimizer improvements in packed structs](https://blog.soliditylang.org/2021/03/23/solidity-0.8.3-release-announcement/#optimizer-improvements): Before 0.8.3, storing packed structs, in some cases, used additional storage read operation. After [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929), if the slot was already cold, this means unnecessary stack operations and extra deploy time costs. However, if the slot was already warm, this means an additional cost of 100 gas alongside the same unnecessary stack operations and extra deploy time costs.)

[Custom errors](https://blog.soliditylang.org/2021/04/21/custom-errors) 24 from 0.8.4, leads to cheaper deploy time cost and run time cost. Note: the run time cost is only relevant when the revert condition is met. In short, replace revert strings with custom errors.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

3: pragma solidity 0.6.11;

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

3: pragma solidity 0.6.11;

```

#### Recommended Mitigation Steps:

Upgrade to at least 0.8.4

### [GAS-29] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

30:     string constant public NAME = "ActivePool";

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

21:     string constant public NAME = "BorrowerOperations";

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

19:     string constant public NAME = "CommunityIssuance";

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

23:     string constant public NAME = "LQTYStaking";

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

32:     string constant internal _NAME = "LUSD Stablecoin";

33:     string constant internal _SYMBOL = "LUSD";

34:     string constant internal _VERSION = "1";

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

150:     string constant public NAME = "StabilityPool";

```

### [GAS-30] USE NAMED RETURNS WHERE APPROPRIATE

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/ActivePool.sol

159:     function getCollateral(address _collateral) external view override returns (uint) {

164:     function getLUSDDebt(address _collateral) external view override returns (uint) {

```

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

419:     function _triggerBorrowingFee(ITroveManager _troveManager, ILUSDToken _lusdToken, uint _LUSDAmount, uint _maxFeePercentage) internal returns (uint) {

432:     function _getUSDValue(uint _coll, uint _price, uint256 _collDecimals) internal pure returns (uint) {

466:         returns (uint, uint)

673:         returns (uint)

695:         returns (uint)

713:         returns (uint, uint)

736:         returns (uint)

748:     function getCompositeDebt(uint _debt) external pure override returns (uint) {

```

```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

102:     function getAllowedCollaterals() external override view returns (address[] memory) {

106:     function isCollateralAllowed(address _collateral) external override view returns (bool) {

112:     ) external override view checkCollateral(_collateral) returns (uint256) {

118:     ) external override view checkCollateral(_collateral) returns (uint256) {

124:     ) external override view checkCollateral(_collateral) returns (uint256) {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

199:     function getPendingCollateralGain(address _user) external view override returns (address[] memory, uint[] memory) {

213:     function getPendingLUSDGain(address _user) external view override returns (uint) {

217:     function _getPendingLUSDGain(address _user) internal view returns (uint) {

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

206:     function getDeploymentStartTime() external view override returns (uint256) {

212:     function totalSupply() external view override returns (uint256) {

216:     function balanceOf(address account) external view override returns (uint256) {

220:     function transfer(address recipient, uint256 amount) external override returns (bool) {

226:     function allowance(address owner, address spender) external view override returns (uint256) {

230:     function approve(address spender, uint256 amount) external override returns (bool) {

235:     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {

242:     function increaseAllowance(address spender, uint256 addedValue) external override returns (bool) {

247:     function decreaseAllowance(address spender, uint256 subtractedValue) external override returns (bool) {

254:     function domainSeparator() public view override returns (bytes32) {

292:     function nonces(address owner) external view override returns (uint256) { // FOR EIP 2612

304:     function _buildDomainSeparator(bytes32 typeHash, bytes32 name, bytes32 version) private view returns (bytes32) {

399:     function name() external view override returns (string memory) {

403:     function symbol() external view override returns (string memory) {

407:     function decimals() external view override returns (uint8) {

411:     function version() external view override returns (string memory) {

415:     function permitTypeHash() external view override returns (bytes32) {

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

307:     function getCollateral(address _collateral) external view override returns (uint) {

311:     function getTotalLUSDDeposits() external view override returns (uint) {

410:     function depositSnapshots_S(address _depositor, address _collateral) external override view returns (uint) {

439:     function _computeLQTYPerUnitStaked(uint _LQTYIssuance, uint _totalLUSDDeposits) internal returns (uint) {

657:     function _getSingularCollateralGain(uint _initialDeposit, address _collateral, Snapshots storage _snapshots) internal view returns (uint) {

683:     function getDepositorLQTYGain(address _depositor) public view override returns (uint) {

693:     function _getLQTYGainFromSnapshots(uint initialDeposit, Snapshots memory snapshots) internal view returns (uint) {

718:     function getCompoundedLUSDDeposit(address _depositor) public view override returns (uint) {

735:         returns (uint)

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

299:     function getTroveOwnersCount(address _collateral) external view override returns (uint) {

303:     function getTroveFromTroveOwnersArray(address _collateral, uint _index) external view override returns (address) {

1045:     function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

1058:     ) public view override returns (uint) {

1066:     function _getCurrentTroveAmounts(address _borrower, address _collateral) internal view returns (uint, uint) {

1123:     function getPendingCollateralReward(address _borrower, address _collateral) public view override returns (uint) {

1138:     function getPendingLUSDDebtReward(address _borrower, address _collateral) public view override returns (uint) {

1152:     function hasPendingRewards(address _borrower, address _collateral) public view override returns (bool) {

1195:     function updateStakeAndTotalStakes(address _borrower, address _collateral) external override returns (uint) {

1201:     function _updateStakeAndTotalStakes(address _borrower, address _collateral) internal returns (uint) {

1213:     function _computeNewStake(address _collateral, uint _coll) internal view returns (uint) {

1361:     function getTCR(address _collateral, uint _price) external view override returns (uint) {

1366:     function checkRecoveryMode(address _collateral, uint _price) external view override returns (bool) {

1382:     returns (bool)

1402:     ) external override returns (uint) {

1425:     function getRedemptionRate() public view override returns (uint) {

1429:     function getRedemptionRateWithDecay() public view override returns (uint) {

1433:     function _calcRedemptionRate(uint _baseRate) internal pure returns (uint) {

1440:     function getRedemptionFee(uint _collateralDrawn) public view override returns (uint) {

1444:     function getRedemptionFeeWithDecay(uint _collateralDrawn) external view override returns (uint) {

1448:     function _calcRedemptionFee(uint _redemptionRate, uint _collateralDrawn) internal pure returns (uint) {

1456:     function getBorrowingRate() public view override returns (uint) {

1460:     function getBorrowingRateWithDecay() public view override returns (uint) {

1464:     function _calcBorrowingRate(uint _baseRate) internal pure returns (uint) {

1471:     function getBorrowingFee(uint _LUSDDebt) external view override returns (uint) {

1475:     function getBorrowingFeeWithDecay(uint _LUSDDebt) external view override returns (uint) {

1479:     function _calcBorrowingFee(uint _borrowingRate, uint _LUSDDebt) internal pure returns (uint) {

1509:     function _calcDecayedBaseRate() internal view returns (uint) {

1516:     function _minutesPassedSinceLastFeeOp() internal view returns (uint) {

1544:     function getTroveStatus(address _borrower, address _collateral) external view override returns (uint) {

1548:     function getTroveStake(address _borrower, address _collateral) external view override returns (uint) {

1552:     function getTroveDebt(address _borrower, address _collateral) external view override returns (uint) {

1556:     function getTroveColl(address _borrower, address _collateral) external view override returns (uint) {

1567:     function increaseTroveColl(address _borrower, address _collateral, uint _collIncrease) external override returns (uint) {

1574:     function decreaseTroveColl(address _borrower, address _collateral, uint _collDecrease) external override returns (uint) {

1581:     function increaseTroveDebt(address _borrower, address _collateral, uint _debtIncrease) external override returns (uint) {

1588:     function decreaseTroveDebt(address _borrower, address _collateral, uint _debtDecrease) external override returns (uint) {

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

222:     function balanceOf() public view override returns (uint256) {

226:     function balanceOfWant() public view returns (uint256) {

230:     function balanceOfPool() public view returns (uint256) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

225:     function availableCapital() public view returns (int256) {

278:     function balance() public view returns (uint256) {

287:     function _freeFunds() internal view returns (uint256) {

295:     function getPricePerFullShare() public view returns (uint256) {

418:     function _calculateLockedProfit() internal view returns (uint256) {

462:     function _chargeFees(address strategy, uint256 gain) internal returns (uint256) {

493:     function report(int256 _roi, uint256 _repayment) external returns (uint256) {

650:     function decimals() public view override returns (uint8) {

659:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

673:     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

144:     function balanceOf() public view virtual override returns (uint256);

203:     function _cascadingAccessRoles() internal view override returns (bytes32[] memory) {

217:     function _hasRole(bytes32 _role, address _account) internal view override returns (bool) {

```

### [GAS-31] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

444:         returns(uint collChange, bool isCollIncrease)

```

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

84:     function issueOath() external override returns (uint issuance) {

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

203:     function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {

```

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

298:     function _chainID() private pure returns (uint256 chainID) {

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

511:         returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)

626:     function getDepositorCollateralGain(address _depositor) public view override returns (address[] memory assets, uint[] memory amounts) {

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

329:         returns (LiquidationValues memory singleLiquidation)

369:         returns (LiquidationValues memory singleLiquidation)

450:         returns (uint debtToOffset, uint collToSendToSP, uint debtToRedistribute, uint collToRedistribute)

488:         returns (LiquidationValues memory singleLiquidation)

593:         returns(LiquidationTotals memory totals)

682:         returns(LiquidationTotals memory totals)

795:         returns(LiquidationTotals memory totals)

875:         returns(LiquidationTotals memory totals)

899:     internal pure returns(LiquidationTotals memory newTotals) {

1171:         returns (uint debt, uint coll, uint pendingLUSDDebtReward, uint pendingCollateralReward)

```

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

89:         returns (uint256 liquidatedAmount, uint256 loss)

104:     function _liquidateAllPositions() internal override returns (uint256 amountFreed) {

114:     function _harvestCore(uint256 _debt) internal override returns (int256 roi, uint256 repayment) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

29:     function asset() external view override returns (address assetTokenAddress) {

37:     function totalAssets() external view override returns (uint256 totalManagedAssets) {

51:     function convertToShares(uint256 assets) public view override returns (uint256 shares) {

66:     function convertToAssets(uint256 shares) public view override returns (uint256 assets) {

79:     function maxDeposit(address) external view override returns (uint256 maxAssets) {

96:     function previewDeposit(uint256 assets) external view override returns (uint256 shares) {

110:     function deposit(uint256 assets, address receiver) external override returns (uint256 shares) {

122:     function maxMint(address) external view override returns (uint256 maxShares) {

139:     function previewMint(uint256 shares) public view override returns (uint256 assets) {

154:     function mint(uint256 shares, address receiver) external override returns (uint256 assets) {

165:     function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {

183:     function previewWithdraw(uint256 assets) public view override returns (uint256 shares) {

206:     ) external override returns (uint256 shares) {

220:     function maxRedeem(address owner) external view override returns (uint256 maxShares) {

240:     function previewRedeem(uint256 shares) external view override returns (uint256 assets) {

262:     ) external override returns (uint256 assets) {

269:     function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {

```

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

319:     function _deposit(uint256 _amount, address _receiver) internal nonReentrant returns (uint256 shares) {

```

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol

95:     function withdraw(uint256 _amount) external override returns (uint256 loss) {

109:     function harvest() external override returns (int256 roi) {

243:         returns (uint256 liquidatedAmount, uint256 loss);

250:     function _liquidateAllPositions() internal virtual returns (uint256 amountFreed);

276:     function _harvestCore(uint256 _debt) internal virtual returns (int256 roi, uint256 repayment);

```

### [GAS-32] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

Do not use return at the end of the function

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

203:     function _getPendingCollateralGain(address _user) internal view returns (address[] memory assets, uint[] memory amounts) {

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

511:         returns (uint collGainPerUnitStaked, uint LUSDLossPerUnitStaked)

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

329:         returns (LiquidationValues memory singleLiquidation)

369:         returns (LiquidationValues memory singleLiquidation)

899:     internal pure returns(LiquidationTotals memory newTotals) {

```

### [GAS-33] USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Description:

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

552:         require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

```

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

266:         require(currentStake > 0, 'LQTYStaking: User must have a non-zero stake');

270:         require(_amount > 0, 'LQTYStaking: Amount must be non-zero');

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

870:         require(_initialDeposit > 0, 'StabilityPool: User must have a non-zero deposit');

874:         require(_amount > 0, 'StabilityPool: Amount must be non-zero');

```

```solidity
File: Ethos-Core/contracts/TroveManager.sol

551:         require(totals.totalDebtInSequence > 0);

754:         require(totals.totalDebtInSequence > 0);

```

### [GAS-34] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

#### Description:

Make it outside and only use it inside.

https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop

#### **Proof Of Concept**

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

207:            address collateral = assets[i];
208:            uint F_Collateral_Snapshot = snapshots[_user].F_Collateral_Snapshot[collateral];

229:            address collateral = collaterals[i];

```

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

832:            address collateral = collaterals[i];
833:            uint currentS = epochToScaleToSum[currentEpochCached][currentScaleCached][collateral];

```
