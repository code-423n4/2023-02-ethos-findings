- Redundancy in setting variable

In line [https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32) the variable addressesSet is initialised as addressesSet = false as follows: bool public addressesSet = false; 

However, the default value for a bool in Solidity is false, so this line can be simplified to 

bool public addressesSet;

- Consistency and shorter name

In line [https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L205](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L205), DEFAULT_ADMIN_ROLE could be simplified to DEFAULT_ADMIN. The “ROLE” is not necessary and it makes the name longer. It is also not consistent with the naming of the other roles, e.g. ADMIN, GUARDIAN, STRATEGIST, KEEPER. For example, the KEEPER role is not named as KEEPER_ROLE so the DEFAULT_ADMIN_ROLE could be simplified to DEFAULT_ADMIN.

- Misleading comment

In line [https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L356](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L356) the comment says “LUSD: Cannot transfer tokens directly to the StabilityPool, TroveManager or BorrowerOps”. However, according to the code, the tokens can be transferred to the “true” StabilityPool as in the function SendToPool but not other StabilityPools.