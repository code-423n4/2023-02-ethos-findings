
# GAS OPTIMIZATIONS

##

### [G-1] STATE VARIABLES ONLY SET IN THE CONSTRUCTOR SHOULD BE DECLARED IMMUTABLE

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

       /// @audit owner (constructor)

       227 : owner = msg.sender;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L227)

##

### [G-2] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations

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

##

## [G-3]  USE A MORE RECENT VERSION OF SOLIDITY

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

##

### [G-4] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

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

### [G-5]  USE FUNCTION INSTEAD OF MODIFIERS

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

## [G-6]  BYTES CONSTANTS ARE MORE EFFICIENT THAN STRING CONSTANTS

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

## [G-7]  USE REQUIRE INSTEAD OF ASSERT

When a require statement fails, any gas spent after the failing condition is refunded to the caller. This can help prevent unnecessary gas usage and make the contract more efficient. On the other hand, when an assert statement fails, all remaining gas is consumed and cannot be refunded, which can be wasteful and lead to more expensive transactions

The assert() and require() functions are a part of the error handling aspect in Solidity. Solidity makes use of state-reverting error handling exceptions. This means all changes made to the contract on that call or any sub-calls are undone if an error is thrown. It also flags an error.

They are quite similar as both check for conditions and if they are not met, would throw an error.

The big difference between the two is that the assert() function when false, uses up all the remaining gas and reverts all the changes made.

Meanwhile, a require() function when false, also reverts back all the changes made to the contract but does refund all the remaining gas fees we offered to pay. This is the most common Solidity function used by developers for debugging and error handling.


File :  BorrowerOperations.sol


   128:  assert(MIN_NET_DEBT > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L128)

   197:  assert(vars.compositeDebt > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L197)

   331:  assert(_collWithdrawal <= vars.coll); 

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L331)



##

## [G-8]  USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

 When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


File :  BorrowerOperations.sol

   175 :  ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L175)

   282 :  ContractsCache memory contractsCache = ContractsCache(troveManager, activePool, lusdToken);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L283)

   300 :  assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L301)

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

         313 :  address[] memory borrowers = new address[](1);

https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L313

##

### [G-9]  REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas

File: 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol

   52 :  require(_collaterals.length != 0, "At least one collateral required");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L52)

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

    525:  require(collateralConfig.isCollateralAllowed(_collateral), "BorrowerOps: Invalid collateral address");

    (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L525)

       529:    require(IERC20(_collateral).balanceOf(_user) >= _collAmount, "BorrowerOperations: Insufficient user collateral balance");

       530:    require(IERC20(_collateral).allowance(_user, address(this)) >= _collAmount, "BorrowerOperations: Insufficient collateral allowance");

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L529-L530)

      534:    require(_collTopUp == 0 || _collWithdrawal == 0, "BorrowerOperations: Cannot withdraw and add coll");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L534)

      538:      require(_collTopUp != 0 || _collWithdrawal != 0 || _LUSDChange != 0, "BorrowerOps: There must be either a collateral change or a debt change");

    (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L538)

     543:   require(status == 1, "BorrowerOps: Trove does not exist or is closed");

  (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L543)

      552 :  require(_LUSDChange > 0, "BorrowerOps: Debt increase requires non-zero debtChange");

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L552)

       require(
            !_checkRecoveryMode(_collateral, _price, _CCR, _collateralDecimals),
            "BorrowerOps: Operation not permitted during Recovery Mode"
        );

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L561-L564)
  
        568 :  require(_collWithdrawal == 0, "BorrowerOps: Collateral withdrawal not permitted Recovery Mode");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L568)

      617 :  require(_newICR >= _MCR, "BorrowerOps: An operation that would result in ICR < MCR is not permitted");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L617)

     621:  require(_newICR >= _CCR, "BorrowerOps: Operation must leave trove with ICR >= CCR");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L621)

     625:   require(_newICR >= _oldICR, "BorrowerOps: Cannot decrease your Trove's ICR in Recovery Mode");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L625)

     629 :  require(_newTCR >= _CCR, "BorrowerOps: An operation that would result in TCR < CCR is not permitted");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L629)

    633:  require (_netDebt >= MIN_NET_DEBT, "BorrowerOps: Trove's net debt must be greater than minimum");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L633)

   637 :   require(_debtRepayment <= _currentDebt.sub(LUSD_GAS_COMPENSATION), "BorrowerOps: Amount repaid must not be larger than the Trove's debt");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L637)
       
     641 :  require(msg.sender == stabilityPoolAddress, "BorrowerOps: Caller is not Stability Pool");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L641)

    645:   require(_lusdToken.balanceOf(_borrower) >= _debtRepayment, "BorrowerOps: Caller doesnt have enough LUSD to make repayment");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L645)

       require(_maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must less than or equal to 100%");

       require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L650-L654)

##

## [G-10]  INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

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

##

### [G-11]  Use TERNARY operator instead of IF-ELSE statements  in possible situations to save gas 

According to the Solidity documentation, the gas cost of a ternary operator is approximately 10 gas units, while the gas cost of an if-else statement is approximately 100 gas units

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol

        - 490:    if (_isDebtIncrease) {
        - 491:     _withdrawLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange, _netDebtChange);
       - 492:    } else {
        - 493:   _repayLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange);
        - 494:     }

       + _isDebtIncrease ? _withdrawLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange, _netDebtChange) :_repayLUSD(_activePool, _lusdToken, _collateral, _borrower, _LUSDChange);

##

### [G-12] SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas

File : 2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol


                require(_maxFeePercentage >= BORROWING_FEE_FLOOR && _maxFeePercentage <= DECIMAL_PRECISION,
                "Max fee percentage must be between 0.5% and 100%");

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L653-L654)

##

### [G-13]  ++I/I++ OR --I/I-- SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} OR  UNCHECKED{--I}/UNCHECKED{I--}WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS


The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

File : 2023-02-ethos/Ethos-Core/contracts/CollateralConfig.sol


          56 :   for(uint256 i = 0; i < _collaterals.length; i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56)

## [G-14]  NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

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