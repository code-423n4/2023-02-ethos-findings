**Ethos-Core/contracts/BorrowerOperations.sol**
- L15 - The console dependency is imported, but it is never used, so it should be removed.

- L47/66 - The created structs do not meet a standard, but instead mixes camelcase and snakecase. For a matter of maintaining a single pattern, it would be beneficial if you use only one and not both at the same time.


**Ethos-Core/contracts/TroveManager.sol**
- L14/18/19 - There is a commented code that does not serve any specific purpose, therefore it should be eliminated, for example the import to ownable
which is also used in the declaration of the contract.

-Missing NATSPEC
Consider adding NATSPEC on all public/external functions to improve documentation.

- L129/140/146 - The created structs do not meet a standard, but instead mixes camelcase and snakecase. For a matter of maintaining a single pattern, it would be beneficial if you use only one and not both at the same time.

- L250/551/716/754/1450/1523/1527/1531/1535/1539 - When used to validate a require, the correct thing to do would be to display a message, so that the user who receives the message knows exactly why it is reverting.


**Ethos-Core/contracts/ActivePool.sol**
- L15 - The console dependency is imported, but it is never used, so it should be removed.

- L220 - The created structs do not comply with a standard, but mixes camelcase and snakecase. For a matter of maintaining a single pattern, it would be beneficial if you use only one and not both at the same time.


**Ethos-Core/contracts/StabilityPool.sol**
- L5/8 - The same interface is imported twice, therefore one of the two imports (IBorrowerOperations) should be eliminated.

- L18 - The console dependency is imported, but it is never used, so it should be removed.

- L160 - A variable is created in storage, lqtyTokenAddress, which is never used, therefore it should be used.

- L174/175 - The Deposit struct is created, but it only has one parameter inside. The concept of structs should be used to unify variables and in this case, there is only one.

- L240 - The DefaultPoolAddressChanged event is created, but it is never used, therefore it should be eliminated.

- L646 - The created structs do not meet a standard, but instead mixes camelcase and snakecase. For a matter of maintaining a single pattern, it would be beneficial if you use only one and not both at the same time.


**Ethos-Core/contracts/LQTY/CommunityIssuance.sol**
- L8 - The LiquityMath dependency is imported, but it is never used, so it should be removed.


**Ethos-Core/contracts/LQTY/LQTYStaking.sol**
- L9 - The console dependency is imported, but it is never used, so it should be removed.

- L50 - The LUSDTokenAddressSet event is created, but it is never used, therefore it should be removed.


**Ethos-Core/contracts/LUSDToken.sol**
- L6/9 - The console and ITroveManager dependencies are imported, but they are never used, therefore they should be removed.


**Ethos-Vault/contracts/ReaperVaultV2.sol**
- L48 - The variable in storage constructionTime is created, but it is only used to be set, it is not used for anything else, therefore the correct thing to do would be to delete it. Since it does not have any functionality.

- L334 - A division by freeFunds is performed, but it is never validated if freeFunds != 0, this should be necessary otherwise an exception would occur that is not being handled.

- L473 - The created structs do not comply with a standard, but mixes camelcase and snakecase. For a matter of maintaining a single pattern, it would be beneficial if you use only one and not both at the same time.


**Ethos-Vault/contracts/ReaperVaultERC4626.sol**
- L3 - Pragma float This contracts in scope are floating the pragma version.

- Missing NATSPEC Consider adding NATSPEC on all public/external functions to improve documentation.


**Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol**
- L3 - Pragma float This contracts in scope are floating the pragma version.

- L10 - The Initializable dependency is imported, but it is never used, therefore it should be removed.

- L25 - To set the value FUTURE_NEXT_PROPOSAL_TIME it is defined as follows: 365 days * 100, but it could be done directly like this: 36500 days.


**Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol**
- L3 - Pragma float This contracts in scope are floating the pragma version.

- L32 - The gWant variable is created in storage, but is never used, if ReaperBaseStrategyv4 wants, so the gWant variable could be removed.

