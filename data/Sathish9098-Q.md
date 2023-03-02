# LOW FINDINGS

##

### [1]  OWNER CAN RENOUNCE OWNERSHIP

TYPE : LOW FINDING

Description :

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

Which sets the contract owner to the zero address. Once ownership has been renounced, the contract owner will no longer be able to perform any actions that require ownership, and ownership of the contract will effectively be transferred to no one

> onlyOwner Functions : 

File : CollateralConfig.sol

       function initialize(
        address[] calldata _collaterals,
        uint256[] calldata _MCRs,
        uint256[] calldata _CCRs
       ) external override onlyOwner {


        function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
        ) external onlyOwner checkCollateral(_collateral) {

[LINK TO CODE] (https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)

File : BorrowerOperations.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L110-L126)

FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

       function setAddresses
       (
        address _oathTokenAddress, 
        address _stabilityPoolAddress
        ) 
        external 
        onlyOwner 
        override 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L69)

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L68)

       101 : function fund(uint amount) external onlyOwner {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101)

       120 :  function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120)

Recommended Mitigation Steps :

We recommend either reimplementing the function to disable it or to clearly specify if it is part of the contract design.


##

### [2]  MIXING AND OUTDATED COMPILER

TYPE : LOW FINDING

Instances (12) :

All contracts in a smart contract system should ideally use the same version of the Solidity programming language in order to ensure compatibility and prevent unexpected errors.

Context:
All contracts

The pragma versions used are:

pragma solidity 0.6.11 and 0.8.0 

The minimum required version must be 0.8.17; otherwise, contracts will be affected by the following important bug fixes

0.8.14:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.
0.8.15

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.
0.8.16

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.
0.8.17

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.
Apart from these, there are several minor bug fixes and improvements

> IN ETHOS -CORE ALL CONTRACTS USING SOLIDITY VERSION  0.6.11. THE VERSION SHOULD BE UPDATED TO SOLIDITY 0.8.17 

INSTANCES (8) :

         3: pragma solidity 0.6.11;

> IN ETHOS -VAULT ALL CONTRACTS USING SOLIDITY VERSION  0.8.0 . THE VERSION SHOULD BE UPDATED TO SOLIDITY 0.8.17 

INSTANCES (4) :

File: 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

       3 :  pragma solidity ^0.8.0;

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

      3:  pragma solidity ^0.8.0;

Recommendation:

Old version of Solidity is used , newer version can be used (0.8.17)

##

### [3]  USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

TYPE : NON CRITICAL (NC)

> The named imports only supports only from solidity version 0.8.0. So, Its possible to update this in Ethos-Vault contracts 

Context:
ReaperVaultERC4626.sol,ReaperBaseStrategyv4.sol,ReaperStrategyGranarySupplyOnly.sol,ReaperVaultV2.sol

Using named imports instead of plain imports can make your Solidity code more readable and reduce the risk of naming conflicts between different contracts and libraries

When you use a plain import statement in Solidity, you are importing all of the contracts and libraries defined in the imported file. This can make it difficult to determine which contracts or libraries are actually being used in your code, and can lead to naming conflicts if you import multiple files that define contracts or libraries with the same name

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better

SOLUTIONS :

    -  import "./Dependencies/CheckContract.sol";
    +  import {CheckContract} from "./Dependencies/CheckContract.sol"; 

Recommended Mitigation : 

   USE NAMED IMPORTS FOR ALL POSSIBLE CONTRACT 

##

### [4]  CONSTANT REDEFINED ELSEWHERE

TYPE : NON CRITICAL (NC)

Consider defining in only one contract so that values cannot become out of sync when only one location is updated.

A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21-L25)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L53-L55)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L35)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23-L25)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L40-L41)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L32-L35)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L42-L52)


##

### [5] ADD A LIMIT FOR THE MAXIMUM NUMBER OF CHARACTERS PER LINE

TYPE : NON CRITICAL (NC)

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) recommends a maximum of 120 characters

Consider adding a limit of 120 characters or less to prevent large lines.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L637)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L675)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L697)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L538)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L528-L530)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L517)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L511)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L343)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L312)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L282)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L268-L272)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L259)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L254)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L248)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L233-L235)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L190)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L172)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L105)



##

### [6] CONTRACT LAYOUT AND ORDER OF FUNCTIONS

TYPE : NON CRITICAL (NC)

The Solidity style guide [recommends](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) declaring modifiers before the functions. Now modifiers declared bottom of the contract .

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L128-L132)

Another recommendation is to declare internal functions below external functions

 If possible, consider adding internal functions below external functions for the contract layout. So please move external function on top of the internal function.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L724-L751)

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L748-L751

##

### [7]  NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

TYPE : NON CRITICAL (NC)

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

       function _getCollChange(
        uint _collReceived,
        uint _requestedCollWithdrawal
        )
        internal
        pure
        returns(uint collChange, bool isCollIncrease)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L438-L444)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L455-L466)

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

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L203)

       function _getOffsetAndRedistributionVals
       (
        uint _debt,
        uint _coll,
        uint _LUSDInStabPool
        )
        internal
        pure
        returns (uint debtToOffset, uint collToSendToSP, uint debtToRedistribute, uint collToRedistribute)

( https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L450 )

      488 :   returns (LiquidationValues memory singleLiquidation)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L488)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1164-L1171)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L86-L89)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L114)

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

       66 :     function convertToAssets(uint256 shares) public view override returns (uint256 assets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66-L69)

        79 :     function maxDeposit(address) external view override returns (uint256 maxAssets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)

       function previewDeposit(uint256 assets) external view override returns (uint256 shares) {
        return convertToShares(assets);
        }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96-L98)

        122 :     function maxMint(address) external view override returns (uint256 maxShares) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)

        165 :     function maxWithdraw(address owner) external view override returns (uint256 maxAssets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165)

        220 :     function maxRedeem(address owner) external view override returns (uint256 maxShares) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
     
        240 :     function previewRedeem(uint256 shares) external view override returns (uint256 assets) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)

##

### [8] DECIMALS() NOT PART OF ERC20 STANDARD

TYPE : LOW FINDING

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

     63 :  uint256 decimals = IERC20(collateral).decimals();

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63

Recommended mitigations:

 Avoids using decimals() function anywhere in the contract 

##

### [9] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

TYPE : NON CRITICAL (NC)

Using scientific notation for large multiples of ten will improve code readability

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L53)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
      uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
      uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L52-L54)

##

### [10] USE LATEST OPENZEPPELIN CONTRACTS 

TYPE : LOW FINDING

Your current version of @openzeppelin/contracts is ^4.7.3 and latest version is 4.8.1

Your current version of @openzeppelin/contracts-upgradeable is ^4.7.3 and latest version is 4.8.1

Recommended Mitigation : 

 Use latest versions of openzeppelin  instead of vulnerable versions

##

### [11]  NatSpec comments should be increased in contracts

TYPE : NON CRITICAL (NC)

##

### [12] Repeated codes can be reused instead repeating same line of codes in multiple functions

TYPE : NON CRITICAL (NC)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        LocalVariables_InnerSingleLiquidateFunction memory vars;

        (singleLiquidation.entireTroveDebt,
        singleLiquidation.entireTroveColl,
        vars.pendingDebtReward,
        vars.pendingCollReward) = getEntireDebtAndColl(_borrower, _collateral);

        _movePendingTroveRewardsToActivePool(_activePool, _defaultPool, _collateral, vars.pendingDebtReward, vars.pendingCollReward);
        _removeStake(_borrower, _collateral);

        singleLiquidation.collGasCompensation = _getCollGasCompensation(singleLiquidation.entireTroveColl);
        singleLiquidation.LUSDGasCompensation = LUSD_GAS_COMPENSATION;
        uint collToLiquidate = singleLiquidation.entireTroveColl.sub(singleLiquidation.collGasCompensation);

This same codes repeated in_liquidateRecoveryMode() and _liquidateNormalMode() Functions . This line of coded can be reused instead of repeating in both functions 

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L321-L380

##

### [13]  LACK OF CHECKS THE INTEGER RANGES

TYPE : LOW FINDING

it's important to properly check integer ranges to avoid unexpected behavior and vulnerabilities in your smart contract.

INTEGER CHECKS :

    <=       ---  Upper bound check 

     >=      ---- Lower bound check

     >=  &&  <=    -- Range check

     >0     ------ Non Zero check

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478-L507)

File : File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L328-L334)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L336-L342)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L320-L326)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L311-L318)

Recommended Mitigation :

Perform all necessary input validations 

##

### [14]  USE PUBLIC CONSTANT CONSISTENTLY

TYPE : NON CRITICAL (NC)

Replace CONSTANT PUBLIC with PUBLIC CONSTANT

FILE : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

       21:  uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%

       25:   uint256 constant public MIN_ALLOWED_CCR = 1.5 ether; // 150%

   (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21-L25)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      48:  uint constant public SECONDS_IN_ONE_MINUTE = 60;
      53:  uint constant public MINUTE_DECAY_FACTOR = 999037758833783000;
      54:  uint constant public override REDEMPTION_FEE_FLOOR = DECIMAL_PRECISION / 1000 * 5; // 0.5%
      55:  uint constant public MAX_BORROWING_FEE = DECIMAL_PRECISION / 100 * 5; // 5%
      61:  uint constant public BETA = 2;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L48-L61)


FILE : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

       21:  string constant public NAME = "BorrowerOperations";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      30 :  string constant public NAME = "ActivePool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30)

FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

      150 :  string constant public NAME = "StabilityPool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150)



##

### [15] INTERCHANGEABLE USAGE OF UINT AND UINT256

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

TYPE : NON CRITICAL (NC)

Context:
ALL CONTRACTS

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L48-L63)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L67-L77)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L616)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L620)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L628)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L663-L669)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L684-L691)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L726-L732)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L130-L137)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L147-L157)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L480-L484)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L673-L679)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L870)
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1236)

##

### [16] _approve() should be replaced with _safeApprove() 

TYPE : LOW FINDING

safeApprove() is generally considered to be a safer and more secure way of approving spenders in an ERC20 contract, as it includes additional checks and safeguards to prevent certain types of vulnerabilities that can arise from a poorly implemented approve() function

The approve() function allows an address to spend a certain amount of tokens on behalf of the token owner. One vulnerability that can arise with the approve() function is known as the "double-spending" attack, where an attacker can approve a spender to transfer tokens, then transfer all their tokens to a different address, making the previously approved transfer invalid.

To prevent this type of attack, the safeApprove() function typically includes additional checks to ensure that the current allowance for a spender is zero before setting a new allowance. This prevents an attacker from double-spending an allowance that has already been approved

File: Ethos-Core/contracts/LUSDToken.sol

       231:         _approve(msg.sender, spender, amount);

       238:         _approve(sender, msg.sender, _allowances[sender][msg.sender].sub(amount, "ERC20: transfer amount exceeds allowance"));

       243:         _approve(msg.sender, spender, _allowances[msg.sender][spender].add(addedValue));

       248:         _approve(msg.sender, spender, _allowances[msg.sender][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));

       289:         _approve(owner, spender, amount);

[LUSDToken.so](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol)

Recommended Mitigation : 

Consider safeApprove() function instead of normal approve() 

##

### [17] _safeMint() should be used rather than _mint() wherever possible

TYPE : LOW FINDING

safeMint() is generally considered to be a safer and more secure way of minting new tokens in an ERC20 contract, as it includes additional checks and safeguards to prevent certain types of vulnerabilities that can arise during token minting.

File: Ethos-Core/contracts/LUSDToken.sol

      188:    _mint(_account, _amount);

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L188

Recommended Mitigation : 

Consider safeMint() function instead of normal mint() 

##

### [18]  CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

TYPE : NON CRITICAL (NC)

It is bad practice to use numbers directly in code without explanation

FIE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

       127 :  require(_bps <= 10_000, "Invalid BPS value");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L127)

       133 : require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L133)

        145 :  require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L145)

File : 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

        23:  uint256 public constant PERCENT_DIVISOR = 10_000;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L23)

##

### [19]  high _bps are often associated with higher risk strategies that may involve locking up assets for a long time 

TYPE : LOW FINDING

As per current protocol implementation using setYieldingPercentage() function possible to set _bps 100% . A yield of 100% could be an annual percentage yield (APY), which means that a user's investment can double in value over a year.

 require(_bps <= 10_000, "Invalid BPS value")

It's important to note that high yield opportunities in DeFi come with their own set of risks, including smart contract vulnerabilities, liquidity risks, and impermanent loss

FIE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

        127 :  require(_bps <= 10_000, "Invalid BPS value");

Recommended mitigation :

Consider reducing _bps as per other yield farming protocols

##

### [20]  GENERATE PERFECT CODE HEADERS EVERY TIME

TYPE : NON CRITICAL (NC)

Description
I recommend using header for Solidity code layout and readability

(https://github.com/transmissions11/headers)

##

### [21] State variables in Solidity do not necessarily need to start with capital letters, although it is common practice to begin them with a capital letter. The Solidity style guide suggests using mixedCase for state variables, where the first word is not capitalized but subsequent words are

TYPE : NON CRITICAL (NC)

FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

         195:  uint public P = DECIMAL_PRECISION;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L195)

##

### [22]  FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

TYPE : NON CRITICAL (NC)

Use camel case for all functions, parameters and variables and snake case for constants

Snake case examples
The following variable names follow the snake case naming convention:

this_is_snake_case
build_docker_image
run_javascript_function
call_python_method
ruby_loves_snake_case


Camel case examples
The following are examples of variables that use the camel case naming convention:

FileNotFoundException
toString
SpringBoot
TomcatServer
getJson

FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

        228 :  mapping (address => uint) public lastCollateralError_Offset;
        229:   uint public lastLUSDLossError_Offset;

Here state variables lastCollateralError_Offset, lastLUSDLossError_Offset using Snake case instead Camel case 

After Mitigation Camel Case :

 lastCollateralErrorOffset, lastLUSDLossErrorOffset 

File:  2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

F_Collateral_Snapshot,F_LUSD_Snapshot variables using snake case instead of camel case 

       mapping (address => uint) F_Collateral_Snapshot;
        uint F_LUSD_Snapshot;

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L35-L36

Functions are using snake case instead of camel case :

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L177)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L187)


##

### [23] IMPORTS CAN BE GROUPED TOGETHER

TYPE : NON CRITICAL (NC)

Consider Dependencies first, then all interfaces

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16)

##

### [24]  Use of Block.timestamp

TYPE : LOW FINDING

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

       87 : int256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

       93:   lastIssuanceTimestamp = block.timestamp

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87-L93)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

   540:  strategy.lastReport = block.timestamp;
   541:  lastReport = block.timestamp;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L540-L541)

##

### [25]  Ensure that the lengths of the amounts and assets arrays are the same

TYPE : LOW FINDING

It is important to check the lengths of the assets and amounts arrays before entering into the loop, as this can prevent potential vulnerabilities and errors that may occur when iterating through the arrays.

In Solidity, arrays are zero-indexed, meaning that the first element in the array is at index 0, the second element is at index 1, and so on. If the lengths of the assets and amounts arrays are not equal, it's possible that the loop may iterate over an index that is out of bounds for one of the arrays, causing an array index out of bounds error.

To prevent this issue, you can add a check at the beginning of your function to ensure that the assets and amounts arrays have the same length

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

unction _sendCollGainToUser(address[] memory assets, uint[] memory amounts) internal {
        uint numCollaterals = assets.length;
        for (uint i = 0; i < numCollaterals; i++) {
            if (amounts[i] != 0) {
                address collateral = assets[i];
                emit CollateralSent(msg.sender, collateral, amounts[i]);
                IERC20(collateral).safeTransfer(msg.sender, amounts[i]);
            }
        }
    }

Recommended Mitigation step:

require(amounts.length == assets.length, "Amounts and assets arrays must have the same length");

##

### [26]  The wrong emit function was used

TYPE : LOW FINDING

If we want to emit the _lusdTokenAddress, we should use the LUSDTokenAddressSet event. However, in this case, the incorrect LQTYTokenAddressSet event was used.

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

      94 :  emit LQTYTokenAddressSet(_lusdTokenAddress);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L94)

Recommended Mitigation : 

        -  85 :  emit LQTYTokenAddressSet(_lusdTokenAddress);
        +  85 :  emit LUSDTokenAddressSet (_lusdTokenAddress);

File: File : 2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

        function _burn(address account, uint256 amount) internal {
        assert(account != address(0));
        
        _balances[account] = _balances[account].sub(amount, "ERC20: burn amount exceeds balance");
        _totalSupply = _totalSupply.sub(amount);
          emit Transfer(account, address(0), amount);  /// @Audit emit Burn event instead of Transfer Event 
          }

Transfer event was used instead burn event 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L333)

##

## [27]  Hardcode the address causes no future updates

TYPE : LOW FINDING

 In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated.

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

       42 :  bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;

       43:   bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L42-L44)

File: 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

        24:  address public constant VELO_ROUTER = 0xa132DAB612dB5cB9fC9Ac426A0Cc215A3423F9c9;

        25:   ILendingPoolAddressesProvider public constant ADDRESSES_PROVIDER =
             ILendingPoolAddressesProvider(0xdDE5dC81e40799750B92079723Da2acAF9e1C6D6);

        27:   IAaveProtocolDataProvider public constant DATA_PROVIDER =
            IAaveProtocolDataProvider(0x9546F673eF71Ff666ae66d01Fd6E7C6Dae5a9995);

        29:   IRewardsController public constant REWARDER = IRewardsController(0x6A0406B8103Ec68EE9A713A073C7bD587c5e04aD);
  

Recommended Mitigation Steps :

    So it is recommended to add the update option with the onlyOwner modifier

##

## [28] Avoid using an underscore "_" as the first character in the name of a state variable

TYPE : NON CRITICAL (NC)

variables with underscores at the beginning of their names are treated differently from other variables in Solidity, Variables with underscores at the beginning of their names are treated as "anonymous variables" by the Solidity compiler

A common convention is to use camelCase for variable names, starting with a lowercase letter

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

   54 :  mapping (address => uint256) private _nonces;

   57:   mapping (address => uint256) private _balances;

   58:   mapping (address => mapping (address => uint256)) private _allowances; 

##

### [29]  USE INTERNAL CONSTANT CONSISTENTLY

TYPE : NON CRITICAL (NC)

Replace CONSTANT INTERNAL with INTERNAL CONSTANT

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

       string constant internal _NAME = "LUSD Stablecoin";
       string constant internal _SYMBOL = "LUSD";
       string constant internal _VERSION = "1";
       uint8 constant internal _DECIMALS = 18;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L32-L35)

##

### [30]  CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE 

TYPE : LOW FINDING

The critical procedures should be two step process

See similar findings in previous Code4rena contests for reference: (https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure)

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

         function updateGovernance(address _newGovernanceAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
        governanceAddress = _newGovernanceAddress;
        emit GovernanceAddressChanged(_newGovernanceAddress);
        }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L151)

        function updateGuardian(address _newGuardianAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)
        guardianAddress = _newGuardianAddress;
        emit GuardianAddressChanged(_newGuardianAddress);
        }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L153-L158)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

       function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address");
        treasury = newTreasury;
         }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631)


Recommended Mitigation Steps

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

##

### [31]  NO SAME VALUE INPUT CONTROL

TYPE : LOW FINDING

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      function updateGovernance(address _newGovernanceAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
        governanceAddress = _newGovernanceAddress;
        emit GovernanceAddressChanged(_newGovernanceAddress);
      }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L151)

      function updateGuardian(address _newGuardianAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)
        guardianAddress = _newGuardianAddress;
        emit GuardianAddressChanged(_newGuardianAddress);
    }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L153-L158)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

   function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address");
        treasury = newTreasury;
    }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631)

Recommended Mitigation : 

 require(owner !=  _newOwner, "The new and old owner address are same  ");

##

### [32] The external functions pauseMinting() and unpauseMinting() are not being utilized by any other contracts can be removed safely 

TYPE : LOW FINDING

Functions may be defined as external because they need to be called from other contracts, but if they are not used anywhere, it may be safe to remove them from the contract to simplify the code and reduce the attack surface.

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

      function pauseMinting() external {
        require(
            msg.sender == guardianAddress || msg.sender == governanceAddress,
            "LUSD: Caller is not guardian or governance"
        );
        mintingPaused = true;
       }

       function unpauseMinting() external {
        _requireCallerIsGovernance();
        mintingPaused = false;
        }

Recommended Mitigation Steps :

If the pauseMinting() and unpauseMinting() functions in a contract are not utilized by any other contracts, and there are no plans to use them in the future, then it is safe to remove them from the contract


##

### [33]  OMISSIONS IN EVENTS

TYPE : LOW FINDING

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible

In updateGovernance(), updateGuardian() functions only new address is emitted . No events added to emit old address 

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

         function updateGovernance(address _newGovernanceAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGovernanceAddress); // must be a smart contract (multi-sig, timelock, etc.)
        governanceAddress = _newGovernanceAddress;
        emit GovernanceAddressChanged(_newGovernanceAddress);
        }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L146-L151)

       function updateGuardian(address _newGuardianAddress) external {
        _requireCallerIsGovernance();
        checkContract(_newGuardianAddress); // must be a smart contract (multi-sig, timelock, etc.)
        guardianAddress = _newGuardianAddress;
        emit GuardianAddressChanged(_newGuardianAddress);
       }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L153-L158) 

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

      function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {
        distributionPeriod = _newDistributionPeriod;
     }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120-L122)

Recommended Mitigation Steps :

   Add events for both old and new values for every critical changes 

##

### [34]  INITIALIZE() FUNCTION CAN BE CALLED BY ANYBODY

TYPE : LOW FINDING

initialize() function can be called anybody when the contract is not initialized

File: 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

       function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
        ) public initializer {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L67)

Recommended Mitigation Steps

Add a control that makes initialize() only call the Deployer Contract;

if (msg.sender != DEPLOYER_ADDRESS) {
						revert NotDeployer();
				}
##

### [35] EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

TYPE : NON CRITICAL (NC)

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor

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

### [36] MARK VISIBILITY OF INITIALIZE() FUNCTION AS EXTERNAL

TYPE : NON CRITICAL (NC)

The initialize() function is often declared as external instead of public or internal because it is intended to be called only once, when the contract is being deployed.

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol

        function initialize(
        address _vault,
        address[] memory _strategists,
        address[] memory _multisigRoles,
        IAToken _gWant
        ) public initializer {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L62-L67)

##

### [37] LOSS OF PRECISION DUE TO ROUNDING

TYPE : LOW FINDING

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

      231:  uint256 stratMaxAllocation = (strategies[stratAddr].allocBPS * balance()) / PERCENT_DIVISOR;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L231)

      237:  uint256 vaultMaxAllocation = (totalAllocBPS * balance()) / PERCENT_DIVISOR;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L237)

      296:  return totalSupply() == 0 ? 10**decimals() : (_freeFunds() * 10**decimals()) / totalSupply();

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L296)

      334:  shares = (_amount * totalSupply()) / freeFunds; 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L334)

     365:  value = (_freeFunds() * _shares) / totalSupply();

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L365)
      
       421 : return lockedProfit - ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L421)

      440 :  uint256 bpsChange = Math.min((loss * totalAllocBPS) / totalAllocated, stratParams.allocBPS);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L440)

     463: uint256 performanceFee = (gain * strategies[strategy].feeBPS) / PERCENT_DIVISOR;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L463)

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol
 
        53 : return (assets * totalSupply()) / _freeFunds();

        68 : return (shares * _freeFunds()) / totalSupply();

##

### [38] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

TYPE : NON CRITICAL (NC)

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

roundUpDiv() internal function should be starting with underscore => _roundUpDiv()

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

     269 : function roundUpDiv(uint256 x, uint256 y) internal pure returns (uint256) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L269)

##

### [39] To add an underscore to the newTreasury function parameter

TYPE : NON CRITICAL (NC)

The parameters that are passed to the function are typically assigned to local variables within the function body. By prefixing these variables with an underscore, we can differentiate them from other variables in the contract, which can make the code more readable and easier to understand.

Solidity developers often use underscores to prefix function parameters as a way to make the code more readable and easier to understand.

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultV2.sol

       function updateTreasury(address newTreasury) external {
        _atLeastRole(DEFAULT_ADMIN_ROLE);
        require(newTreasury != address(0), "Invalid address");
        treasury = newTreasury;
       }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L627-L631)

##

### [40] The updateCollateralRatios() function contains incorrect condition checks

TYPE : LOW FINDING

According to the protocol descriptions, the updateCollateralRatios() function is designed to decrease the _MCR and _CCR values for a specific asset that has proven itself during tough times

> To prevent the same values from being set for variables, the condition check _MCR <= config.MCR and _CCR <= config.CCR should be modified to _MCR < config.MCR and _CCR < config.CCR. If the values are the same, it is unnecessary to set them again, and this process can be skipped

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

    function updateCollateralRatios(
        address _collateral,
        uint256 _MCR,
        uint256 _CCR
    ) external onlyOwner checkCollateral(_collateral) {
        Config storage config = collateralConfig[_collateral];
        require(_MCR <= config.MCR, "Can only walk down the MCR");  // @audit Wrong condition check
        require(_CCR <= config.CCR, "Can only walk down the CCR");   // @audit   Wrong condition check

        require(_MCR >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
        config.MCR = _MCR;

        require(_CCR >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
        config.CCR = _CCR;
        emit CollateralRatiosUpdated(_collateral, _MCR, _CCR);
    }

Recommended Mitigation :

Use < instead of <= when check _MCR and _CCR 

          require(_MCR < config.MCR, "Can only walk down the MCR"); 
          require(_CCR < config.CCR, "Can only walk down the CCR");  

##

### [41] Avoid shadowing the state variable names with the same contract name

TYPE : NON CRITICAL (NC)

Shadowing state variable names with the same contract name can lead to unexpected behavior and make the code harder to read and maintain.

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

    35:  mapping (address => Config) public collateralConfig;

Here the contract name is CollateralConfig and the state variable name also collateralConfig. This is not good code practice.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L35) 

##

### [42] Unwanted check of currentStake != 0

TYPE : LOW FINDING

This (currentStake != 0) already checked when calculating collGainAssets, collGainAmounts, LUSDGain  values.

currentStake varibale value is not changed anywhere inside the stake() function. 

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

            134:     if (currentStake != 0) { //@audit Unnecessary condition check
            135:     lusdToken.transfer(msg.sender, LUSDGain);
            136:    _sendCollGainToUser(collGainAssets, collGainAmounts);
            137:     }



Recommended Mitigation :

Remove Line 134,137. Unnecessary condition check

           -134:     if (currentStake != 0) {  
            135:     lusdToken.transfer(msg.sender, LUSDGain);
            136:    _sendCollGainToUser(collGainAssets, collGainAmounts);
           -137:     }

##

### [43] Events called without using emit keyword

TYPE : LOW FINDING

If you don't use the emit keyword to trigger an event in Solidity, the event will not be emitted, and no event logs will be created on the blockchain

So if you forget to emit an event in your Solidity contract, it won't have any direct impact on the contract's behavior or state. However, if you intended for the event to be triggered and relied on its emission to perform some action outside of the contract, then your code may not work as expected

File : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

        function decreaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveMorSP();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].sub(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); /// @audit ActivePoolLUSDDebtUpdated  Event 
       called without emit 
        keyword
        }

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L197-L202)

        function increaseLUSDDebt(address _collateral, uint _amount) external override {
        _requireValidCollateralAddress(_collateral);
        _requireCallerIsBOorTroveM();
        LUSDDebt[_collateral] = LUSDDebt[_collateral].add(_amount);
        ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); /// @audit ActivePoolLUSDDebtUpdated Event 
        called without emit keyword
        }
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L190-L195)

Recommended Mitigation :

Call ActivePoolLUSDDebtUpdated event using emit keyword

       emit ActivePoolLUSDDebtUpdated(_collateral, LUSDDebt[_collateral]); 
  

   





















      


    
    

   






































