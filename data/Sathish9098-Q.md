# LOW FINDINGS

##

### [1]  OWNER CAN RENOUNCE OWNERSHIP

Description

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The non-fungible Ownable used in this project contract implements renounceOwnership . This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

 which sets the contract owner to the zero address. Once ownership has been renounced, the contract owner will no longer be able to perform any actions that require ownership, and ownership of the contract will effectively be transferred to no one

#### onlyOwner functions : 

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

[LINK TO CODE](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)

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

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L61-L68)

      101 : function fund(uint amount) external onlyOwner {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L101)

      120 :  function updateDistributionPeriod(uint256 _newDistributionPeriod) external onlyOwner {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L120)


Recommended Mitigation Steps

We recommend either reimplementing the function to disable it or to clearly specify if it is part of the contract design.


##

### [2]  MIXING AND OUTDATED COMPILER

All contracts in a smart contract system should ideally use the same version of the Solidity programming language in order to ensure compatibility and prevent unexpected errors.

Context:
All contracts

The pragma version used are:

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

File : BorrowerOperations.sol

          3: pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L3)

File: 2023-02-ethos/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol

      3 :  pragma solidity ^0.8.0;

File : 2023-02-ethos/Ethos-Vault/contracts/ReaperVaultERC4626.sol

     3:  pragma solidity ^0.8.0;

Recommendation:

Old version of Solidity is used , newer version can be used (0.8.17)

##

### [3]  USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Context:
All contracts

Using named imports instead of plain imports can make your Solidity code more readable and reduce the risk of naming conflicts between different contracts and libraries

When you use a plain import statement in Solidity, you are importing all of the contracts and libraries defined in the imported file. This can make it difficult to determine which contracts or libraries are actually being used in your code, and can lead to naming conflicts if you import multiple files that define contracts or libraries with the same name

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better

Recommended Mitigation Steps :

    - 5:  import "./Dependencies/CheckContract.sol";
    +5:  import {CheckContract} from "./Dependencies/CheckContract.sol"; 

Implement named imports for all contracts 

##

### [5]  CONSTANT REDEFINED ELSEWHERE

Consider defining in only one contract so that values cannot become out of sync when only one location is updated.

A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L21-L25)

##

### [6] ADD A LIMIT FOR THE MAXIMUM NUMBER OF CHARACTERS PER LINE

The solidity (documentation)[https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length] recommends a maximum of 120 characters

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

### [7] CONTRACT LAYOUT AND ORDER OF FUNCTIONS

The Solidity style guide (recommends)[https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout] declaring modifiers before the functions. Now modifiers declared bottom of the contract .

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L128-L132)

Another recommendation is to declare internal functions below external functions

 If possible, consider adding internal functions below external functions for the contract layout. So please move external function on top of the internal function.

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L724-L751)

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L748-L751

##

### [8]  NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

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

##

### [9] DECIMALS() NOT PART OF ERC20 STANDARD

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

     63 :  uint256 decimals = IERC20(collateral).decimals();

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L63

##

### [10] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

Using scientific notation for large multiples of ten will improve code readability

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L53)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      uint256 public yieldSplitTreasury = 20_00; // amount of yield to direct to treasury in BPS
      uint256 public yieldSplitSP = 40_00; // amount of yield to direct to stability pool in BPS
      uint256 public yieldSplitStaking = 40_00; // amount of yield to direct to OATH Stakers in BPS

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L52-L54)

##

### [11] USE LATEST OPENZEPPELIN CONTRACTS 

Your current version of @openzeppelin/contracts is ^4.7.3 and latest version is 4.8.1

Your current version of @openzeppelin/contracts-upgradeable is ^4.7.3 and latest version is 4.8.1

##

### [12]  NatSpec comments should be increased in contracts

##

### [13] Repeated codes can be reused instead repeating same line of codes in multiple functions


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

### [14]  LACK OF CHECKS THE INTEGER RANGES

All methods lack checks on the following integer arguments. 

!=0 conditions not checked . The maximum range condition is not checked. This is not good code practice proceed arguments without checking.

So before calling external calls or doing any mathematical operations the input arguments must be checked .

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478-L507)

##

### [15]  USE PUBLIC CONSTANT CONSISTENTLY

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

### [16] INTERCHANGEABLE USAGE OF UINT AND UINT256

Consider using only one approach throughout the codebase, e.g. only uint or only uint256.

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

### [17] _approve() should be replaced with _safeApprove() 

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

##

### [18] _safeMint() should be used rather than _mint() wherever possible

safeMint() is generally considered to be a safer and more secure way of minting new tokens in an ERC20 contract, as it includes additional checks and safeguards to prevent certain types of vulnerabilities that can arise during token minting.

File: Ethos-Core/contracts/LUSDToken.sol

   188:    _mint(_account, _amount);

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L188

##

### [19]  CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

It is bad practice to use numbers directly in code without explanation

FIE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

    127 :  require(_bps <= 10_000, "Invalid BPS value");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L127)

   133 : require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L133)

    145 :  require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L145)

##

### [20]  high _bps are often associated with higher risk strategies that may involve locking up assets for a long time 

As per current protocol implementation usung setYieldingPercentage() function possible to set _bps 100% . A yield of 100% could be an annual percentage yield (APY), which means that a user's investment can double in value over a year.

 require(_bps <= 10_000, "Invalid BPS value")

It's important to note that high yield opportunities in DeFi come with their own set of risks, including smart contract vulnerabilities, liquidity risks, and impermanent loss

FIE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

      127 :  require(_bps <= 10_000, "Invalid BPS value");

Recommended mitigation :

Consider reducing _bps as per other yield farming protocols

##

### [21]  GENERATE PERFECT CODE HEADERS EVERY TIME

Description
I recommend using header for Solidity code layout and readability

(https://github.com/transmissions11/headers)

##

### [22] State variables in Solidity do not necessarily need to start with capital letters, although it is common practice to begin them with a capital letter. The Solidity style guide suggests using mixedCase for state variables, where the first word is not capitalized but subsequent words are


FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

         195:  uint public P = DECIMAL_PRECISION;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L195)

##

### [23]  FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

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

### [24] IMPORTS CAN BE GROUPED TOGETHER

Consider Dependencies first, then all interfaces

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L5-L11)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L5-L16)



##

### [25]  Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

     87 : int256 endTimestamp = block.timestamp > lastDistributionTime ? lastDistributionTime : block.timestamp;

    93:   lastIssuanceTimestamp = block.timestamp;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L87-L93)

##

### [26]  Ensure that the lengths of the amounts and assets arrays are the same


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

### [27]  The wrong emit function was used. If we want to emit the _lusdTokenAddress, we should use the LUSDTokenAddressSet event. However, in this case, the incorrect LQTYTokenAddressSet event was used.

File : 2023-02-ethos/Ethos-Core/contracts/LQTY/LQTYStaking.sol

      85 :  emit LQTYTokenAddressSet(_lusdTokenAddress);

Recommended Mitigation : 

     -  85 :  emit LQTYTokenAddressSet(_lusdTokenAddress);
    +  85 :  emit LUSDTokenAddressSet (_lusdTokenAddress);

##

## [28]  Hardcode the address causes no future updates

 In case the addresses change due to reasons such as updating their versions in the future, addresses coded as constants cannot be updated

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

    42 :  bytes32 private constant _PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;

    43:   bytes32 private constant _TYPE_HASH = 0x8b73c3c69bb8fe3d512ecc4cf759cc79239f7b179b0ffacaa9a75d522b39400f;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L42-L44)

Recommended Mitigation Steps :

    so it is recommended to add the update option with the onlyOwner modifier

## [29] Avoid "_ " from state variables 

 it is generally not necessary or recommended to prefix state variables with an underscore

State variables are typically declared at the beginning of a contract's body, outside of any functions, and are given descriptive names that reflect their purpose

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

   54 :  mapping (address => uint256) private _nonces;

   57:   mapping (address => uint256) private _balances;

   58:   mapping (address => mapping (address => uint256)) private _allowances; 

##

### [30]  USE INTERNAL CONSTANT CONSISTENTLY

Replace CONSTANT INTERNAL with INTERNAL CONSTANT

File :  2023-02-ethos/Ethos-Core/contracts/LUSDToken.sol

    string constant internal _NAME = "LUSD Stablecoin";
    string constant internal _SYMBOL = "LUSD";
    string constant internal _VERSION = "1";
    uint8 constant internal _DECIMALS = 18;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LUSDToken.sol#L32-L35)

##

### [31]  CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE 

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

Recommended Mitigation Steps

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

##

### [31]  NO SAME VALUE INPUT CONTROL

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

##

### [32] External functions pauseMinting(),unpauseMinting()  are not used anywhere in other contracts 

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

If the pauseMinting() and unpauseMinting() functions in a contract are not used anywhere else in other contracts and there are no plans to use them in the future, they can be safely removed


##

### [33]  OMISSIONS IN EVENTS

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



   





















      


    
    

   






































NC-1	Missing checks for address(0) when assigning values to address state variables	26
NC-2	require() / revert() statements should have descriptive reason strings	10
NC-3	Return values of approve() not checked	5
NC-4	TODO Left in the code	2
NC-5	Event is missing indexed fields	93
NC-6	Functions not used internally could be marked external	2
NC-7	Typos	32

L-1	abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()	1
L-2	Do not use deprecated library functions	1
L-3	Empty Function Body - Consider commenting why	2
L-4	Initializers could be front-run	8
L-5	Unsafe ERC20 operation(s)	4