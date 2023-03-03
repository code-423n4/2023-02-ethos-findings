# Report


## Low Risk and Non-Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [LOW-1] | OWNER CAN RENOUNCE OWNERSHIP | 12 |
| [LOW-2] | LOSS OF PRECISION DUE TO ROUNDING | 3 |
| [LOW-3] | INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY | 1 |
| [LOW-4] | VARIABLE NAME IS CONFUSING | 1 |
| [LOW-5] | HARDCODE THE ADDRESS CAUSES NO FUTURE UPDATES | 6 |
| [LOW-6] | REQUIRE()/REVERT() STATEMENTS SHOULD HAVE DESCRIPTIVE REASON STRINGS | 11 |
| [LOW-7] | RECIPIENT MAY BE ADDRESS(0) | 3 |
| [LOW-8] | MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE | 6 |
| [N-1] | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT | 8 |
| [N-2] | FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS | 1 |
| [N-3] | LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTION | 1 |
| [N-4] | UNUSED CONSTRUCTOR | 1 |
| [N-5] | EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING | 2 |
| [N-6] | FLOATING PRAGMA | 4 |
| [N-7] | NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING | 1 |
| [N-8] | USE REQUIRE INSTEAD OF ASSERT | 20 |
| [N-9] | OPEN TODOS | 2 |
| [N-10] | DUPLICATE IMPORT STATEMENTS | 2 |
| [N-11] | NUMERIC VALUES HAVING TO DO WITH TIME SHOULD USE TIME UNITS FOR READABILITY | 2 |
| [N-12] | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION | 1 |
| [N-13] | USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18) | 1 |
| [N-14] | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION | 6 |
| [N-15] | COMMENT DOUBLE SPACING | 1 |
| [N-16] | INCONSISTENT NATSPEC FORMAT | 1 |

### [LOW-1] OWNER CAN RENOUNCE OWNERSHIP
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

*Instances (12)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/CollateralConfig.sol

46:    function initialize(
47:        address[] calldata _collaterals,
48:        uint256[] calldata _MCRs,
49:        uint256[] calldata _CCRs
50:    ) external override onlyOwner 

85:    function updateCollateralRatios(
86:            address _collateral,
87:            uint256 _MCR,
88:            uint256 _CCR
89:    ) external onlyOwner checkCollateral(_collateral) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L50
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L85-L89

```solidity
File: Ethos-Core/contracts/ActivePool.sol

71:  function setAddresses(
72:        address _collateralConfigAddress,
73:        address _borrowerOperationsAddress,
74:        address _troveManagerAddress,
75:        address _stabilityPoolAddress,
76:        address _defaultPoolAddress,
77:        address _collSurplusPoolAddress,
78:        address _treasuryAddress,
79:        address _lqtyStakingAddress,
80:        address[] calldata _erc4626vaults
81:    )
82:        external
83:        onlyOwner

132: function setYieldingPercentageDrift(uint256 _driftBps) external onlyOwner {

138: function setYieldClaimThreshold(address _collateral, uint256 _threshold) external onlyOwner {

144: function setYieldDistributionParams(uint256 _treasurySplit, uint256 _SPSplit, uint256 _stakingSplit) external onlyOwner {

214: function manualRebalance(address _collateral, uint256 _simulatedAmountLeavingPool) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L71-L83
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L132
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L138
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L144
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L214

```solidity
File: Ethos-Core/contracts/StabilityPool.sol

261:    function setAddresses(
262:        address _borrowerOperationsAddress,
263:        address _collateralConfigAddress,
264:           address _troveManagerAddress,
265:            address _activePoolAddress,
266:            address _lusdTokenAddress,
267:            address _sortedTrovesAddress,
268:            address _priceFeedAddress,
269:            address _communityIssuanceAddress
270:        )
271:        external
272:        override
273:        onlyOwner  
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L261-L273

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

61:        function setAddresses
62:    (
63:        address _oathTokenAddress, 
64:        address _stabilityPoolAddress
65:    ) 
66:        external 
67:        onlyOwner 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L67

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

101:    function fund(uint amount) external onlyOwner {

120:    function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120

```solidity
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol

66:    function setAddresses
67:    (
68:        address _lqtyTokenAddress,
69:        address _lusdTokenAddress,
70:        address _troveManagerAddress, 
71:        address _borrowerOperationsAddress,
72:        address _activePoolAddress,
73:        address _collateralConfigAddress
74:    ) 
75:        external 
76:        onlyOwner 
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L66-L76


### [LOW-2] LOSS OF PRECISION DUE TO ROUNDING

*Instances (3)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

231:         uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;
237:         uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;
365:         value = (_freeFunds() * _shares) / totalSupply();

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L231
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L237
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L365


### [LOW-3] INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY
initialize() function can be called anybody when the contract is not initialized.

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:    function initialize(
63:        address _vault,
64:        address[] memory _strategists,
65:        address[] memory _multisigRoles,
66:        IAToken _gWant
67:    ) public initializer {
68:        gWant = _gWant;
69:        want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:    }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72

#### Recommended Mitigation Steps
Add a control that makes initialize() only call the Deployer Contract;

```solidity
if (msg.sender != DEPLOYER_ADDRESS) {
	revert NotDeployer();
}
```

### [LOW-4] VARIABLE NAME IS CONFUSING
Use length as a variable name is confusing, since solidity has its only length method such as array.length. Recommend to use TroveOwnersArrayLength directly.

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

1345:  uint length = TroveOwnersArrayLength;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1345


### [LOW-5] HARDCODE THE ADDRESS CAUSES NO FUTURE UPDATES
In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated, so it is recommended to add the update option with the onlyOwner modifier.

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/LUSDToken.sol

42:     bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
44:     bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L42
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L44

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

24:     address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;
25:     ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER = 
26:     ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);
27:     IAaveProtocolDataProvider public constant DATA_PROVIDER =
28:     IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);
29:     IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29


### [LOW-6] REQUIRE()/REVERT() STATEMENTS SHOULD HAVE DESCRIPTIVE REASON STRINGS
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

*Instances (11)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

250:         require(msg.sender == owner);
551:         require(totals.totalDebtInSequence > 0);
716:         require(_troveArray.length != 0);
754:         require(totals.totalDebtInSequence > 0);
1450:        require(redemptionFee < _collateralDrawn);
1523:        require(msg.sender == borrowerOperationsAddress);
1527:        require(msg.sender == address(redemptionHelper));
1531:        require(msg.sender == borrowerOperationsAddress || msg.sender == address(redemptionHelper));}
1535:        require(Troves[_borrower][_collateral].status == Status.active);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L250
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L716
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1450
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1523
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1527
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1531
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1535

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

167:         require(step[0] != address(0));
168:         require(step[1] != address(0));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L167-L168


### [LOW-7] RECIPIENT MAY BE ADDRESS(0)
The recipient of a fee may be address(0), leading to a lose.

*Instances (3)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/ActivePool.sol

186:        IERC20(_collateral).safeTransfer(_account, _amount);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L186

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

127:        OathToken.transfer(_account, _OathAmount);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L127

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

410:        token.safeTransfer(_receiver, value);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L410


### [LOW-8] MISSING EVENT FOR CRITICAL PARAMETERS INIT AND CHANGE

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

225:        constructor() public {
226:            // makeshift ownable implementation to circumvent contract size limit
227:            owner = msg.sender;
228:        }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L225-L228

```solidity
File: Ethos-Core/contracts/LQTY/CommunityIssuance.sol

57:         constructor() public {
58:             distributionPeriod = 14 days;
59:         }

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L57-L59

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

84:     constructor
85:       ( 
86:        address _troveManagerAddress,
87:        address _stabilityPoolAddress,
88:        address _borrowerOperationsAddress,
89:        address _governanceAddress,
90:        address _guardianAddress
91:       ) 
92:     public 

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L84-L92

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

111:    constructor(
112:        address _token,
113:        string memory _name,
114:        string memory _symbol,
115:        uint256 _tvlCap,
116:        address _treasury,
117:        address[] memory _strategists,
118:        address[] memory _multisigRoles
119:    ) ERC20(string(_name), string(_symbol)) {

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L119

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

16:    constructor(
17:        address _token,
18:        string memory _name,
19:        string memory _symbol,
20:        uint256 _tvlCap,
21:        address _treasury,
22:        address[] memory _strategists,
23:        address[] memory _multisigRoles
24:    ) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L16-L24

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

61:     constructor() initializer {}

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61


### [N-1] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USED TO IMMUTABLE RATHER THAN CONSTANT

*Instances (8)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

73:         bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
74:         bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
75:         bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
76:         bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

49:         bytes32 public constant KEEPER = keccak256("KEEPER");
50:         bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
51:         bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
52:         bytes32 public constant ADMIN = keccak256("ADMIN");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52


### [N-2] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS
Internal and private functions: the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

269:		function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L269


### [N-3] LACK OF EVENT EMISSION AFTER CRITICAL INITIALIZE() FUNCTION
To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the initialize() function

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

62:	    function initialize(
63:        address _vault,
64:        address[] memory _strategists,
65:        address[] memory _multisigRoles,
66:        IAToken _gWant
67:    ) public initializer {
68:        gWant = _gWant;
69:        want = _gWant.UNDERLYING_ASSET_ADDRESS();
70:        __ReaperBaseStrategy_init(_vault, want, _strategists, _multisigRoles);
71:        rewardClaimingTokens = [address(_gWant)];
72:    }
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L72


### [N-4] UNUSED CONSTRUCTOR

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

61:		constructor() initializer {}
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61


### [N-5] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

61:		constructor() initializer {}
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L61

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

24:		) ReaperVaultV2(_token, _name, _symbol, _tvlCap, _treasury, _strategists, _multisigRoles) {}
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L24


### [N-6] FLOATING PRAGMA
Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103

Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

*Instances (4)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

3:		pragma solidity ^0.8.0;
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperVaultERC4626.sol

3:		pragma solidity ^0.8.0;
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3

```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

3:		pragma solidity ^0.8.0;
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3

```solidity
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

3:		pragma solidity ^0.8.0;
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3


### [N-7] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

240:    function _liquidatePosition(uint256 _amountNeeded)
241:        internal
242:        virtual
243:        returns (uint256 liquidatedAmount, uint256 loss);
	
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L240-L243

### [N-8] USE REQUIRE INSTEAD OF ASSERT
Assert should not be used except for tests, require should be used

Prior to Solidity 0.8.0, pressing a confirm consumes the remainder of the process’s available gas instead of returning it, as request()/revert() did.

*Instances (20)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/BorrowerOperations.sol

128:	assert(MIN_NET_DEBT > 0);
197: 	assert(vars.compositeDebt > 0);
301:	assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
331:	assert(_collWithdrawal <= vars.coll); 

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331

```solidity
File: Ethos-Core/contracts/TroveManager.sol

417:	 assert(_LUSDInStabPool != 0);
1224:	 assert(totalStakesSnapshot[_collateral] > 0);
1279:	 assert(closedStatus != Status.nonExistent && closedStatus != Status.active);
1342: 	 assert(troveStatus != Status.nonExistent && troveStatus != Status.active);
1348:	 assert(index <= idxLast);
1414:	 assert(newBaseRate > 0); 
1489:	 assert(decayedBaseRate <= DECIMAL_PRECISION);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1348
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489

```solidity
File: Ethos-Core/contracts/StabilityPool.soll

526:	assert(_debtToOffset <= _totalLUSDDeposits);
551:	assert(_LUSDLossPerUnitStaked <= DECIMAL_PRECISION);
591:	assert(newP > 0);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591

```solidity
File: Ethos-Core/contracts/LUSDToken.sol

312:    assert(sender != address(0));
313:    assert(recipient != address(0));
321: 	assert(account != address(0));
329:	assert(account != address(0));
337:    assert(owner != address(0));
338:    assert(spender != address(0));

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337-L338


### [N-9] OPEN TODOS
Use temporary TODOs as you work on a feature, but make sure to treat them before merging. Either add a link to a proper issue in your TODO, or remove it from the code.

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/StabilityPool.sol

335:	/* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.
380:	/* TODO tess3rac7 unused var, but previously included in ETHGainWithdrawn event log.

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L335
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L380


### [N-10] DUPLICATE IMPORT STATEMENTS

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/StabilityPool.sol

5:		import './Interfaces/IBorrowerOperations.sol';
8:		import './Interfaces/IBorrowerOperations.sol';

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L5
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L8


### [N-11] NUMERIC VALUES HAVING TO DO WITH TIME SHOULD USE TIME UNITS FOR READABILITY
There are https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units for seconds, minutes, hours, days, and weeks, and since they’re defined, they should be used.

*Instances (2)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

48:		uint constant public SECONDS_IN_ONE_MINUTE = 60;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48

```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

125:	lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125


### [N-12] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION
Using scientific notation for large multiples of ten will improve code readability.

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

53:		uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53


### [N-13] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)
While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

40:		uint256 public constant DEGRADATION_COEFFICIENT = 10**18;

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40


### [N-14] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

*Instances (6)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

551:         require(totals.totalDebtInSequence > 0);
754:         require(totals.totalDebtInSequence > 0);

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L754


```solidity
File: Ethos-Vault/contracts/ReaperVaultV2.sol

180:         require(strategies[_strategy].activation != 0, "Invalid strategy address");
193:         require(strategies[_strategy].activation != 0, "Invalid strategy address");

155:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
181:         require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L193
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L155
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L181


### [N-15] COMMENT DOUBLE SPACING
Double spacing between  // and if

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Core/contracts/TroveManager.sol

742:       } else {  //  if !vars.recoveryModeAtStart

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L742


### [N-16] INCONSISTENT NATSPEC FORMAT
For Solidity you may choose /// for single or multi-line comments, or /** and ending with */. /** */ is used in all of the contracts, except below one: 

*Instances (1)*:
#### Proof of Concept
```solidity
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

60:       /// @custom:oz-upgrades-unsafe-allow constructor

```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L60

### Tools
Manual Audit