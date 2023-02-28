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

------------------------------------------------------

> ### variable = 0
### Introduction
Gas optimization is a crucial aspect of smart contract development because it helps to reduce the overall cost of executing contracts on the Ethereum blockchain. In Solidity, using certain operators can impact the efficiency of a contract, resulting in higher gas costs and slower execution times. This report analyzes the gas optimization issue in Solidity code where using the > 0 operator with unsigned integers is less efficient than using the != 0 operator.

### Issue Description
The gas optimization issue arises when comparing unsigned integers to zero in Solidity. Solidity optimizes the != 0 operator to a single JUMP instruction, while the > 0 operator requires two JUMP instructions. As a result, using != 0 can lead to lower gas costs and faster contract execution times.


### Issue found here - 

 https://raw.githubusercontent.com/code-423n4/2023-02-ethos/main/Ethos-Vault/contracts/ReaperVaultV2.sol

code 

    function report(int256 _roi, uint256 _repayment) external returns (uint256) {
        LocalVariables_report memory vars;
        vars.stratAddr = msg.sender;
        StrategyParams storage strategy = strategies[vars.stratAddr];
        require(strategy.activation != 0, "Unauthorized strategy");

        if (_roi < 0) {
            vars.loss = uint256(-_roi);
            _reportLoss(vars.stratAddr, vars.loss);
        } else if (_roi > 0){  //should be != 0
            vars.gain = uint256(_roi);
            vars.fees = _chargeFees(vars.stratAddr, vars.gain);
            strategy.gains += vars.gain;
        }

        vars.available = availableCapital();
        if (vars.available < 0) { 
            vars.debt = uint256(-vars.available);
            vars.debtPayment = Math.min(vars.debt, _repayment);

            if (vars.debtPayment != 0) {
                strategy.allocated -= vars.debtPayment;
                totalAllocated -= vars.debtPayment;
                vars.debt -= vars.debtPayment; // tracked for return value
            }
        } else if (vars.available > 0) {                //should be != 0
            vars.credit = uint256(vars.available);
            strategy.allocated += vars.credit;
            totalAllocated += vars.credit;
        }


in  https://raw.githubusercontent.com/code-423n4/2023-02-ethos/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol

code 

    function increaseF_Collateral(address _collateral, uint _collFee) external override {
        _requireCallerIsTroveManagerOrActivePool();
        uint collFeePerLQTYStaked;
     
        if (totalLQTYStaked > 0) //should be != 0
     {collFeePerLQTYStaked = _collFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}

        F_Collateral[_collateral] = F_Collateral[_collateral].add(collFeePerLQTYStaked);
        emit F_CollateralUpdated(_collateral, F_Collateral[_collateral]);
    }

    function increaseF_LUSD(uint _LUSDFee) external override {
        _requireCallerIsBorrowerOperations();
        uint LUSDFeePerLQTYStaked;
        
        if (totalLQTYStaked > 0) //should be != 0
     {LUSDFeePerLQTYStaked = _LUSDFee.mul(DECIMAL_PRECISION).div(totalLQTYStaked);}
        
        F_LUSD = F_LUSD.add(LUSDFeePerLQTYStaked);
        emit F_LUSDUpdated(F_LUSD);

and code 

    function _requireCallerIsTroveManagerOrActivePool() internal view {
        address redemptionHelper = address(ITroveManager(troveManagerAddress).redemptionHelper());
        require(
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == activePoolAddress,
            "LQTYStaking: caller is not TroveM or ActivePool"
        );
    }

    function _requireCallerIsBorrowerOperations() internal view {
        require(msg.sender == borrowerOperationsAddress, "LQTYStaking: caller is not BorrowerOps");
    }

    function _requireUserHasStake(uint currentStake) internal pure {  
        require(currentStake > 0, //should be != 0             
    'LQTYStaking: User must have a non-zero stake');  
    }

    function _requireNonZeroAmount(uint _amount) internal pure {
        require(_amount > 0, //should be != 0
    'LQTYStaking: Amount must be non-zero');
    }
}

in https://raw.githubusercontent.com/code-423n4/2023-02-ethos/main/Ethos-Core/contracts/BorrowerOperations.sol

code - 
  
    if (!_isDebtIncrease && _LUSDChange > 0) { //should be != 0
            _requireAtLeastMinNetDebt(_getNetDebt(vars.debt).sub(vars.netDebtChange));
            _requireValidLUSDRepayment(vars.debt, vars.netDebtChange);
            _requireSufficientLUSDBalance(contractsCache.lusdToken, _borrower, vars.netDebtChange);
      }


      }

     function _requireNonZeroDebtChange(uint _LUSDChange) internal pure {
        require(_LUSDChange > 0,  //should be != 0 
     "BorrowerOps: Debt increase requires non-zero debtChange");
     }
   


### Impact
Using the > 0 operators with unsigned integers can lead to higher gas costs and slower contract execution times, as it requires two JUMP instructions. In contrast, using the != 0 operator can reduce gas costs and speed up contract execution by optimizing the code to a single JUMP instruction.

### Conclusion
Using the != 0 operator with unsigned integers in Solidity is more efficient than using the > 0 operators. The != 0 operator optimizes code to a single JUMP instruction, while the > 0 operator requires two JUMP instructions. By using the != 0 operator, developers can reduce gas costs and improve the overall efficiency of their smart contracts.

### References -  https://code4rena.com/reports/2022-03-timeswap/#low-risk-and-non-critical-issues 

### https://twitter.com/gzeon/status/1485426745247698946