
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

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {
            // we need to cache it, because current user is likely going to be deleted
            address nextUser = _sortedTroves.getPrev(_collateral, vars.user);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L608-L612)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L819)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L108-L113)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351-L357)

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

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

   30 :  string constant public NAME = "ActivePool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L30)

FILE :  2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

   150 :  string constant public NAME = "StabilityPool";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L150)

FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

    19:    string constant public NAME = "CommunityIssuance";

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L19)

   

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


FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

    1281 :  assert(closedStatus != Status.nonExistent && closedStatus != Status.active);

 (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1279)

    1342 :  assert(troveStatus != Status.nonExistent && troveStatus != Status.active);

    1348 :  assert(index <= idxLast);

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

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L313)

##

## [G-9] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      1045 :  function getNominalICR(address _borrower, address _collateral) public view override returns (uint) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1045)

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

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L478-L488)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L582-L594)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L671-L682)

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L898-L913)


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

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      608 :   for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L608)

     690 :  for (vars.i = 0; vars.i < _n; vars.i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L690)

    808 :  for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L808)

   882 : for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L882)

FIE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

    108 :  for(uint256 i = 0; i < numCollaterals; i++) {

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L108)

FILE : 2023-02-ethos/Ethos-Core/contracts/StabilityPool.sol

         351:  for (uint i = 0; i < numCollaterals; i++) {
    
(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351)
     
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

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1321-L1333)

     

##

### [G-15] ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

SOLUTION:

 require(a < b); x = b - a => require(a <b); unchecked { x = b - a } 

require(a > b); x = a - b => require(a > b); unchecked { x = a - b } 

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol


            493:   if (_collDecimals < LiquityMath.CR_CALCULATION_DECIMALS) {
            494:   cappedCollPortion = cappedCollPortion.div(10 ** (LiquityMath.CR_CALCULATION_DECIMALS - _collDecimals));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L493-L494)

           495 :  else if (_collDecimals > LiquityMath.CR_CALCULATION_DECIMALS) {
            496:  cappedCollPortion = cappedCollPortion.mul(10 ** (_collDecimals - LiquityMath.CR_CALCULATION_DECIMALS));

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L495-L496)

## [G-16]  USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size

(https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Each operation involving a uint128 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint128, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

    1321 :  function _addTroveOwnerToArray(address _borrower, address _collateral) internal returns (uint128 index) {

    1329 :  index = uint128(TroveOwners[_collateral].length.sub(1));

    1344 :   uint128 index = Troves[_borrower][_collateral].arrayIndex;

    1414 :   assert(newBaseRate > 0);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1344)

##

### [17] SUPERFLUOUS EVENT FIELDS

block.timestamp are added to event information by default so adding them manually wastes gas

FILE : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

      1505 :  emit LastFeeOpTimeUpdated(block.timestamp);

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1505)

##

### [G-18] State variables that are only set within the setAddresses() function should be declared as immutable. According to protocol, the setAddresses() function behaves like a constructor function. Once the setAddresses() function is called, it is not possible to recall it from any other location within the contract. 

Avoids a Gsset (20000 gas) in the setAddresses(), and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas)

 File : 2023-02-ethos/Ethos-Core/contracts/TroveManager.sol

        borrowerOperationsAddress = _borrowerOperationsAddress;
        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPool = IStabilityPool(_stabilityPoolAddress);
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lqtyToken = IERC20(_lqtyTokenAddress);
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);
        redemptionHelper = IRedemptionHelper(_redemptionHelperAddress);   

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L266-L278)

FILE : 2023-02-ethos/Ethos-Core/contracts/ActivePool.sol

       collateralConfigAddress = _collateralConfigAddress;
        borrowerOperationsAddress = _borrowerOperationsAddress;
        troveManagerAddress = _troveManagerAddress;
        stabilityPoolAddress = _stabilityPoolAddress;
        defaultPoolAddress = _defaultPoolAddress;
        collSurplusPoolAddress = _collSurplusPoolAddress;
        treasuryAddress = _treasuryAddress;
        lqtyStakingAddress = _lqtyStakingAddress;

(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L96-L103)

FILE:  2023-02-ethos/Ethos-Core/contracts/BorrowerOperations.sol


        collateralConfig = ICollateralConfig(_collateralConfigAddress);
        troveManager = ITroveManager(_troveManagerAddress);
        activePool = IActivePool(_activePoolAddress);
        defaultPool = IDefaultPool(_defaultPoolAddress);
        stabilityPoolAddress = _stabilityPoolAddress;
        gasPoolAddress = _gasPoolAddress;
        collSurplusPool = ICollSurplusPool(_collSurplusPoolAddress);
        priceFeed = IPriceFeed(_priceFeedAddress);
        sortedTroves = ISortedTroves(_sortedTrovesAddress);
        lusdToken = ILUSDToken(_lusdTokenAddress);
        lqtyStakingAddress = _lqtyStakingAddress;
        lqtyStaking = ILQTYStaking(_lqtyStakingAddress);

       (https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/BorrowerOperations.sol#L142-L153)


FILE : 2023-02-ethos/Ethos-Core/contracts/LQTY/CommunityIssuance.sol

        OathToken = IERC20(_oathTokenAddress);
        stabilityPoolAddress = _stabilityPoolAddress;


(https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L74-L75)

     







     
       

      






























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