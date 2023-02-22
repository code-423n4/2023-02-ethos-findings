 Gas optimization issue in ReaperStrategyGranarySupplyOnly.sol

Summary:
In the ReaperStrategyGranarySupplyOnly.sol contract located at https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol, we identified a gas optimization issue related to the initialization of a private constant variable. Specifically, in line 35, the variable LENDER_REFERRAL_CODE_NONE is initialized with the value 0, which is redundant as Solidity assumes a default value of 0 for uint variables that are not explicitly initialized.

Impact:

This issue has a minimal impact on the functionality of the contract, but it results in unnecessary gas consumption when deploying and executing the contract.

Recommendation:
We recommend removing the redundant initialization of the LENDER_REFERRAL_CODE_NONE variable in line 35 to optimize the gas consumption of the contract. The updated code would look like this:

Copy code
Solution 

uint16 private constant LENDER_REFERRAL_CODE_NONE;

We believe that this change will not affect the behavior or functionality of the contract, as the default value of 0 will be assumed for this variable if it is not explicitly initialized.
