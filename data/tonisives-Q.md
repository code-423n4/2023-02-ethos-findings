### List of 5 issues

- TroveManager should have an ownership transfer process.

- _bps 10_000 should be a constant [https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol/#L127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol/#L127). It is used in multiple places and it is common practice to store constants in a shared place.

- `BO.moveCollateralGainToTrove` is not called and can be removed [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol/#L248)

- StabilityPool never calls BorrowOperations, but there is a check in `BorrowOperations._adjustTrove` [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol/#L301)
    
    ```solidity
    (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0)
    ```
    
    Moreover, StabilityPools-s borrowerOperations is never called and can be removed [here](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol/#L152)
    
    ```solidity
    IBorrowerOperations public borrowerOperations;
    ```

- Remove `ActivePool.manualRebalance()`. Contract should not have a simulation function that is only used in testing. This gives unnecessary privilege to the owner who can use this to rebalance by unnatural amounts.