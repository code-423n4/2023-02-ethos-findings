
# GAS OPTIMIZATIONS

##

### [G-1]  ++I/I++ OR --I/I-- SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} OR  UNCHECKED{--I}/UNCHECKED{I--}WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS


The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

File : CollateralConfig.sol


          56 :   for(uint256 i = 0; i < _collaterals.length; i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56)

##

## [G-2]  USE A MORE RECENT VERSION OF SOLIDITY

Using the latest version of a smart contract can help you reduce gas costs because the latest version is usually optimized for efficiency and can use fewer gas fees to execute the same function compared to an older version. In addition, the latest version may include new features that can further optimize gas usage, such as the use of newer algorithms or data structures.

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

File : CollateralConfig.sol

       3 :  pragma solidity 0.6.11;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L3)

##

### [G-3] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

In Solidity, declaring a variable inside a loop can result in higher gas costs compared to declaring it outside the loop. This is because every time the loop runs, a new instance of the variable is created, which can lead to unnecessary memory allocation and increased gas costs

On the other hand, if you declare a variable outside the loop, the variable is only initialized once, and you can reuse the same instance of the variable inside the loop. This approach can save gas and optimize the execution of your code

GAS SAMPLE TEST IN remix ide :

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
            address collateral = _collaterals[i];  //@Audit Declared inside the loop. This should be avoided . Should declare outside the loop
            checkContract(collateral);
            collaterals.push(collateral);

            Config storage config = collateralConfig[collateral];   //@Audit Declared inside the loop. This should be avoided . Should declare outside the loop
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

##

### [G-4]  USE FUNCTION INSTEAD OF MODIFIERS

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

## [G-5]  BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

bytes32 is a fixed-size data type, meaning that its size is known at compile time and cannot change during runtime. This allows Solidity to allocate the exact amount of memory needed for a bytes32 variable, which can be more efficient than allocating memory dynamically for a string variable

Because string variables are dynamically-sized and can grow or shrink during runtime, performing operations on them can be more expensive in terms of gas costs than operations on fixed-size data types like bytes32.

PROOF OF CONCEPT :

IDE USED REMIX : 

> USING STRING :

pragma solidity ^0.8.17;

contract StorageTest {

 string constant public NAME = "BorrowerOperations";

}

Remix Reports : 

gas                      :  152195 gas
transaction cost  :  132343 gas 
execution cost	   :   73523 gas 


> USING BYTES32 :

pragma solidity ^0.8.17;

contract StorageTest {

bytes32 constant public NAME = bytes32("BorrowerOperations");

}

           gas	               : 113157 gas
           transaction cost   : 98397 gas 
           execution cost	: 41893 gas

GAS SAVED :

       gas :                        39038
       transaction cost      33946
       execution cost	       31630

File :  BorrowerOperations.sol


         21 : string constant public NAME = "BorrowerOperations";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L21)

##

## [G-6]  USE REQUIRE INSTEAD OF ASSERT

When a require statement fails, any gas spent after the failing condition is refunded to the caller. This can help prevent unnecessary gas usage and make the contract more efficient. On the other hand, when an assert statement fails, all remaining gas is consumed and cannot be refunded, which can be wasteful and lead to more expensive transactions

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.


File :  BorrowerOperations.sol


   128:  assert(MIN_NET_DEBT > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L128)




       




      






























GAS-1	Use assembly to check for address(0)	8
GAS-2	Using bools for storage incurs overhead	9
GAS-3	Cache array length outside of loop	8
GAS-4	State variables should be cached in stack variables rather than re-reading them from storage	10
GAS-5	Use calldata instead of memory for function arguments that do not get mutated	1
GAS-6	Use Custom Errors	73
GAS-7	Don't initialize variables with default value	23
GAS-8	Long revert strings	35
GAS-9	Functions guaranteed to revert when called by normal users can be marked payable	7
GAS-10	++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)	17
GAS-11	Using private rather than public for constants, saves gas	30
GAS-12	Use != 0 instead of > 0 for unsigned integer comparison	27
GAS-13	internal functions not called by the contract should be removed	1