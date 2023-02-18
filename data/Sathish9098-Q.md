# LOW FINDINGS

##

### [L-1]  OWNER CAN RENOUNCE OWNERSHIP

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

Recommended Mitigation Steps

We recommend either reimplementing the function to disable it or to clearly specify if it is part of the contract design.


##

### [L-2]  OUTDATED COMPILER

### ALL CONTRACTS USING THE OUTDATED COMPILER VERSIONS  solidity 0.6.11

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

##

### [L-3]  USE NAMED IMPORTS INSTEAD OF PLAIN `IMPORT ‘FILE.SOL’

Using named imports instead of plain imports can make your Solidity code more readable and reduce the risk of naming conflicts between different contracts and libraries

When you use a plain import statement in Solidity, you are importing all of the contracts and libraries defined in the imported file. This can make it difficult to determine which contracts or libraries are actually being used in your code, and can lead to naming conflicts if you import multiple files that define contracts or libraries with the same name


Recommended Mitigation Steps :

    - 5:  import "./Dependencies/CheckContract.sol";
    +5:  import {CheckContract} from "./Dependencies/CheckContract.sol"; 

### Need to modify all import statements like above mitigation way

File : CollateralConfig.sol

     6:  import "./Dependencies/Ownable.sol";
     7:  import "./Dependencies/SafeERC20.sol";
     8:  import "./Interfaces/ICollateralConfig.sol";

[LINK TO CODE](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol)




##

# NON CRITICAL FINDINGS

##

### [N-1]  USE A MORE RECENT VERSION OF SOLIDITY

Use a solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>)

File : CollateralConfig.sol


     3 :  pragma solidity 0.6.11;
























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