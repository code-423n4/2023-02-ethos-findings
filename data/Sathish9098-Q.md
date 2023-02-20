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

Recommended Mitigation Steps

We recommend either reimplementing the function to disable it or to clearly specify if it is part of the contract design.


##

### [2]  OUTDATED COMPILER

### ALL CONTRACTS USING THE OUTDATED COMPILER VERSIONS  solidity 0.6.11

Context:
All contracts

The pragma version used are:

pragma solidity 0.6.11;

#### Here are some additional reasons why it's generally a good idea to use the latest version of Solidity instead of older versions

Bug fixes: Each new version of Solidity includes bug fixes that address issues discovered in previous versions. Using the latest version of Solidity can help you avoid these issues and write more reliable smart contracts.

New features: Each new version of Solidity often includes new features that can make it easier to write smart contracts. 

Easier development: Using the latest version of Solidity can make it easier to develop and test your smart contracts. For example, newer versions of Solidity often include improvements to the language's syntax and error handling, which can help you write better code and catch errors more easily.

Community support: The Solidity community is active and growing, and using the latest version of the language can help you take advantage of new resources, such as forums, documentation, and libraries. Additionally, using the latest version of Solidity can make it easier to collaborate with other developers who are also using the latest tools and technologies.

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

Recommendation:

Old version of Solidity is used , newer version can be used (0.8.17)



##

### [3]  USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Context:
All contracts

Using named imports instead of plain imports can make your Solidity code more readable and reduce the risk of naming conflicts between different contracts and libraries

When you use a plain import statement in Solidity, you are importing all of the contracts and libraries defined in the imported file. This can make it difficult to determine which contracts or libraries are actually being used in your code, and can lead to naming conflicts if you import multiple files that define contracts or libraries with the same name


Recommended Mitigation Steps :

    - 5:  import "./Dependencies/CheckContract.sol";
    +5:  import {CheckContract} from "./Dependencies/CheckContract.sol"; 

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better

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