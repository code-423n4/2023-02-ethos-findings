> ###  Gas optimization issue in ReaperStrategyGranarySupplyOnly.sol

Summary:
In the ReaperStrategyGranarySupplyOnly.sol contract located at https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol, we identified a gas optimization issue related to the initialization of a private constant variable. Specifically, in line 35, the variable LENDER_REFERRAL_CODE_NONE is initialized with the value 0, which is redundant as Solidity assumes a default value of 0 for uint variables that are not explicitly initialized.

Impact:

This issue has a minimal impact on the functionality of the contract, but it results in unnecessary gas consumption when deploying and executing the contract.

Recommendation:
We recommend removing the redundant initialization of the LENDER_REFERRAL_CODE_NONE variable in line 35 to optimize the gas consumption of the contract. The updated code would look like this:

Solution 

     uint16 private constant LENDER_REFERRAL_CODE_NONE;

We believe that this change will not affect the behavior or functionality of the contract, as the default value of 0 will be assumed for this variable if it is not explicitly initialized.


------------------------------------------------------------------------------------------------------------------------------------------------------


> ### USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
 When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. 

    contract LUSDToken is CheckContract, ILUSDToken {
    using SafeMath for uint256;
    
    uint256 private _totalSupply;
    string constant internal _NAME = "LUSD Stablecoin";
    string constant internal _SYMBOL = "LUSD";
    string constant internal _VERSION = "1";
    uint8 constant internal _DECIMALS = 18; //HERE

    bool public mintingPaused = false;
    
----------------------------------

    function permit
    (
        address owner, 
        address spender, 
        uint amount, 
        uint deadline, 
        uint8 v,  //HERE
        bytes32 r, 
        bytes32 s
    ) 
        external 
        override 
    { 

--------------------------------------
     }

    function decimals() external view override returns (uint8) //HERE  
    { 
        return _DECIMALS;
    }

In code - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol 
------------------------------------


     function decimals() public view override returns (uint8) //HERE
    {
        return token.decimals();
    }

in code - https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol