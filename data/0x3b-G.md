# Unnecessary SLOAD 

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L422-L431



        uint totalLUSD = totalLUSDDeposits; // cached to save an SLOAD

        if (totalLUSD == 0 || _LQTYIssuance == 0) {return;}

        uint LQTYPerUnitStaked;
        LQTYPerUnitStaked =_computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSD);

        uint marginalLQTYGain = LQTYPerUnitStaked.mul(P);

In the `if`  statement, it is not necessary to check `totalLUSD` since `totalLUSD=totalLUSDDeposits`, and since it is not needed  there is also no need for `totalLUSD` to be initialized, since it is only used in 1 place- `LQTYPerUnitStaked =_computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSD)`
to fix the code you can redo it as follows: 

        if (_LQTYIssuance == 0) {return;}

        uint LQTYPerUnitStaked;
        LQTYPerUnitStaked =_computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSDDeposits);

This can be even extended to the code bellow, although this is increasing complexity and lowers code readability.

    uint marginalLQTYGain = _computeLQTYPerUnitStaked(_LQTYIssuance, totalLUSDDeposits).mul(P)


# Use uint256(1)/uint256(2) instead of bool in structs

This prevents unnecessary gas usage, since uint is cheaper than bool
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L28

        struct Config {
        bool allowed; 
        uint256 decimals;
        uint256 MCR;
        uint256 CCR;
       }

To be:

      struct Config {
        uint allowed;
        uint256 decimals;
        uint256 MCR;
        uint256 CCR;
       }

# Useless struct

The one line of code located in `Deposit` can be moved out of the struct and the struct can be removed. Since there is no purpose for the struct to hold only 1 variable.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L174-L176

       struct Deposit {
       uint initialValue;
       }

# Mark secure math with `unchecked{}`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L181

        upgradeProposalTime = block.timestamp + FUTURE_NEXT_PROPOSAL_TIME;

Since `upgradeProposalTime` is uint256 and `FUTURE_NEXT_PROPOSAL_TIME` is only 3 153 600 000, it is unlikely to overflow 

# a=a+b is cheaper than a+=b 
It is much cheaper to make it  as a = a+b rather than a+=b.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L505
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194-L196
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L452
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L133

    strategy.gains += vars.gain; 

should be: 

    strategy.gains = strategy.gains + vars.gain;


# Use named returns  

[Ethos-Core/contracts/LQTY/LQTYStaking.sol/L217](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L217)

     function _getPendingLUSDGain(address _user) internal view returns (uint ) {
        uint F_LUSD_Snapshot = snapshots[_user].F_LUSD_Snapshot;
        uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);
        return LUSDGain;
      }

Fix:

     function _getPendingLUSDGain(address _user) internal view returns (uint LUSDGain) {
        uint F_LUSD_Snapshot = snapshots[_user].F_LUSD_Snapshot;
        uint LUSDGain = stakes[_user].mul(F_LUSD.sub(F_LUSD_Snapshot)).div(DECIMAL_PRECISION);
      }

  [Ethos-Vault/contracts/ReaperVaultV2.sol/L462](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462)

[Ethos-Core/contracts/TroveManager.sol/L1066](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1066)
[Ethos-Core/contracts/TroveManager.sol/L1201](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1201)
[Ethos-Core/contracts/TroveManager.sol/L1045](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1045)

 [Ethos-Core/contracts/BorrowerOperations.sol/L661](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661)
 [Ethos-Core/contracts/BorrowerOperations.sol/L682](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L682)
[Ethos-Core/contracts/BorrowerOperations.sol/L703](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L703)
[Ethos-Core/contracts/BorrowerOperations.sol/L724](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L724)








